- 用于转义ASCII字符
- ASCII一共是7位，最大127，八进制最大177，十六进制最大7F
- 在C/C++中使用`\`用于转义，转义的格式为三位八进制或x加二位十六进制
	- 八进制：``\nnn``
		- 最大177
		- 可以有前导0，比如``\000``，等价于``\0``，等价于``\00``
	- 十六进制：`\xhh`
		- 最大7f
		- 可以有前导0，比如`\x01`，等价于`\x1`
- 为了方便，定义了一些字母转义简写方式
	- | Escape sequence | Hex value in ASCII | Character represented |
	  | ---- | ---- | ---- |
	  | \a | 07 | Alert (Beep, Bell) |
	  | \b | 08 | Backspace |
	  | \e | 1B | Escape character |
	  | \f | 0C | [Formfeed](https://en.wikipedia.org/wiki/Formfeed) [Page Break](https://en.wikipedia.org/wiki/Page_Break) |
	  | \n | 0A | [Newline](https://en.wikipedia.org/wiki/Newline) (Line Feed); see notes below |
	  | \r | 0D | [Carriage Return](https://en.wikipedia.org/wiki/Carriage_Return) |
	  | \t | 09 | [Horizontal Tab](https://en.wikipedia.org/wiki/Horizontal_Tab) |
	  | \v | 0B | [Vertical Tab](https://en.wikipedia.org/wiki/Vertical_Tab) |
	  | \\ | 5C | [Backslash](https://en.wikipedia.org/wiki/Backslash) |
	  | \' | 27 | [Apostrophe](https://en.wikipedia.org/wiki/Apostrophe) or single quotation mark |
	  | \" | 22 | Double [quotation mark](https://en.wikipedia.org/wiki/Quotation_mark) |
	  | \? | 3F | [Question mark](https://en.wikipedia.org/wiki/Question_mark) (used to avoid [trigraphs](https://en.wikipedia.org/wiki/Digraphs_and_trigraphs#C)) |
-