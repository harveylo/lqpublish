# The long-running agent problem
	- 用人话讲就是：基于LLM无状态的特性，如何让LLM在一个新对话中，**[[$red]]==短时间内快速获取==**当前项目状态、目标背景、可利用的工具、接下来的目标等核心信息，以让LLM在一个有效的上下文中工作？(**[[$red]]==Recover to Work==**)
	- 我个人觉得，这个问题实际上已经被spec-driven基本解决了
	- Claude Code团队自身使用了一个双agent模式来解决这个问题
		- Initialization agent， 用于在第一次会话时生成后续所需的环境artifacts：一个`init.sh`脚本，一个记录agent工作的claude-progress.txt文件，一个显示添加了哪些文件的初始git commit
		- Coding agent，之后的每一次对话，都由此agent递增推进工作进程，并留下结构化的更新
- # Environment management
	- 之前提到的，claude code在初次对话时，会生成一些环境artifacts，这些artifacts主要包括以下东西
	- ## Feature List
		- 当用户给出比较高维与模糊的自然语言描述工作内容之后，agent会自动提取并分割出达成相应需求所需要的feature，记为一个feature list
		- ```json
		  {
		      "category": "functional",
		      "description": "New chat button creates a fresh conversation",
		      "steps": [
		        "Navigate to main interface",
		        "Click the 'New Chat' button",
		        "Verify a new conversation is created",
		        "Check that chat area shows welcome state",
		        "Verify conversation appears in sidebar"
		      ],
		      "passes": false
		    }
		  ```
		- 使用强烈语气的提示词，显式要求llm不得更改此文件内容，只能更改`passes`字段。
		- 选择使用json文件，也是因为llm更加偏向不修改json文件，而md文件就没有这个待遇
	- ## Incremental Progress
		- 后续的coding agent会被要求一次只工作在一个feature上。
		- 可以用外部文件记录进度，要求agent王其中添加进度汇报
		- 同时也要求agent在适当的时机直接做出git commit，并在message中也做出适当的描述
	- ## Testing
		- 在没有额外工具的帮助下，LLM可能也会有意识地写单元测试。
		- 但是如果能给到合适的测试相关的prompt与MCP，能显著提高agent的代码能力。
		- 例如，给ClaudeCode配置puppeteer MCP之后，claudecode能自主完成前端的端到端测试，极大提高了代码质量与debug效率。
- # Conclusion
	- Agent failure modes and solutions
		- | **Problem** | **Initializer Agent Behavior** | **Coding Agent Behavior** |
		  | Claude declares victory on the entire project too early. | Set up a feature list file: based on the input spec, set up a structured JSON file with a list of end-to-end feature descriptions. | Read the feature list file at the beginning of a session. Choose a single feature to start working on. |
		  | Claude leaves the environment in a state with bugs or undocumented progress. | An initial git repo and progress notes file is written. | Start the session by reading the progress notes file and git commit logs, and run a basic test on the development server to catch any undocumented bugs. End the session by writing a git commit and progress update. |
		  | Claude marks features as done prematurely. | Set up a feature list file. | Self-verify all features. Only mark features as “passing” after careful testing. |
		  | Claude has to spend time figuring out how to run the app. | Write an `init.sh` script that can run the development server. | Start the session by reading `init.sh`. |