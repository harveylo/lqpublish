# 安装texlive
	- ``sudo apt install texlive-full``
- # VSCode安装插件和更改设置
	- 安装latex-work shop
	- 在用户settings.json中加入：
		- ```json
		      "latex-workshop.latex.autoBuild.run": "onFileChange",
		      "latex-workshop.showContextMenu": true,
		      "latex-workshop.intellisense.package.enabled": true,
		      "latex-workshop.message.error.show": true,
		      "latex-workshop.message.warning.show": true,
		      "latex-workshop.latex.tools": [
		          {
		              "name": "xelatex",
		              "command": "xelatex",
		              "args": [
		                  "-synctex=1",
		                  "-interaction=nonstopmode",
		                  "-file-line-error",
		                  "%DOCFILE%"
		              ]
		          },
		          {
		              "name": "pdflatex",
		              "command": "pdflatex",
		              "args": [
		                  "-synctex=1",
		                  "-interaction=nonstopmode",
		                  "-file-line-error",
		                  "%DOCFILE%"
		              ]
		          },
		          {
		              "name": "latexmk",
		              "command": "latexmk",
		              "args": [
		                  "-synctex=1",
		                  "-interaction=nonstopmode",
		                  "-file-line-error",
		                  "-pdf",
		                  "-outdir=%OUTDIR%",
		                  "%DOCFILE%"
		              ]
		          },
		          {
		              "name": "bibtex",
		              "command": "bibtex",
		              "args": [
		                  "%DOCFILE%"
		              ]
		          }
		      ],
		      "latex-workshop.latex.recipes": [
		          {
		              "name": "XeLaTeX",
		              "tools": [
		                  "xelatex"
		              ]
		          },
		          {
		              "name": "PDFLaTeX",
		              "tools": [
		                  "pdflatex"
		              ]
		          },
		          {
		              "name": "BibTeX",
		              "tools": [
		                  "bibtex"
		              ]
		          },
		          {
		              "name": "LaTeXmk",
		              "tools": [
		                  "latexmk"
		              ]
		          },
		          {
		              "name": "xelatex -> bibtex -> xelatex*2",
		              "tools": [
		                  "xelatex",
		                  "bibtex",
		                  "xelatex",
		                  "xelatex"
		              ]
		          },
		          {
		              "name": "pdflatex -> bibtex -> pdflatex*2",
		              "tools": [
		                  "pdflatex",
		                  "bibtex",
		                  "pdflatex",
		                  "pdflatex"
		              ]
		          },
		      ],
		      "latex-workshop.latex.clean.fileTypes": [
		          "*.aux",
		          "*.bbl",
		          "*.blg",
		          "*.idx",
		          "*.ind",
		          "*.lof",
		          "*.lot",
		          "*.out",
		          "*.toc",
		          "*.acn",
		          "*.acr",
		          "*.alg",
		          "*.glg",
		          "*.glo",
		          "*.gls",
		          "*.ist",
		          "*.fls",
		          "*.log",
		          "*.fdb_latexmk"
		      ],
		      "latex-workshop.latex.autoClean.run": "onFailed",
		      "latex-workshop.latex.recipe.default": "lastUsed",
		      "latex-workshop.view.pdf.internal.synctex.keybinding": "double-click"
		  ```