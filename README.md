# Deepseek联网搜索

为 MaiBot 提供联网搜索与网页查看能力，通过 DeepSeek API 检索公开资料并返回摘要。

向 MaiBot 注册两个工具：

| 工具 | 功能 |
| --- | --- |
| `search_and_summarize` | 联网搜索并整理检索结果，支持搜索档位和时间、地区、语言、版本、来源偏好。 |
| `fetch_page` | 检索指定公开 HTTP/HTTPS URL，核对来源后返回页面内容摘要。 |

**搜索通过 DeepSeek API 执行，需要 API 密钥并会产生费用。** DeepSeek 负责搜索及检索资料摘要，最终聊天回复仍由 MaiBot 主模型组织。

## 环境与安装

- MaiBot 1.x，采用 Plugin SDK v2 的宿主。
- Python 3.11+。
- `maibot-plugin-sdk>=2.5.2,<3.0.0`。
- `anthropic>=0.104.0,<1.0.0`。

使用根目录 `plugin.py` 的 `create_plugin()` 工厂和 SDK 的 `@Tool` 注册工具。manifest v2 声明插件类型为 `integration`，能力请求列表为空。

将发布包解压至 `plugins/deepseek-web-searcher/`，保证 `plugin.py` 和 `_manifest.json` 直接位于该目录。依赖应安装到 **MaiBot 实际运行的 Python 环境**，例如在宿主使用的 uv 项目中执行：

```powershell
uv pip install --python <MaiBot使用的Python路径> "anthropic>=0.104.0,<1.0.0" "maibot-plugin-sdk>=2.5.2,<3.0.0"
```

重启或重新加载插件。首次加载由宿主生成 `config.toml`，可在 WebUI 配置。

## 配置

推荐在 MaiBot 进程环境中设置 `DEEPSEEK_API_KEY`，保持 `auth.api_key` 留空。也可以在 WebUI 填写密钥，配置密钥优先于环境变量。不要提交实际 `config.toml`。

| 分组 | 设置 | 默认与含义 |
| --- | --- | --- |
| plugin | enabled | `true`；关闭后全部入口停止请求。 |
| plugin | max_concurrent_requests | `4`；其余请求排队，总等待队列有上限。 |
| plugin | coalesce_requests | `true`；仅合并同一会话中完全相同的在途请求，不缓存答案。 |
| auth | api_key / api_key_env | 配置密钥或环境变量，默认 `DEEPSEEK_API_KEY`。 |
| model | model_choice | 默认 `deepseek-flash`，可选择 `deepseek-v4-pro`。 |
| model | max_tokens | `4096`；检索摘要输出上限。 |
| model | request_timeout_seconds | `120`；包含排队；SDK 自动重试关闭。 |
| search | enabled | `true`；关闭后搜索和网页查看均不请求 API。 |
| search | web_search_tool | 默认 `web_search_20260209`，可选择 `web_search_20250305`。 |
| search | max_search_uses | `5`；每请求最多搜索次数，范围 1–20。 |
| search | default_depth | `standard`；quick 最多 2 次、standard 最多 3 次、deep 使用管理员上限。 |
| debug | log_search_sources | `true`；来源写入日志，可能含 URL 查询参数。 |
| debug | log_raw_summary | `false`；开启后记录耗时、token 和停止原因，不记录完整提示词或回答。 |

搜索范围是传给后端的偏好，不是硬过滤。所有档位固定关闭思考。每次搜索在出队后附加服务器本地时间与 UTC 偏移，部署时请确认服务器时间正确。

## 使用

正常聊天中由支持工具调用的 MaiBot 主模型决定何时搜索：

```text
帮我查一下某个软件最近的更新，优先查官方说明。
打开 https://example.com/article 看看主要讲了什么。
```

工具返回检索摘要或错误提示。没有搜索结果、来源不匹配、搜索服务错误或输出截断都会明确提示。

## 网页查看边界

`fetch_page` 通过搜索检索指定网页并核对来源，**不保证获得全文**。无法读取登录页面、无法保证绕过反爬或读取动态内容。返回的资料可能只是搜索索引片段。

网页来源需匹配指定 URL。仅规范化主机大小写、默认端口与空路径，保留路径、查询和片段；同站首页、父子页面、不同文章参数不算读取目标页。重定向、锚点及尾部斜杠变化可能造成校验失败。即使匹配也不能证明全文已读取。

检索请求会发送至 DeepSeek；网页资料按不可信输入处理。工具不会主动添加完整来源清单，但后端摘要可能包含引用链接。

若工具结果在 MaiBot 后处理阶段被截断，可检查“麦麦设置 → 后处理 → 高级设置”的 `enable_overflow_return_all`，具体位置以宿主版本为准。

## 插件信息

插件信息：

- 中文名：Deepseek联网搜索。
- 插件 ID：`Qhzy1411.deepseek-web-searcher`。
- 仓库：`https://github.com/Qhzy1411/Maibot_Deepseek_Web_Searcher`。

## 许可

MIT，许可声明见 [LICENSE](LICENSE)。

## 来源与授权

本项目由 Qhzy1411 维护和发布，部分实现基于 [DeepSeek Anthropic Provider](https://github.com/LowValueTarget777/deepseek-anthropic-provider) 开发。该项目由 LowValueTarget 开发，并以 MIT 许可证授权使用、修改和分发。本项目同样采用 MIT 许可证，LICENSE 中保留了 LowValueTarget 的版权声明，并补充了 Qhzy1411 对本项目修改部分的版权声明。
