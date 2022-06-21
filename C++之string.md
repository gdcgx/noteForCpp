# C++之string

## **特性描述**

> `int capacity()const;    										//返回当前容量（即string中不必增加内存即可存放的元素个数）`
> `int max_size()const;    										//返回string对象中可存放的最大字符串的长度`
> `int size()const;        										//返回当前字符串的大小`
> `int length()const;       										//返回当前字符串的长度`
> `bool empty()const;        										//当前字符串是否为空`
> `void resize(int len,char c);									//把字符串当前大小置为len，并用字符c填充不足的部分`

```c++
int main(){
    std::string str = "hello world!!!";
    int cap = str.capacity();
    int max = str.max_size();
    int size = str.size();
    int len = str.length();
    bool emp = str.empty();
    str.resize(50,'.');
    std::cout << cap << "\n"
              << max << "\n"
              << size << "\n"
              << len << "\n"
              << emp << "\n"
              << str << std::endl;
	return 0;
} 
*****************************************************************************
15
-1
14
14
0
hello world!!!....................................
```

## **输入输出操作**

`string`类重载运算符`operator>>`用于输入，同样重载运算符`operator<<`用于输出操作。
函数`getline(istream &in,string &s);`用于从输入流`in`中读取字符串到`s`中，以换行符`'\n'`分开。

## **赋值**

> `string &operator=(const string &s);							//把字符串s赋给当前字符串`
> `string &assign(const char *s);									//用c类型字符串s赋值`
> `string &assign(const char *s,int n);							//用c字符串s开始的n个字符赋值`
> `string &assign(const string &s);								//把字符串s赋给当前字符串`
> `string &assign(int n,char c);									//用n个字符c赋值给当前字符串`
> `string &assign(const string &s,int start,int n);				//把字符串s中从start开始的n个字符赋给当前字符串`
> `string &assign(const_iterator first,const_itertor last);		//把first和last迭代器之间的部分赋给字符串`

```c++
int main(){
    std::string str = "hello world!!!";
    std::string m_str;
    char* c_str = "hello china!!!";
    m_str = str;			//string &operator=(const string &s);
    std::cout << m_str << std::endl;
    m_str.assign(c_str);		//string &assign(const char *s);
    std::cout << m_str << std::endl;
    m_str.assign(c_str,5);		//string &assign(const char *s,int n);
    std::cout << m_str << std::endl;
    m_str.assign(str);			//string &assign(const string &s);
    std::cout << m_str << std::endl;
    m_str.assign(10,'z');		//string &assign(int n,char c);	
    std::cout << m_str << std::endl;
    m_str.assign(str,2,6);		//string &assign(const string &s,int start,int n);	
    std::cout << m_str << std::endl;
    std::string::iterator it = str.begin()+2;
    std::string::iterator iit = str.end()-2;
    m_str.assign(it,iit);		//string &assign(const_iterator first,const_itertor last);
    std::cout << m_str << std::endl;
}
********************************************************************************
hello world!!!
hello china!!!
hello
hello world!!!
zzzzzzzzzz
llo wo
llo world!
```

## **连接**

> `string &operator+=(const string &s);							//把字符串s连接到当前字符串的结尾` 
> `string &append(const char *s);            						//把c类型字符串s连接到当前字符串结尾`
> `string &append(const char *s,int n);							//把c类型字符串s的前n个字符连接到当前字符串结尾`
> `string &append(const string &s);    							//同operator+=()`
> `string &append(const string &s,int pos,int n);					//字符串中从pos开始的n个字符连接到当前字符串的结尾`
> `string &append(int n,char c);        							//在当前字符串结尾添加n个字符c`
> `string &append(const_iterator first,const_iterator last);		//把迭代器first和last之间的部分连接到当前字符串的结尾`

```c++
int main(){
	//std::string str = "zyq,dzy,chz,xxr,kxy,fh,scy,lb,hfg;";
    //std::string str1 = "123";
    std::string str = "hello world!!!";
    std::string m_str = "boom shakalaka";
    char* c_str = "hello china!!!";
    str += m_str;			//string &operator+=(const string &s);
    std::cout << str << std::endl;
    str.append(c_str);		//string &append(const char *s);     
    std::cout << str << std::endl;
    str.append(c_str,5);	//string &append(const char *s,int n);
    std::cout << str << std::endl;
    str.append(m_str);		//string &append(const string &s); 
    std::cout << str << std::endl;
    str.append(m_str,4,3);	//string &append(const string &s,int pos,int n);	
    std::cout << str << std::endl;
    str.append(5,'z');		//string &append(int n,char c);   
    std::cout << str << std::endl;
    std::string::iterator it = str.begin()+2;
    std::string::iterator iit = str.end()-2;
    str.append(it,iit);		//string &append(const_iterator first,const_iterator last);
    std::cout << str << std::endl;
}
*************************************************************************************
hello world!!!boom shakalaka
hello world!!!boom shakalakahello china!!!
hello world!!!boom shakalakahello china!!!hello
hello world!!!boom shakalakahello china!!!helloboom shakalaka
hello world!!!boom shakalakahello china!!!helloboom shakalaka sh
hello world!!!boom shakalakahello china!!!helloboom shakalaka shzzzzz
hello world!!!boom shakalakahello china!!!helloboom shakalaka shzzzzzllo world!!!boom shakalakahello china!!!helloboom shakalaka shzzz
```

## **比较**

> `bool operator==(const string &s1,const string &s2)const;		//比较两个字符串是否相等`
> `运算符”>”,”<”,”>=”,”<=”,”!=”均被重载用于字符串的比较；`
> `int compare(const string &s) const;							//比较当前字符串和s的大小`
> `int compare(int pos, int n,const string &s)const;				//比较当前字符串从pos开始的n个字符组成的字符串与s的大小`
> `int compare(int pos, int n,const string &s,int pos2,int n2)const;`
> `int compare(const char *s) const;`
> `int compare(int pos, int n,const char *s) const;`
> `int compare(int pos, int n,const char *s, int pos2) const;`
> `compare函数在>时返回1，<时返回-1，==时返回0` 

```c++
int main(){
    std::string str = "hello world!!!";
    std::string m_str = "boom shakalaka";
    char* c_str = "hello china!!!";

    int i = str.compare(m_str);				//int compare(const string &s) const;
    std::cout << i << std::endl;
    bool com = std::operator==(str,m_str);		//bool operator==(const string &s1,const string &s2)const;
    std::cout << com << std::endl;
    int ii = str.compare(2,6,m_str);		//int compare(int pos, int n,const string &s)const;
    std::cout << ii << std::endl;
    int iii = str.compare(0,5,m_str,0,5);	//int compare(int pos, int n,const string &s,int pos2,int n2)const;
    std::cout << iii << std::endl;
    int iiii = str.compare(c_str);			//int compare(const char *s) const;
    std::cout << iiii << std::endl;
    int iiiii = str.compare(0,5,c_str);		//int compare(int pos, int n,const char *s) const;
    std::cout << iiiii << std::endl;
    int iiiiii = str.compare(0,5,c_str,0,5);	//int compare(int pos, int n,const char *s, int pos2) const;
    std::cout << iiiiii << std::endl;
}
****************************************************************************
1
0
1
1
1
-9
0   
```

## **交换**

> `void swap(string &s2); //交换当前字符串与s2的值`

```c++
int main(){
    std::string str = "hello world!!!";
    std::string m_str = "boom shakalaka";
    char* c_str = "hello china!!!";
    swap(str,m_str);
    std::cout << str << std::endl;
    std::cout << m_str << std::endl;
}
****************************************************************************
boom shakalaka
hello world!!!
```

## **查找 **

> `int find(char c, int pos = 0) const;					//从pos开始查找字符c在当前字符串的位置`
> `int find(const char *s, int pos = 0) const;			//从pos开始查找字符串s在当前串中的位置`
> `int find(const char *s, int pos, int n) const;			//从pos开始查找字符串s中前n个字符在当前串中的位置`
> `int find(const string &s, int pos = 0) const;			//从pos开始查找字符串s在当前串中的位置`
> `//查找成功时返回所在位置，失败返回string::npos的值`
> `int rfind(char c, int pos = npos) const;				//从pos开始从后向前查找字符c在当前串中的位置`
> `int rfind(const char *s, int pos = npos) const;`
> `int rfind(const char *s, int pos, int n = npos) const;`
> `int rfind(const string &s,int pos = npos) const;`
> `//从pos开始从后向前查找字符串s中前n个字符组成的字符串在当前串中的位置，成功返回所在位置，失败时返回string::npos的值`
> `int find_first_of(char c, int pos = 0) const;			//从pos开始查找字符c第一次出现的位置`
> `int find_first_of(const char *s, int pos = 0) const;`
> `int find_first_of(const char *s, int pos, int n) const;`
> `int find_first_of(const string &s,int pos = 0) const;`
> `//从pos开始查找当前串中第一个在s的前n个字符组成的数组里的字符的位置。查找失败返回string::npos`
> `int find_first_not_of(char c, int pos = 0) const;`
> `int find_first_not_of(const char *s, int pos = 0) const;`
> `int find_first_not_of(const char *s, int pos,int n) const;`
> `int find_first_not_of(const string &s,int pos = 0) const;`
> `//从当前串中查找第一个不在串s中的字符出现的位置，失败返回string::npos`
> `int find_last_of(char c, int pos = npos) const;`
> `int find_last_of(const char *s, int pos = npos) const;`
> `int find_last_of(const char *s, int pos, int n = npos) const;`
> `int find_last_of(const string &s,int pos = npos) const;`
> `int find_last_not_of(char c, int pos = npos) const;`
> `int find_last_not_of(const char *s, int pos = npos) const;`
> `int find_last_not_of(const char *s, int pos, int n) const;`
> `int find_last_not_of(const string &s,int pos = npos) const;`
> `//find_last_of和find_last_not_of与find_first_of和find_first_not_of相似，只不过是从后向前查找`

`string::find()`函数：是一个字符或字符串查找函数，该函数有唯一的返回类型，即`string::size_type`，即一个无符号整形类型，可能是整数也可能是长整数。如果查找成功，返回按照查找规则找到的第一个字符或者子串的位置；如果查找失败，返回`string::npos`，即`-1`（当然打印出的结果不是`-1`，而是一个很大的数值，那是因为它是无符号的）。

`string::npos`静态成员常量：是对类型为`size_t`的元素具有最大可能的值。当这个值在字符串成员函数中的长度或者子长度被使用时，该值表示 "直到字符串结尾" 。作为返回值他通常被用作表明没有匹配。`string::npos`是这样定义的：`static const size_type npos = -1`；

```c++
//在示例字符串中找到zyq这个子串
#include<iostream>
#include<string>
int main(){
	std::string str = "  021,h f g ,lsdk,jk  ,ksjh  ;zyq and chz and dzy mmd";
	std::string::size_type index = str.find("zyq");
	if(index != str.npos){
        std::string result;
        result.push_back(str[index]);
        result.push_back(str[index+1]);
        result.push_back(str[index+2]);
        std::cout << result << std::endl;
	}
	else{
        std::cout << "not find this string ! " << std::endl;
	}
	return 0;	 
}
```

## **删除**

> `iterator erase(iterator first, iterator last);		//删除[first，last）之间的所有字符，返回删除后迭代器的位置`
> `iterator erase(iterator it);						//删除it指向的字符，返回删除后迭代器的位置`
> `string &erase(int pos = 0, int n = npos);			//删除pos开始的n个字符，返回修改后的字符串`

标准库类型`string`表示可变长度的字符序列。可以通过`string`类的`erase()`函数来对该字符序列进行删除操作。`erase()`函数共有3种格式，分别用来删除指定位置的字符、删除指定长度的字符串和删除指定范围的字符串。

### 删除指定位置的字符

`erase()`函数删除指定位置字符通过迭代器实现。其中，参数`iterator`是迭代器，即删除迭代器`p`指向的字符。返回值为删除后的字符串。

```c++
std::string str = "  021,h f g ,lsdk,jk  ,ksjh  ;zyq and chz and dzy mmd";
std::string::iterator it = str.begin();
str.erase(it);
```

### 删除指定长度的字符串

`erase()`函数用于删除指定长度的字符串。`string& erase(size_t pos=0, size_t len = npos);`其中，参数`pos`表示要删除字符串的起始位置，其默认值是`0`；`len`表示要删除字符串的长度，其默认值是`string::npos`。返回值是删除后的字符串。

```c++
//从下标为10的元素起（包括10），删除5个元素
std::string str = "  021,h f g ,lsdk,jk  ,ksjh  ;zyq and chz and dzy mmd";
str.erase(10，5); 
```

### 删除指定范围的字符串

`erase()`函数用于删除指定范围的字符串。`iterator erase (iterator first, iterator last);`其中，`first`和`last`分别表示范围的起点和终点。需要注意的是，该范围包含`first`，但是不包含`last`，即该范围是`[first, last)`。`erase()`函数的返回值是删除后的字符串。

```c++
//从第一个元素起删除10个元素
std::string str = "  021,h f g ,lsdk,jk  ,ksjh  ;zyq and chz and dzy mmd";
std::string::iterator it = str.begin();
str.erase(it，it+10);
```

## **替换**

> `string &replace(int p0, int n0,const char *s);							//删除从p0开始的n0个字符，然后在p0处插入串s`
> `string &replace(int p0, int n0,const char *s, int n);					//删除p0开始的n0个字符，然后在p0处插入字符串s的前n个字符`
> `string &replace(int p0, int n0,const string &s);						//删除从p0开始的n0个字符，然后在p0处插入串s`
> `string &replace(int p0, int n0,const string &s, int pos, int n);		//删除p0开始的n0个字符，然后在p0处插入串s中从pos开始的n个字符`
> `string &replace(int p0, int n0,int n, char c);							//删除p0开始的n0个字符，然后在p0处插入n个字符c`
> `string &replace(iterator first0, iterator last0,const char *s);		//把[first0，last0）之间的部分替换为字符串s`
> `string &replace(iterator first0, iterator last0,const char *s, int n);	//把[first0，last0）之间的部分替换为s的前n个字符`
> `string &replace(iterator first0, iterator last0,const string &s);		//把[first0，last0）之间的部分替换为串s`
> `string &replace(iterator first0, iterator last0,int n, char c);		//把[first0，last0）之间的部分替换为n个字符c`
> `string &replace(iterator first0, iterator last0,const_iterator first, const_iterator last);		//把[first0，last0）之间的部分替换成[first，last）之间的字符串`

## **插入**

> `string &insert(int p0, const char *s);`
> `string &insert(int p0, const char *s, int n);`
> `string &insert(int p0,const string &s);`
> `string &insert(int p0,const string &s, int pos, int n);`
> `//前4个函数在p0位置插入字符串s中pos开始的前n个字符`
> `string &insert(int p0, int n, char c);										//此函数在p0处插入n个字符c`
> `iterator insert(iterator it, char c);										//在it处插入字符c，返回插入后迭代器的位置`
> `void insert(iterator it, const_iterator first, const_iterator last);		//在it处插入[first，last）之间的字符`
> `void insert(iterator it, int n, char c);									//在it处插入n个字符c`

## **迭代器处理**

> `string类提供了向前和向后遍历的迭代器iterator，迭代器提供了访问各个字符的语法，类似于指针操作，迭代器不检查范围。`
> `用string::iterator或string::const_iterator声明迭代器变量，const_iterator不允许改变迭代的内容。常用迭代器函数有：`
> `const_iterator begin()const;`
> `iterator begin(); //返回string的起始位置`
> `const_iterator end()const;`
> `iterator end(); //返回string的最后一个字符后面的位置`
> `const_iterator rbegin()const;`
> `iterator rbegin(); //返回string的最后一个字符的位置`
> `const_iterator rend()const;`
> `iterator rend(); //返回string第一个字符位置的前面`
> `rbegin和rend用于从后向前的迭代访问，通过设置迭代器string::reverse_iterator,string::const_reverse_iterator实现`

# string实例操作

## 实例1：删除一个字符串中所有的空格以及第一个；后面的所有元素

```c++
#include<iostream>
#include<string>
void ClearSpace(std::string &str){
	if(!str.empty()){
		std::string::size_type index; 
		while((index = find(' ')) != str.npos){
			str.erase(index,1);
		}
	}
}

void ClearSpecial(std::string &str){
	if(!str.empty()){
		std::string::size_type index = str.find(';');
		if(index != str.npos){
			str.erase(index+1,str.size()-index-1);
		}
	}
}

int main(){
	std::string str = "  021,h f g ,lsdk,jk  ,ksjh  ;zyq and chz and dzy mmd";
	std::cout << str << std::endl;
	ClearSpace(str);
	std::cout << str << std::endl;
	ClearSpecial(str);
	std::cout << str << std::endl;
	return 0;
}
```

## 实例2：解析csv，去除 ,分行输出

```c++
//给定csv格式字符串"zyq,dzy,chz,xxr,kxy,fh,scy,lb,hfg;"
#include<iostream>
#include<string>
void analysis(std::string &str){
    std::string::size_type index; 
    while((index = str.find(',')) != str.npos){
        str.replace(index,1,"\n");
    }
}

int main(){
	std::string str = "zyq,dzy,chz,xxr,kxy,fh,scy,lb,hfg;";
	std::cout << str << std::endl;
	analysis(str);
	std::cout << str << std::endl;
	return 0;
} 
```

