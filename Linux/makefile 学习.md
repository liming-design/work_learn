## 简单的makefile
![image](https://img2022.cnblogs.com/blog/1865959/202207/1865959-20220702231918803-1312638697.png)

## 变量 :=
1. $@
表示规则中目标，例如 hello.out。

2. $<
表示规则中的第一个依赖条件，例如 hello.c

3. $^
表示规则中的所有依赖条件，由于我们示例中都只有一个依赖条件，这种情况下 $^ 和 $< 区别不大。

```bash
foo = $( bar)
bar = $( ugh)
ugh = Huh?
all:
	echo $(foo)
```
我们执行“make all”将会打出变量\$(foo)的值是“Huh?”\$(foo)的值是\$(bar)$ (bar)的值是$(ugh)，$(ugh)的值是“Huh?”)可见，变量是可以使用后面的变量来定义的。
这个功能有好的地方，也有不好的地方，好的地方是，我们可以把变量的真实值推到后面来定义，如:
但这种形式也有不好的地方，那就是递归定义，如:
`CFLAGS = $(CFLAGS) -o`
这会让make 陷入无限的变量展开过程中去，当然，我们的make是有能力检测这样的定义，并会报错。还有就是如果在变量中使用函数，那么，这种方式会让我们的make运行时非常慢，更糟糕的是，他会使得两个make 的函数“wildcard”和“shell”发生不可预知的错误。因为你不会知道这两个函数会被调用多少次。
为了避免上面的这种方法，我们可以使用make 中的另一种用变量来定义变量的方法。这种方法使用的是“:=”操作符，如:
x := foo
y := $(x)bar
x := later
值得一提的是，这种方法，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量。

## 函数相关
https://www.cnblogs.com/edver/p/7229792.html





