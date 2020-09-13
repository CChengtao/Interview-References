#### 1、cin

使用流提取符">>"从输入流中提取数据，遇到空格、Tab键、换行符就停止读取后面的字符，会吸收掉空格、Tab键，不会吸收'\n'，即'\n'仍然留在输入流中。

注：键盘输入完数据并按Enter键后，该行数据(包括换行符)才被送入到键盘缓冲区，形成输入流，流提取符">>"才能从中提取数据。

```cpp
#include<iostream>
#include<cstring>
using namespace std;
int main() {
    char a[100];
	cin>>a;    
	cout<<a<<endl;
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello
```

#### 2、gets（）

遇到回车键结束（有时与getchar()配合使用）；

```cpp
#include<iostream>
#include<cstdio>
using namespace std;
int main() {
    char a[100];
	gets(a);
	cout<<a<<endl;
	return 0;
}
```

输入：

```
hello world!
```

 输出：

```
hello world!
```

#### 3、cin.get（）

可以接收空格，遇到回车键结束（有时与getchar()配合使用）；

```cpp
#include<iostream>
using namespace std;
int main() {
    char a[100];
	cin.get(a,100);
	cout<<a<<endl;
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello world!
```

此外还有

```cpp
cin.get(ch);
ch = cin.get();
```

其中ch 是char型，可以一个一个接收字符，包括空字。

#### 4、getline（）

针对string类型，可以接收空格，遇到回车键结束（有时与getchar()配合使用）；

```cpp
#include<iostream>
#include<string>
using namespace std;
int main() {
	string a;
	getline(cin,a);
	cout<<a<<endl;
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello world!
```

#### 5、cin.getline（）

cin.getline()函数与cin.get()函数类似，可以接收空格，遇到回车键结束（有时与getchar()配合使用）；

```cpp
#include<iostream>
using namespace std;
int main() {
	char a[100];
	cin.getline(a,100);
	cout<<a<<endl;
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello world!
```

#### 6、scanf

① 遇到空格或者回车键结束（有时与getchar()配合使用）；

```cpp
#include <iostream>
using namespace std;
int main() {
	char a[20];
	scanf("%s",a);
	puts(a);
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello
```

② 让scanf可以接收空格，遇到回车键结束（有时与getchar()配合使用）；

这里主要介绍一个参数，％[　]，这个参数的意义是读入一个字符集合。[　]是个集合的标志,因此%[　]特指读入此集合所限定的那些字符，比如%[A-Z]是输入大写字母，一旦遇到不在此集合的字符便停止。如果集合的第一个字符是“^”，这说明读取不在“^“后面集合的字符，即遇到”^“后面集合的字符便停止。此时读入的字符串是可以含有空格的。

```cpp
#include <iostream>
using namespace std;
int main() {
	char a[100];
	scanf("%[^\n]",a);
	puts(a);
	return 0;
}
```

输入：

```
hello world!
```

输出：

```
hello world!
```

```cpp
#include <iostream>
using namespace std;
int main() {
	int n;
	char a[100];
	cin>>n;
	getchar();//吸收换行符，一定要加 
	for(int i=0;i<n;i++)
		scanf("%[^\n]",a);
	puts(a);
	return 0;
}
```

输入：

    11
    hello world!

输出：

```
hello world!
```

#### 7、cin与getline连用问题

cin>>与getline的工作方式，流提取运算符根据它后面的变量类型读取数据，从非空白符号开始，遇到Enter、Space、Tab键时结束。getline函数从istream中读取一行数据，当遇到“\n”时结束返回。

造成程序错误结果的原因是，用户输入完年龄后按回车结束输入，把“\n”留在了输入流里，而cin不会主动删除输入流内的换行符，这样换行符就被getline读取到，getline遇到换行符返回，因此程序不会等待用户输入。
解决的办法是手动清除换行符，在cin>>后加上

```cpp
cin.ignore();
```

