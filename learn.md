
  - vibe/cli/：CLI 层。entrypoint.py 负责参数解析、加载配置/历史、选择 programmatic 模式或 Textual UI；textual_ui/ 目录是基于 Textual 的 TUI（app.py 入口，widgets/handlers 组成界面和事件
    流）；commands.py 定义斜杠命令；history_manager.py 管理本地历史文件。
  - vibe/core/：核心逻辑。
      - agent.py 是主对话/工具协调器，维护消息列表、调用 LLM backend、运行中间件（token 预算、价格限制、自动压缩等）、保存会话日志。
      - config.py 定义配置模型（providers/models/session_logging/MCP 等）和加载逻辑。
      - interaction_logger.py 负责会话日志的存储/读取/列出。
      - types.py 定义公共数据模型（LLMMessage/LLMChunk/Usage/Stats 等）。
      - system_prompt.py、prompts/（在 repo 外 ~/.vibe）生成系统提示。
      - llm/：LLM 后端抽象与适配器。backend/ 下有 generic.py（OpenAI 风格，含流式解析与 token 计数）、mistral.py 等；format.py 负责消息/工具 schema 转换；types.py 约束接口。
      - middleware/：会话中间件（turn limit、price limit、自动压缩等）。
      - tools/：工具体系。base.py、manager.py 管理工具注册/调用；builtins/ 提供 bash/read/write/search 等内置工具；approval 处理工具执行审批。
      - output_formatters.py：程序化模式下的输出组装。
      - programmatic.py：非交互模式的运行入口。
  - vibe/setup/：首次运行的 onboarding（收集 API key、主题等）。
  - tests/：覆盖 CLI、工具、LLM 交互等的测试；包括 Textual UI snapshot/UI 测试、工具单测、agent 行为测试等。
  - 顶层文件：
      - pyproject.toml、uv.lock：依赖与工具配置。
      - README.md、CONTRIBUTING.md：使用说明与贡献指引。
      - scripts/：可能包含发布/辅助脚本。
      - distribution/、action.yml 等 CI/打包元数据。

  运行流程（简要）：

  1. vibe 命令触发 entrypoint.py；加载配置、选择模式。
  2. 创建 Agent：初始化系统提示、工具管理器、LLM backend，中间件上线文。
  3. 每轮用户输入 → 中间件检查 → LLM 请求（llm/backend/* 通过 provider/model 配置决定） → 工具调用/处理 → 消息更新，必要时记日志（interaction_logger）。
  4. UI 层（Textual）通过 VibeApp 与 Agent 协程交互，渲染消息流、工具输出、审批对话框等。

  如果想深入，可以从 vibe/core/agent.py + vibe/core/llm/backend/generic.py（流式/工具调用）和 vibe/core/tools/builtins/（工具接口）入手，再看 cli/textual_ui/app.py 如何绑定到核心。