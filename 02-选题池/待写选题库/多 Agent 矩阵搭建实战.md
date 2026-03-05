---
标题：多 Agent 矩阵搭建实战
阶段：待写
关联参考：[[饼干哥哥 OpenClaw+Obsidian 内容工厂爆文生产]]
标签：[OpenClaw, 多 Agent, 软链接，权限配置，知识管理]
创建日期：2026-03-06
来源：饼干哥哥 AGI 微信公众号
---

# 多 Agent 矩阵搭建实战

## 选题背景

OpenClaw 安全沙盒机制严格，每个 Agent 只能访问自己工作区内的文件。但内容中台的终极形态需要多个专职 Agent 协作：
- **研究员**：全网监控抓取素材
- **小红书专员**：撰写种草文案
- **公众号 Agent**：深度文章创作
- **审稿总管**：统筹全局

如何让多个 Agent 共享同一套知识库？

## 核心方案：软链接（Symlink）

### 1. 建立中立的物理存储

```bash
# 真实数据源位置
/Users/binggan/Documents/bgggcontent
# 或本地
/home/chengnanfeng/myos
```

### 2. 为各 Agent 建立专属通道

```bash
ln -s /home/chengnanfeng/myos /root/.openclaw/workspace-gzh/myos
ln -s /home/chengnanfeng/myos /root/.openclaw/workspace-xhs/myos
ln -s /home/chengnanfeng/myos /root/.openclaw/workspace-research/myos
```

### 3. 效果

- 素材库是唯一真实数据源
- 研究员把竞品分析丢进爆款素材库，公众号 Agent 零延迟读取并写作
- 所有 Agent 操作同步到同一知识库

## 待验证要点

1. 软链接权限问题（root vs 普通用户）
2. Obsidian 是否能正确识别软链接后的双向链接
3. 多 Agent 同时写入的冲突处理

## 参考

- [[饼干哥哥 OpenClaw+Obsidian 内容工厂爆文生产]] - 架构设计章节
