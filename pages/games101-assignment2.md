- **[[$red]]==MSAA一直不生效的原因==**
	- `insideTriangle`函数的形参x和y是整数类型，因此每次计算子像素的时候并没有真的计算子像素，还是在计算当前像素的坐标，导致像素插值计算失败
	- 需要将这两个形参的类型改为`float`即可生效