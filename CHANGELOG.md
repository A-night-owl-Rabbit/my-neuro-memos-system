# 更新日志

## 2026-03-29

### 检索 API（`/search`）与时间字段

- **知识图谱「仅图谱」补全**：原先 `get_memory` 返回的结构中时间戳只在 `payload` 内，导致合并进检索结果后 **`created_at` / `timestamp` 为空**。现已将图谱补全行 **展平** 为与向量检索一致的字典，保证返回中带创建时间。
- **格式化结果兜底**：在组装返回给前端的 `memories` 列表时，若顶层缺少时间，会从 **`payload`** 读取 `created_at`、`timestamp`、`updated_at`，避免遗漏。
- **响应增强**：检索结果中在可用时为每条记忆附带 **`id`**，便于与 WebUI / 工具修正记忆等流程对照。

### 说明

- 本仓库 **不包含** 带真实 API Key 的 `memos_config.json`；部署时请本地配置 LLM 与 Embedding。
- 与前端插件 [my-neuro-plugin-memos](https://github.com/A-night-owl-Rabbit/my-neuro-plugin-memos) 同日更新一并使用，端到端时间戳展示最完整。
