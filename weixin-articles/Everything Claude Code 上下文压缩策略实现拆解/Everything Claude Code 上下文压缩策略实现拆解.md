---
title: "Everything Claude Code 上下文压缩策略实现拆解"
公众号: "凡人修AI传"
date: "1970-01-01 08:33"
source: "https://mp.weixin.qq.com/s/mfrirEc95qqy74SW2fNCHA"
---

## Token问题从哪来

用过Claude Code的人应该都遇到过这种情况——一开始对话还挺顺，聊着聊着就变慢了，最后弹出"context window full"。这时候才想起来，Claude的上下文是有上限的（Sonnet 200K tokens）。

ECC（Everything Claude Code）这个项目是在Anthropic hackathon上获奖的作品，作者Affaan Mustafa花了10个月打磨。它要解决的就是这个硬限制问题：**怎么塞更多功能还不让会话崩掉**。

这篇分享一下几个关键技术点。


## Agent描述压缩：从26K压到2K


### 问题在哪

ECC有27个specialized agents，每个都是markdown文件，包含完整说明、工具定义、使用场景。全部加载的话，**光是agent描述就要占121KB，约26K tokens**。

什么概念？上下文窗口直接吃掉13%。


### 三层压缩模式

scripts/lib/agent-compress.js里实现了压缩库，提供三种模式：

**catalog模式（只留元数据）**


```
functioncompressToCatalog(agent) {return {name: agent.name,description: agent.description,tools: agent.tools,model: agent.model,  };}
```

这个模式只保留name、description、tools、model四个字段，**27个agent压缩后只有2-3K tokens**，压缩率超过85%。

**summary模式（元数据+首段摘要）**


```
functionextractSummary(body, maxSentences = 1) {const lines = body.split('\n');const paragraphs = [];let current = [];let inCodeBlock = false;for (const line of lines) {const trimmed = line.trim();// 跟踪代码块状态if (trimmed.startsWith('```')) {      inCodeBlock = !inCodeBlock;continue;    }if (inCodeBlock) continue;if (trimmed === '') {if (current.length > 0) {        paragraphs.push(current.join(' '));        current = [];      }continue;    }// 跳过标题、列表项、编号列表、表格行if (      trimmed.startsWith('#') ||      trimmed.startsWith('- ') ||      trimmed.startsWith('* ') ||/^\d+\.\s/.test(trimmed) ||      trimmed.startsWith('|')    ) {if (current.length > 0) {        paragraphs.push(current.join(' '));        current = [];      }continue;    }    current.push(trimmed);  }const firstParagraph = paragraphs.find(p => p.length > 0);if (!firstParagraph) return'';const sentences = firstParagraph.match(/[^.!?]+[.!?]+/g) || [firstParagraph];return sentences.slice(0, maxSentences).map(s => s.trim()).join(' ').trim();}
```

这个模式会解析agent正文，跳过代码块和标题，提取第一段内容。压缩后大约4-5K tokens。

**full模式（完整内容）**

只有真正调用某个agent时，才懒加载整个内容。


### 懒加载机制

最巧妙的是lazyLoadAgent函数：


```
functionlazyLoadAgent(agentsDir, agentName) {// 安全校验：只允许字母数字下划线和横线if (!/[\w-]+$/.test(agentName)) returnnull;const filePath = path.resolve(agentsDir, `${agentName}.md`);// 路径遍历防护if (!filePath.startsWith(resolvedAgentsDir + path.sep)) returnnull;if (!fs.existsSync(filePath)) returnnull;returnloadAgent(filePath);}
```

这个设计保证了三点：

1. 1. 安全：有路径遍历防护，防止恶意agentName读取任意文件
2. 2. 按需加载：catalog模式只加载元数据，真正要用的时候才加载完整body
3. 3. 快速选择：agent选择阶段只需要2-3K tokens，而不是26K

**安全测试**（tests/lib/agent-compress.test.js）：


```
// 测试路径遍历防护if (test('lazyLoadAgent rejects path traversal attempts', () => {const agent = lazyLoadAgent(tmpDir, '../etc/passwd');  assert.strictEqual(agent, null);}));// 测试非法字符过滤if (test('lazyLoadAgent rejects names with invalid characters', () => {const agent1 = lazyLoadAgent(tmpDir, 'foo/bar');  assert.strictEqual(agent1, null);const agent2 = lazyLoadAgent(tmpDir, 'foo bar');  // 空格  assert.strictEqual(agent2, null);const agent3 = lazyLoadAgent(tmpDir, 'foo..bar'); // 双点  assert.strictEqual(agent3, null);}));// 测试不存在的agentif (test('lazyLoadAgent returns null for non-existent agent', () => {const agent = lazyLoadAgent(tmpDir, 'does-not-exist');  assert.strictEqual(agent, null);}));
```

从测试代码里能看到效果验证：


```
// tests/lib/agent-compress.test.jsconst result = buildAgentCatalog(realAgentsDir, { mode: 'catalog' });assert.ok(result.agents.length > 0, 'Should find at least one agent');assert.ok(result.stats.compressedBytes < result.stats.originalBytes,'Catalog should be smaller than original');// 验证压缩率const ratio = result.stats.compressedBytes / result.stats.originalBytes;assert.ok(ratio < 0.5, `Compression ratio ${ratio.toFixed(2)} should be < 0.5`);// 实际结果: ratio ≈ 0.15 (85% 压缩率)// 验证token估算assert.ok(  result.stats.compressedTokenEstimate < 5000,`Token estimate ${result.stats.compressedTokenEstimate} exceeds 5000`);// 27 个 agent 压缩后约 2500 tokens
```

**压缩率超过50%，token占用从26K降到2-3K**。


### Frontmatter解析实现

Agent文件用YAML frontmatter存元数据，解析器要处理各种边界情况：


```
functionparseFrontmatter(content) {// 匹配 ---\n frontmatter \n---\n bodyconst match = content.match(/^---\r?\n([\s\S]*?)\r?\n---(?:\r?\n([\s\S]*))?$/);if (!match) {return { frontmatter: {}, body: content };  }const frontmatter = {};for (const line of match[1].split('\n')) {const colonIdx = line.indexOf(':');if (colonIdx === -1) continue;const key = line.slice(0, colonIdx).trim();let value = line.slice(colonIdx + 1).trim();// 处理 JSON 数组（如 tools: ["Read", "Grep"]）if (value.startsWith('[') && value.endsWith(']')) {try {        value = JSON.parse(value);      } catch {// 解析失败保持原字符串      }    }// 去除引号包裹if (typeof value === 'string' &&        value.startsWith('"') && value.endsWith('"')) {      value = value.slice(1, -1);    }    frontmatter[key] = value;  }return { frontmatter, body: match[2] || '' };}
```

**测试覆盖**：


```
// tests/lib/agent-compress.test.jsif (test('parseFrontmatter extracts YAML frontmatter and body', () => {const content = '---\nname: test-agent\ndescription: A test\ntools: ["Read", "Grep"]\nmodel: sonnet\n---\n\nBody text here.';const { frontmatter, body } = parseFrontmatter(content);  assert.strictEqual(frontmatter.name, 'test-agent');  assert.strictEqual(frontmatter.description, 'A test');  assert.deepStrictEqual(frontmatter.tools, ['Read', 'Grep']);  assert.strictEqual(frontmatter.model, 'sonnet');  assert.ok(body.includes('Body text here.'));}));if (test('parseFrontmatter handles content without frontmatter', () => {const content = 'Just a regular markdown file.';const { frontmatter, body } = parseFrontmatter(content);  assert.deepStrictEqual(frontmatter, {});  assert.strictEqual(body, content);}));if (test('parseFrontmatter handles colons in values', () => {const content = '---\nname: test\ndescription: Use this: it works\n---\n\nBody.';const { frontmatter } = parseFrontmatter(content);  assert.strictEqual(frontmatter.description, 'Use this: it works');}));
```


## Context Budget：审计Token去哪了

ECC提供了/context-budget命令，专门帮你分析token消耗。


### 四大组件扫描

它会扫描设置，统计四类组件的token消耗。

**核心扫描逻辑**（skills/context-budget/SKILL.md）：


```
// Agent 扫描functionscanAgents(agentsDir) {const agents = fs.readdirSync(agentsDir)    .filter(f => f.endsWith('.md'))    .map(f => {const content = fs.readFileSync(path.join(agentsDir, f), 'utf8');const lines = content.split('\n');const tokens = Math.ceil(lines.length * 1.3); // 每行约 1.3 tokens// 解析 frontmatter 检查 description 长度const { frontmatter } = parseFrontmatter(content);const descWords = frontmatter.description?.split(/\s+/).length || 0;return {name: f,lines: lines.length,        tokens,heavy: lines.length > 200,        // 标记重型 agentbloated: descWords > 30,          // 标记臃肿 frontmatter      };    });return {count: agents.length,totalTokens: agents.reduce((sum, a) => sum + a.tokens, 0),issues: agents.filter(a => a.heavy || a.bloated),  };}// MCP Server 扫描functionscanMcpServers(mcpConfig) {const servers = Object.values(mcpConfig.mcpServers || {});const toolCount = servers.reduce((sum, s) =>    sum + (s.tools?.length || 0), 0);const cliTools = ['gh', 'git', 'npm', 'aws', 'supabase', 'vercel'];const cliReplaceable = servers.filter(s =>    cliTools.some(cli =>      s.name?.includes(cli) || s.description?.includes(cli)    )  );return {serverCount: servers.length,    toolCount,estimatedTokens: toolCount * 500,  // 每个 tool 约 500 tokensissues: [      servers.length > 10 && `${servers.length} MCP servers (recommend <10)`,      ...cliReplaceable.map(s =>`${s.name} is CLI-replaceable`),    ].filter(Boolean),  };}
```

**扫描细节：**

**Agents** (agents/*.md)

- • 每行按1.3个token估算
- • 标记超过200行的"重型agent"
- • 标记description超过30个词的"臃肿frontmatter"

**Skills** (skills/*/SKILL.md)

- • 同样按1.3 token/行估算
- • 标记超过400行的大文件
- • 检查.agents/skills/里的重复副本

**MCP Servers** (.mcp.json)

- • 每个tool schema大约500 tokens
- • 标记超过20个tools的server
- • 标记那些包装简单CLI命令的server（比如gh、git、npm）

**CLAUDE.md**

- • 项目级+用户级的CLAUDE.md文件
- • 标记合并后超过300行的情况


### 输出报告示例


```
Context Budget Report═══════════════════════════════════════Total estimated overhead: ~33,000 tokensContext model: Claude Sonnet (200K window)Effective available context: ~167,000 tokens (83%)Component Breakdown:┌─────────────────┬────────┬───────────┐│ Component       │ Count  │ Tokens    │├─────────────────┼────────┼───────────┤│ Agents          │ 16     │ ~12,400   ││ Skills          │ 28     │ ~6,200    ││ Rules           │ 8      │ ~4,800    ││ MCP tools       │ 87     │ ~43,500   ││ CLAUDE.md       │ 2      │ ~1,200    │└─────────────────┴────────┴───────────┘⚠ Issues Found (3):- 3 heavy agents (>200 lines)- 14 MCP servers enabled (recommend <10)- 3 CLI-replaceable MCP serversTop 3 Optimizations:1. Remove 3 MCP servers → save ~27,500 tokens (47% overhead reduction)2. Use catalog mode for agents → save ~11,000 tokens3. Compact CLAUDE.md → save ~400 tokens
```

这个工具的价值在于**可视化**——很多时候你以为只是加了个小功能，实际上可能已经吃掉了一半的上下文窗口。


## Strategic Compact：在合适的时机做合适的事

ECC不依赖Claude Code的自动compaction（那个往往在不合适的时机触发，比如任务做到一半突然压缩，重要上下文就丢了）。他们搞了一个**Strategic Compact**机制。


### 原理：在工具调用时计数

在scripts/hooks/suggest-compact.js里，通过PreToolUse hook（在Edit/Write操作前触发）来计数：


```
// 完整实现asyncfunctionmain() {// 从环境变量获取 session ID，并做安全过滤const sessionId = (process.env.CLAUDE_SESSION_ID || 'default')    .replace(/[^a-zA-Z0-9_-]/g, '') || 'default';// 会话隔离：每个 session 有独立的计数文件const counterFile = path.join(getTempDir(), `claude-tool-count-${sessionId}`);// 读取阈值配置（默认 50），带边界检查const rawThreshold = parseInt(process.env.COMPACT_THRESHOLD || '50', 10);const threshold = Number.isFinite(rawThreshold) &&    rawThreshold > 0 && rawThreshold <= 10000    ? rawThreshold    : 50;let count = 1;// 原子读写：使用文件描述符避免并发冲突try {const fd = fs.openSync(counterFile, 'a+');try {const buf = Buffer.alloc(64);const bytesRead = fs.readSync(fd, buf, 0, 64, 0);if (bytesRead > 0) {const parsed = parseInt(buf.toString('utf8', 0, bytesRead).trim(), 10);// 边界检查：防止超大值或损坏数据        count = (Number.isFinite(parsed) && parsed > 0 && parsed <= 1000000)          ? parsed + 1          : 1;      }// 截断并写入新值      fs.ftruncateSync(fd, 0);      fs.writeSync(fd, String(count), 0);    } finally {      fs.closeSync(fd);    }  } catch {// 降级方案：如果 fd 操作失败，静默处理writeFile(counterFile, String(count));  }// 触发建议逻辑if (count === threshold) {log(`[StrategicCompact] ${threshold} tool calls reached - consider /compact`);  }if (count > threshold && (count - threshold) % 25 === 0) {log(`[StrategicCompact] ${count} tool calls - good checkpoint for /compact`);  }// 关键：始终退出 0，不阻塞 Claude  process.exit(0);}
```

默认阈值是50次工具调用，之后每隔25次提醒一次。

**测试验证**（tests/hooks/suggest-compact.test.js）：


```
// 阈值触发测试if (test('suggests compact at threshold', () => {runCompact({ CLAUDE_SESSION_ID: testSession, COMPACT_THRESHOLD: '3' });runCompact({ CLAUDE_SESSION_ID: testSession, COMPACT_THRESHOLD: '3' });const result = runCompact({ CLAUDE_SESSION_ID: testSession, COMPACT_THRESHOLD: '3' });  assert.ok(result.stderr.includes('3 tool calls reached'));}));// 每 25 次间隔提醒if (test('suggests at threshold + 25 interval', () => {  fs.writeFileSync(counterFile, '27');  // 下次运行将是第 28 次const result = runCompact({ CLAUDE_SESSION_ID: testSession, COMPACT_THRESHOLD: '3' });  assert.ok(result.stderr.includes('28 tool calls'));}));// 损坏数据恢复if (test('resets counter on corrupted file', () => {  fs.writeFileSync(counterFile, 'not-a-number');runCompact({ CLAUDE_SESSION_ID: testSession });const count = parseInt(fs.readFileSync(counterFile, 'utf8'), 10);  assert.strictEqual(count, 1);}));// 超大值处理if (test('resets counter on extremely large value', () => {  fs.writeFileSync(counterFile, '9999999');runCompact({ CLAUDE_SESSION_ID: testSession });const count = parseInt(fs.readFileSync(counterFile, 'utf8'), 10);  assert.strictEqual(count, 1);}));// 始终退出 0，不阻塞 Claudeif (test('always exits 0', () => {const result = runCompact({ CLAUDE_SESSION_ID: testSession });  assert.strictEqual(result.code, 0);}));
```


```
if (count === threshold) {log(`[StrategicCompact] ${threshold} tool calls reached - consider /compact`);}if (count > threshold && (count - threshold) % 25 === 0) {log(`[StrategicCompact] ${count} tool calls - good checkpoint for /compact`);}
```


### 策略性压缩的时机

ECC建议你在这些**明确的任务边界**做compact：

阶段转换

是否压缩

原因

Research → Planning

是

研究成果很占地方，但计划是提炼后的精华

Planning → Implementation

是

计划可以存到TodoWrite或文件里，腾出空间写代码

Implementation → Testing

看情况

如果测试要引用刚写的代码，先别压

Debugging → Next feature

是

调试trace对新功能来说是噪音

Mid-implementation

否

会丢失变量名、文件路径等关键信息


### 什么东西能存活过compaction

很多人不敢compact，怕丢东西。ECC明确列出了哪些内容会保留：

**能存活的：**

- • CLAUDE.md指令（always loaded）
- • TodoWrite任务列表
- • Memory文件(~/.claude/memory/)
- • Git状态（commits、branches）
- • 磁盘上的文件

**会丢失的：**

- • 中间推理和分析过程
- • 之前读过的文件内容
- • 多轮对话上下文
- • 工具调用历史和计数
- • 口头上表达的细微偏好

所以ECC的建议是：**重要内容先存到文件或memory再compact**。

**会话隔离实现**：


```
// Sanitize session ID to prevent path traversalconst sessionId = (process.env.CLAUDE_SESSION_ID || 'default')  .replace(/[^a-zA-Z0-9_-]/g, '') || 'default';const counterFile = path.join(getTempDir(), `claude-tool-count-${sessionId}`);
```

**测试验证**：


```
// 测试会话隔离if (test('uses separate counter files per session ID', () => {const sessionA = `compact-a-${Date.now()}`;const sessionB = `compact-b-${Date.now()}`;const fileA = getCounterFilePath(sessionA);const fileB = getCounterFilePath(sessionB);runCompact({ CLAUDE_SESSION_ID: sessionA });runCompact({ CLAUDE_SESSION_ID: sessionA });runCompact({ CLAUDE_SESSION_ID: sessionB });const countA = parseInt(fs.readFileSync(fileA, 'utf8').trim(), 10);const countB = parseInt(fs.readFileSync(fileB, 'utf8').trim(), 10);  assert.strictEqual(countA, 2, 'Session A should have count 2');  assert.strictEqual(countB, 1, 'Session B should have count 1');}));
```


## 环境变量层面的优化

除了架构层面的设计，ECC还提供了一套推荐的环境变量配置，**最高可以降低70%的token消耗**：


```
{"model":"sonnet","env":{"MAX_THINKING_TOKENS":"10000","CLAUDE_AUTOCOMPACT_PCT_OVERRIDE":"50","CLAUDE_CODE_SUBAGENT_MODEL":"haiku"}}
```

这些环境变量在代码中被读取并应用：


```
// scripts/hooks/suggest-compact.jsconst rawThreshold = parseInt(process.env.COMPACT_THRESHOLD || '50', 10);const threshold = Number.isFinite(rawThreshold) &&  rawThreshold > 0 && rawThreshold <= 10000  ? rawThreshold  : 50;// 边界处理：防止非法值if (test('ignores invalid COMPACT_THRESHOLD (negative)', () => {  fs.writeFileSync(counterFile, '49');const result = runCompact({CLAUDE_SESSION_ID: testSession,COMPACT_THRESHOLD: '-5'  });// 负值回退到 50  assert.ok(result.stderr.includes('50 tool calls reached'));}));if (test('ignores non-numeric COMPACT_THRESHOLD', () => {  fs.writeFileSync(counterFile, '49');const result = runCompact({CLAUDE_SESSION_ID: testSession,COMPACT_THRESHOLD: 'abc'  });// NaN 回退到 50  assert.ok(result.stderr.includes('50 tool calls reached'));}));
```

**MAX_THINKING_TOKENS=10000**

- • 默认是31999，Claude会用这些tokens做内部推理
- • 降到10000可以节省约70%的"隐藏成本"
- • 简单任务可以设为0完全关闭

**CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50**

- • 默认是95%，等到快满了才压缩，这时候质量已经下降了
- • 50%触发更健康，给复杂任务留足够余量

**CLAUDE_CODE_SUBAGENT_MODEL=haiku**

- • 子agent（Task tool）默认继承主模型，通常是Sonnet
- • 换成Haiku便宜80%，对于文件读取、代码搜索这类任务完全够用


## MCP Server的取舍

ECC在README里有个警告：**每个启用的MCP server都会往上下文窗口里塞tool定义**。如果MCP开太多，200K的窗口可能只剩70K。

MCP Server扫描逻辑实现：


```
// skills/context-budget/SKILL.mdfunctionscanMcpServers(mcpConfig) {const servers = Object.values(mcpConfig.mcpServers || {});const toolCount = servers.reduce((sum, s) =>    sum + (s.tools?.length || 0), 0);// 估算token消耗：每个tool约500 tokensconst estimatedTokens = toolCount * 500;// 检测CLI可替代的服务器const cliTools = ['gh', 'git', 'npm', 'aws', 'supabase', 'vercel'];const cliReplaceable = servers.filter(s =>    cliTools.some(cli =>      s.name?.includes(cli) || s.description?.includes(cli)    )  );const issues = [];if (servers.length > 10) {    issues.push(`${servers.length} MCP servers (recommend <10)`);  }if (toolCount > 100) {    issues.push(`High tool count: ${toolCount} tools (~${estimatedTokens} tokens)`);  }return {serverCount: servers.length,    toolCount,    estimatedTokens,    cliReplaceable,    issues,  };}
```

他们的建议：

1. 1. 保持10个以下MCP server
2. 2. 优先用CLI工具（gh代替GitHub MCP，aws代替AWS MCP）
3. 3. 用disabledMcpServers按项目禁用不需要的server
4. 4. Memory MCP默认配置了但实际没用，可以考虑关掉

Context Budget扫描器会标记"CLI-replaceable"的MCP server，告诉你哪些可以用命令行工具替代。


## 总结：Token优化是个系统工程

1. 1. Agent Compression —— 懒加载+三层压缩，26K→2～3K
2. 2. Context Budget —— 审计工具，让你知道token去哪了
3. 3. Strategic Compact —— 在任务边界做压缩，避免中断
4. 4. Model Routing —— Haiku做简单任务，Sonnet/Opus做复杂任务
5. 5. Thinking Tokens控制 —— 降低隐藏成本
6. 6. MCP管理 —— 减少不必要的tool schema开销

这些技术点背后有个共同的思路：**不要一次性把所有东西塞进上下文，而是按需加载、用完即弃、在合适的时机做清理**。

如果你也在用Claude Code做复杂项目，这些技巧值得参考。


## 关键源码参考


### Agent压缩实现


```
// scripts/lib/agent-compress.jsfunctionbuildAgentCatalog(agentsDir, options = {}) {const mode = options.mode || 'catalog';let agents = loadAgents(agentsDir);const originalBytes = agents.reduce((sum, a) => sum + a.byteSize, 0);let compressed;if (mode === 'catalog') {    compressed = agents.map(compressToCatalog);  } elseif (mode === 'summary') {    compressed = agents.map(compressToSummary);  } else {    compressed = agents.map(a => ({...a, body: a.body}));  }const compressedJson = JSON.stringify(compressed);const compressedTokenEstimate = Math.ceil(compressedJson.length / 4);return {agents: compressed,stats: {totalAgents: agents.length,      originalBytes,compressedBytes: Buffer.byteLength(compressedJson, 'utf8'),      compressedTokenEstimate,      mode,    },  };}
```


### Strategic Compact Hook


```
// scripts/hooks/suggest-compact.jsasyncfunctionmain() {const sessionId = (process.env.CLAUDE_SESSION_ID || 'default')    .replace(/[^a-zA-Z0-9_-]/g, '') || 'default';const counterFile = path.join(getTempDir(), `claude-tool-count-${sessionId}`);const threshold = parseInt(process.env.COMPACT_THRESHOLD || '50', 10);let count = 1;const fd = fs.openSync(counterFile, 'a+');try {const buf = Buffer.alloc(64);const bytesRead = fs.readSync(fd, buf, 0, 64, 0);if (bytesRead > 0) {const parsed = parseInt(buf.toString('utf8', 0, bytesRead).trim(), 10);      count = (Number.isFinite(parsed) && parsed > 0 && parsed <= 1000000)        ? parsed + 1        : 1;    }    fs.ftruncateSync(fd, 0);    fs.writeSync(fd, String(count), 0);  } finally {    fs.closeSync(fd);  }if (count === threshold) {log(`[StrategicCompact] ${threshold} tool calls reached - consider /compact`);  }if (count > threshold && (count - threshold) % 25 === 0) {log(`[StrategicCompact] ${count} tool calls - good checkpoint for /compact`);  }}
```


### 压缩效果验证测试


```
// tests/lib/agent-compress.test.jsconst result = buildAgentCatalog(realAgentsDir, { mode: 'catalog' });assert.ok(result.stats.compressedBytes < result.stats.originalBytes);const ratio = result.stats.compressedBytes / result.stats.originalBytes;assert.ok(ratio < 0.5, `Compression ratio ${ratio.toFixed(2)} should be < 0.5`);// 实际结果: ratio ≈ 0.15 (85% 压缩率)
```


## 运行测试验证

你可以直接运行ECC的测试套件来验证这些token优化功能：


```
# 运行Agent压缩测试node tests/lib/agent-compress.test.js
```

预期输出：


```
=== Testing agent-compress ===  ✓ parseFrontmatter extracts YAML frontmatter and body  ✓ parseFrontmatter handles content without frontmatter  ✓ parseFrontmatter handles colons in values  ✓ parseFrontmatter strips surrounding quotes  ✓ extractSummary returns the first paragraph of the body  ✓ extractSummary skips code blocks  ✓ extractSummary respects maxSentences  ✓ compressToCatalog strips body and keeps only metadata  ✓ buildAgentCatalog in catalog mode produces minimal output  ✓ buildAgentCatalog throws on invalid mode  ✓ lazyLoadAgent rejects path traversal attempts  ✓ lazyLoadAgent rejects names with invalid characters  ✓ buildAgentCatalog works with real agents directory  ✓ catalog mode token estimate is under 5000 for real agentsResults: Passed: 27, Failed: 0
```


```
# 运行Strategic Compact测试node tests/hooks/suggest-compact.test.js
```

预期输出：


```
=== Testing suggest-compact.js ===Basic counter functionality: ✓ creates counter file on first run ✓ increments counter on subsequent runsThreshold suggestion: ✓ suggests compact at threshold (COMPACT_THRESHOLD=3) ✓ does NOT suggest compact before thresholdInterval suggestion: ✓ suggests at threshold + 25 intervalEnvironment variable handling: ✓ uses default threshold (50) when COMPACT_THRESHOLD is not set ✓ ignores invalid COMPACT_THRESHOLD (negative) ✓ ignores non-numeric COMPACT_THRESHOLDCorrupted counter file: ✓ resets counter on corrupted file content ✓ resets counter on extremely large value ✓ handles empty counter fileSession isolation: ✓ uses separate counter files per session IDExit code: ✓ always exits 0 (never blocks Claude)Results: Passed: 20, Failed: 0
```

**相关资源：**

- • GitHub: https://github.com/affaan-m/everything-claude-code
- • Token Optimization Guide: docs/token-optimization.md
- • The Longform Guide: 作者Twitter上有完整的长文（@affaanmustafa）

*文章基于ECC v1.9.0代码分析，Agent Description Compression是本文重点分析的feature。*

**

PS：都看到这里，来个点赞、在看、关注吧。 您的支持是我坚持的最大动力！

