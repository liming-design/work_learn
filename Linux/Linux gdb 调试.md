## gdb调试
参考：[](https://blog.csdn.net/qq_26686565/article/details/79978229)
在编辑器中的代码
```
#include<stdio.h>
#include <stdlib.h>
struct ListNode 
{
     int val;
     struct ListNode *next;
};
struct ListNode* Create(int a)
{
	int c=0;
	struct ListNode *t1,*t2,*t3;
	t1=(struct ListNode*)malloc(sizeof(struct ListNode));
	t1->next=NULL;
	t2=t1;
	while(a)
	{
		c=a%10;
		t2->val=c;
		a/=10;
		if(a>0)
		{
			t3=(struct ListNode*)malloc(sizeof(struct ListNode));
			t3->next=NULL;
			t2->next=t3;
			t2=t2->next;
		}
		else
		{
			t2->next=NULL;
		}
	}
	
	return t1;
}

void Print(struct ListNode* ll)
{
	struct ListNode *l=ll; 
	while(l)
	{
		printf("%d",l->val);
		if(l->next)
			printf("->");
		else
			printf("\n");
		l=l->next;
	}
}


struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) 
{
	//在此写下您的代码
	struct ListNode *ll1=l1,*ll2=l2; 
	struct ListNode *ans,*ans1,*t3;
	
	ans=(struct ListNode*)malloc(sizeof(struct ListNode));
	ans->next=NULL;
	ans1=ans;
	
	int s=0;
	
	while(ll1&&ll2)
	{
		int t=ll1->val+ll2->val+s;
		ans1->val=(t%10);
		
		if(ll1->next&&ll2->next)
		{
			t3=(struct ListNode*)malloc(sizeof(struct ListNode));
			t3->next=NULL;
			ans1->next=t3;
			ans1=ans1->next;
		}
		else
		{
			ans1->next=NULL;
		}
		
		s=t/10;
		ll1=ll1->next;
		ll2=ll2->next;
	}
	while(ll1)
	{
		t3=(struct ListNode*)malloc(sizeof(struct ListNode));
		t3->next=NULL;
		t3->val=ll1->val;
		ans1->next=t3;
		ll1=ll1->next;
	}
	while(ll2)
	{
		t3=(struct ListNode*)malloc(sizeof(struct ListNode));
		t3->next=NULL;
		t3->val=ll2->val;
		ans1->next=t3;
		ll2=ll2->next;
	}
	return ans;
}

void DeleteAll(struct ListNode* l)
{
	struct ListNode* ll=l;
	while(ll)
	{
		struct ListNode* t=ll;
		ll=ll->next;
		free(t);
	}
}

int main()
{
	int a,b;
	struct ListNode *t1,*t2,*t3;
	//scanf("%d",&a);
	//scanf("%d",&b);
	t1=Create(342);
	t2=Create(465);
	t3=addTwoNumbers(t1,t2);
	Print(t1);
	Print(t2);
	Print(t3);
	DeleteAll(t1);
	DeleteAll(t2);
	DeleteAll(t3);
}
```
将.c文件在编辑器中写好后上传Linux虚拟机。

### 编译运行阶段
gcc t2.c  -o t2out 

### 运行：
./t2out

### 调试阶段：
```
list(l)：列出源代码，接着上次的位置往下列，每次列10行
list 行号：列出产品从第几行开始的源代码
list 函数名：列出某个函数的源代码
start：开始执行程序，停在main函数第一行语句前面等待命令
next(n)：执行下一列语句
step(s):执行下一行语句，如果有函数调用则进入到函数中
breaktrace(或bt)：查看各级函数调用及参数
frame(f) 帧编号：选择栈帧
info(i) locals：查看当前栈帧局部变量的值
finish：执行到当前函数返回，然后挺下来等待命令
print(p)：打印表达式的值，通过表达式可以修改变量的值或者调用函数
set var：修改变量的值
quit：退出gdb
```

```
break(b) 行号：在某一行设置断点
break 函数名：在某个函数开头设置断点
break...if...：设置条件断点
continue(或c)：从当前位置开始连续而非单步执行程序
delete breakpoints：删除所有断点
delete breakpoints n：删除序号为n的断点
disable breakpoints：禁用断点
enable breakpoints：启用断点
info(或i) breakpoints：参看当前设置了哪些断点
run(或r)：从开始连续而非单步执行程序
display 变量名：跟踪查看一个变量，每次停下来都显示它的值
undisplay：取消对先前设置的那些变量的跟踪
```

```
watch：设置观察点
info(或i) watchpoints：查看当前设置了哪些观察点
```

```bash
(gdb) directory ../test1/
Source directories searched: /mnt/hgfs/share/C++/GDB/ManyFiles/../test1:$cdir:$cwd 

(gdb) b test1.cpp:2
Breakpoint 1 at 0x4008b2: file ../test1/test1.cpp, line 2.(gdb) rStarting program: /mnt/hgfs/share/C++/GDB/ManyFiles/test ​Breakpoint 1, test1 () at ../test1/test1.cpp:55       cout<
```

## 打印链表
### 编写脚本文件
print-list.gdb
```
define print-list
    set $list = ($arg1*)$arg0
    while $list
        print *$list
        set $list = $list->next
    end
end
```

### 在gdb中
```
(gdb)source print-list.dgb
(gdb)print-list t1 list_node
```

## 程序被优化，无法打印局部变量
```bash
(gdb) bt
#0  strategy_judgment (pSess=<optimized out>) at ./npb_process.c:1053
Segmentation fault

[root@localhost mp_npb]# scl enable devtoolset-10 bash
[root@localhost mp_npb]# gdb -p $(pidof mp_npb)

```

### 只运行当前线程
```bash
set scheduler-locking on
```


