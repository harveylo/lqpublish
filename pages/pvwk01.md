- #### 3.4
- 满足部分正确性：i), ii), iv), vi)
- 满足完全正确性：i), ii), iv), vi)
- #### 3.9
##### (i)

$$
\begin{align*}
&<S,\sigma>
\\&\rightarrow<count:=y;while\ldots,\sigma[prod:=0]>
\\&\rightarrow<while\ldots,\sigma'[count:=\sigma(y)]>
\\&\rightarrow<prod:=prod+x;count:=count-1;while\ldots,\sigma'>
\\&\rightarrow<count:=count-1;while\ldots,\sigma'[prod=4]>
\\&\rightarrow<while\ldots,\sigma''[count:=2]>
\\&\rightarrow<prod:=prod+x;count:=count-1;while\ldots,\sigma'''>
\\&\rightarrow<count:=count-1;while\ldots,\sigma'''[prod:=8]>
\\&\rightarrow<while\ldots,\sigma''''[count:=1]>
\\&\rightarrow<prod:=prod+x;count:=count-1;while\ldots,\sigma'''''>
\\&\rightarrow<count:=count-1;while\ldots,\sigma'''''[prod:=12]>
\\&\rightarrow<while\ldots,\sigma''''''[count:=0]>
\\&\rightarrow<E,\sigma'''''''>
\end{align*}
$$
##### (ii)

给出一个循环不变式p：
$$
p\equiv prod=x*(y-count)\wedge count>=0
$$
给出一个边界函数：
$$
t\equiv count
$$
给出一个可行的证明框架：
$$
\begin{align*}
&\{x\ge0\wedge y\ge0\}
\\&prod:=0;
\\&\{x\ge0\wedge y\ge0\wedge prod=0\}
\\&count:=y;
\\&\{x\ge0\wedge y\ge0\wedge prod=0\wedge count=y\}
\\&\{p\}
\\&\{inv:p\}\ \ \{bd:t\}
\\&while\ count>0\ do
\\&\qquad\{p\wedge count>0\}
\\&\qquad prod:=prod+x;
\\&\qquad \{count>0\wedge prod=x*(y-count+1)\}
\\&\qquad count:=count-1;
\\&\qquad \{count>-1\wedge prod=x*(y-count)\}
\\&\qquad\{p\}
\\&od
\\&\{p\wedge count\le0\}
\\&\{prod=x*y\}
\end{align*}
$$
#### 3.10
##### (i)

设一个循环不变式p：
$$
p\equiv x=fib(n-count)\wedge count\ge0\wedge y=fib(n-count+1)
$$
设一个不变式t：
$$
t\equiv count
$$
给出一个可行的证明框架：
$$
\begin{align*}
&\{n\ge0\}
\\&x:=0;
\\&\{n\ge0\wedge x=0\}
\\&y:=1;
\\&\{n\ge0\wedge x=0\wedge y=1\}
\\& count:=n;
\{n\ge0\wedge x=0\wedge y=1\wedge count:=n\}
\\&\{p\}
\\&\{inv:p\}\ \ \{bd:t\}
\\&while\ count>0\ do
\\&\qquad\{p\wedge count>0\}
\\&\qquad h:=y;
\\&\qquad \{p\wedge count>0\wedge h=y\}
\\&\qquad y:=x+y;
\\&\qquad \{count>0\wedge h=fib(n-count+1)\wedge y=fib(n-count+2)\}
\\&\qquad x:=h;
\\&\qquad\{count>0\wedge x=fib(n-count+1)\wedge y=fib(n-count+2)\}
\\&\qquad count:=count-1
\\&\qquad \{count>-1\wedge x=fib(n-count)\wedge y=fib(n-count+1)\}
\\&\qquad\{p\}
\\& od
\\&\{p\wedge count\le0\}
\\&\{x=fib(n)\}
\end{align*}
$$