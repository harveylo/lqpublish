- 粘贴到系统剪贴板："*y
- 从系统剪贴板粘贴："*p
	- 如果实在insert mode下，使用<C-r><C-p>*
- # NERDTree 图表显示错误解决思路
	- 使用[bin/scripts/lib/](https://github.com/ryanoasis/nerd-fonts/blob/master/bin/scripts/lib)中的脚本安装字体
	- 更改终端字体(preferences -> unnamed -> text -> custom font)
- # vim 的自定义
	- 通过编辑~/.vimrc 来自定义vim
- # 按键描述规则
	- ^：表示ctrl键
		- 例如^V表示ctrl+v
	- 两个键组合可以表示为：
		- Ctrl-V
		- <C-V>
		- <S-V>，表示shift+v
- # vim状态
	- ## Normal 状态
		- **[[$red]]==转换方法==**
			- 打开vim时默认进入该状态
			- 任何其他状态下按<ESC>进入该状态
		- 该状态主要负责进行文件的快速浏览和一些编辑操作，输入不进入buffer，而是直接作为操作命令
		- 按下[[$blue]]==**u**==撤销编辑
			- **U**仅撤销在当前行的操作
		- 按下[[$blue]]==**<C-R>**== redo
		- **navigating**
			- 虽然vim可以使用方向键移动光标，但在normal mode下更推荐使用hjkl来移动
				- hjkl仅在normal mode下可以移动光标
			- hl：左右
			- jk：下上
			- w：向前一个单词
			- b：向后一个单词
			- e：移动到（下一个）单词末尾
			- 0：移动到当前行的开头
			- $：移动到当前行的末尾
			- ^（caret）：移动到当前行的第一个非空（可显示）字符
			- <C-U>：向上滚动
			- <C-D>：向下滚动
			- gg：跳到第一行
			- G：跳到最后一行
			- L：跳到当前页面的最低（lowest）行
			- M：跳到当前页面的中间（middle）行
			- H：跳到当前页面的最高（highest）行
			- <f-letter>：跳到当前行中，从当前cursor位置往后第一个字母所在的位置
			- <F-letter>：类似f，但是是向前跳转w
			- <t-letter>：类似f，但是比跳到字母所在位置前一个字符处，也有T版本
			- %：在两个对应的括号之间跳转
		- **editing**
			- O：在选中行开启新行，原行下移一行，并进入insert mode
			- o：在选中行的下一行开启新行，并进入insert mode
			- d：删除，配合相关键完成对应操作
				- w：从当前字符开始向后删除一个单词，保留分隔符（标点符号，空格）
				- W：同上，但不保留任何分隔符
				- e：从当前字符开始删除直到单词末尾
				- E：同上，但不保留任何标点符号
				- d：删除此行
			- c：类似于删除，但是删除之后进入insert mode
				- d下除了d之外的参数也可用
				- c：删除此行，但不删除换行，并进入insert mode
			- x：删除选中字符，不能删除换行
			- r：按下之后需要给出一个字符输入作为参数，将当前选中字符替换为输入的字符
			- y：复制（yank），需要一个参数完成复制操作
				- y：复制当前行
				- w：复制当前单词
				- 可以进入visual mode选中复制内容
			- ~：将选中内容的大小写切换
			- .：在当前位置重复上一次在inser mode下进行的编辑输入
		- **modifier**
			- 可以快速修改括号或引号内内容，通过c进行，后接：
			- i：在内部修改（inside，保留左右括号引号），[[$red]]==**后接一个括号引号字符**==，左右皆可
			- a：在外部修改（around，不保留左右括号引号），[[$red]]==**后接一个括号引号字符**==，左右皆可
		- **count**
			- 用于重复操作
			- 输入一个数字，然后跟随一个指令，可以是navigation指令也可以是editing指令，甚至可以是某个editing指令的参数，例如d2w\
		- **search**
			- /：搜索从当前位置开始第一次出现所搜寻模式的位置（当前文件看作头尾相连的环）
			- ?：向上搜索
			- 获取搜索结果之后按n跳转到下一个匹配结果，N到上一个匹配结果
			- 搜索完成之后使用nohlsearch(noh)关闭高亮
		- **numeric**
			- <C-A>：数字自增
			- <C-X>：数字自减
		- **search and replace**
			- 形如：``:(%)s/pattern/replace/options``
			- ``s``表示在当前行进行搜索替换
			- ``%s``表示全局搜索替换
			- pattern可包含正则表达式
			- 指令前可包含起作用的行数
				- ``5,20s/^/#/``
			- **options**
				- ``g``：替换所有pattern
					- 不添加该选项则只替换第一个符合pattern的
				- ``c``：对每一次替换都进行确认
				- ``I``：查找pattern时大小写敏感
					- 强制大小写不敏感搜索：`/harttle\c`
					- 强制大小写敏感搜索：`/harttle\C`
					- 强制大小写不敏感替换：`s/harttle\c/Harttle`
					- 强制大小写敏感替换：`s/harttle\C/Harttle`
					- 设置为大小写敏感：`:set ignorecase`
					- 设置为大小写不敏感：`:set noignorecase`
					- 设置为智能模式（有大写时敏感否则不敏感）：`:set smartcase`
					- 设置为非智能模式：`:set nosmartcase`
					-
	- ## Insert状态
		- **[[$red]]==转换方法==**
			- 在**normal**状态下按i键进入该状态
			- 按a也可以进入该状态，但是指针向后移动一个字符（append）
		- 该状态下进行输入操作，键盘输入进入文件中
	- ## Replace状态
		- **[[$red]]==转换方法==**
			- 在normal状态下按<S-R>键
	- ## Select状态
		- 分为三个子状态
		- ### Visual 状态
			- **[[$red]]==转换方法==**
				- 在normal状态下按v键
			- 在该状态下移动指针选定文本内容
			- y：复制当前选中内容并回到normal mode
			- d：剪切当前内容并回到normal mode
		- **Visual-line状态**
			- **[[$red]]==转换方法==**
				- 在normal状态下按<S-V>
			- 类似于visual mode，该状态下选择的内容都是以行为单位
		- **Visual-block**状态
			- **[[$red]]==转换方法==**
				- 在normal状态下按<C-V>
			- 以正方形选择文本内容
			- 选中一个区域之后，按下``I``可以在**[[$red]]==多行同时输入内容==**，按esc即可完成输入
	- ## Command-line状态
		- **[[$red]]==转换方法==**
			- 在normal状态下输入':'
		- **replace**
			- s：在当前行里查找并替代
			- %s：在全局查找
			- 格式：
				- (%)s/{pattern}/{target}/g
				- g表示全局替换
		- !：临时回到shell中执行shell命令，不会退出vim
- # 多窗口
	- 实际上，Vim也支持多窗口，多tab
	- 在command line模式下，使用指令sp课对同一个文件进行分屏显示，也可以附加文件名，分屏展示不同的文件
		- <C-W>加方向键选择向某个方向切换cursor所在的窗口
		- 在命令行模式中输入q退出（关闭）当前窗口
		- 输入qa关闭所有窗口和tab并退出vim
		- ``<C-W> <``：减少pane的宽度 ``<``前可以加数字，表示减少多少个单位
		- ``<C-W> >``：增加pane的宽度
		- ``<C-W> +``：增加pane高度
		- ``<C-W> -``：减少pane高度
		-
	- tab是更大的一个概念，一个tab可以分出多个窗口
		- 使用 tabnew打开新的tab，也可以加上要打开的文件名直接在新tab中打开文件
		- 使用命令tabn切换到下一个tab，或者使用快捷键``g t``
		- 使用命令tabp切换到上一个tab，或者使用快捷键``g T``
		- ``n g t``：切换到第``n``个tab，从1开始
- # Macros
	- 使用q加一个字符来开始录制宏，录制好之后的宏的名称为该字符
	- 再按一次q结束录制
	- 使用@加宏名称执行宏
	- 宏也可以配合counter使用
- # 插件指令
	- ## Snippet
		- 使用``<C-J>``激活snippet
	- ## easymotion
		- 使用ss 进入搜索模式，输入两个字符
		- 匹配最近的模式，按对应键到对应位置
	- ## ctags
		- 在需要查看的tags的目录运行ctags
		- 在vim中：`<Ctrl-]>`跳转到对应函数或变量的定义处，`<Ctrl-t>`回到查找处
	- ## NerdCommenter
		- ``<leader>``按键默认为``\``
		- ``<leader>cc``：注释
		- ``<leader>cn``：强制嵌套注释
		- ``<leader>c<space>``：切换注释状态，第一行被选中行如果是注释状态，则后续所有行都将解除注释，反之亦然
		- ``<leader>ci``：切换注释状态，每一行独立切换
		- 所有的指令前都可以加一个数字``n``，表示对接下来的n行进行对应操作
	- ## NerdTree
		- ?：快捷查看指令
	- ## YouCompleteMe
		- 快速配置指南：
			- 安装依赖
			- 安装ycm
			- 编译ycm
			- 安装lsp(clangd)
			- 启用语义trigger
				- ```
				  let g:ycm_semantic_triggers =  {
				    \   'c,cpp,objc': [ 're!\w{3}', '_' ],
				    \   'c': ['->', '.'],
				    \   'objc': ['->', '.', 're!\[[_a-zA-Z]+\w*\s', 're!^\s*[^\W\d]\w*\s',
				    \            're!\[.*\]\s'],
				    \   'ocaml': ['.', '#'],
				    \   'cpp,cuda,objcpp': ['->', '.', '::'],
				    \   'perl': ['->'],
				    \   'php': ['->', '::'],
				    \   'cs,d,elixir,go,groovy,java,javascript,julia,perl6,python,scala,typescript,vb': ['.'],
				    \   'ruby,rust': ['.', '::'],
				    \   'lua': ['.', ':'],
				    \   'erlang': [':'],
				    \   'json': [ 're!\w' ],
				  \ }
				  ```
		- 安装compilation database生成工具(bear, compliedb)
			- 在项目目录下生称compile_commands.json文件
		- 一些实用配置选项：
			- ```
			  let g:ycm_collect_identifiers_from_tags_files = 1           " 开启 YCM 基于标签引擎
			  let g:ycm_collect_identifiers_from_comments_and_strings = 1 " 注释与字符串中的内容也用于补全
			  "let g:syntastic_ignore_files=[".*\.py$"]
			  let g:ycm_seed_identifiers_with_syntax = 1                  " 语法关键字补全
			  let g:ycm_complete_in_comments = 1
			  let g:ycm_confirm_extra_conf = 0
			  "let g:ycm_key_list_select_completion = ['<c-n>', '<Down>']  " 映射按键, 没有这个会拦截掉tab, 导致其他插件的tab不能用.
			  "let g:ycm_key_list_previous_completion = ['<c-p>', '<Up>']
			  let g:ycm_complete_in_comments = 1                          " 在注释输入中也能补全
			  let g:ycm_complete_in_strings = 1                           " 在字符串输入中也能补全
			  let g:ycm_collect_identifiers_from_comments_and_strings = 1 " 注释和字符串中的文字也会被收入补全
			  let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'
			  "let g:ycm_show_diagnostics_ui = 0                           " 禁用语法检查
			  "inoremap <expr> <CR> pumvisible() ? "\<C-y>" : "\<CR>" |            " 回车即选中当前项
			  nnoremap <c-j> :YcmCompleter GoToDefinitionElseDeclaration<CR>|     " 跳转到定义处
			  ```