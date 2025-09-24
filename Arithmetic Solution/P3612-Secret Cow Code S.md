# 题解

```c++
#include<iostream>
#include<string>
using namespace std;

int main() {
	string s; cin >> s;
	long long n; cin >> n;
	//确定len, len的大小为 len/2< n <=len ,而且len为 m.length()的倍数
	long long len = s.length();
	while (len < n) len <<= 1;
    
	while (n > s.length()) {
		len >>= 1;//提前处理
		if (n > len)//n在字符串的后半段的情况,
		{
			if (n == len + 1) {
				n = len;
			}
			else {
				n -= len + 1;
			}
		}
		else {//n在字符串的前半段,只需len/2即可,因为对n操作后,会出现这种情况
			continue;
		}
	}
	cout << s[n - 1];
	return 0;
}
```

题目要求翻译为: 先复制最后一个字符,在复制它的前缀,以此为思路,

分成两种情况

1. ==最后一个字符==,
2. 他的==前缀==

这两种请况都是为复制后的前半部分,后半部分为分为==第一个字符==和他的==后缀==

通过不断的==缩小==n的大小到原字符串的范围内

1. 当n在字符串的后半部分时
   * n为后半部分的第一位,那么缩小的方式为(把n变为前半部分的最后一位),即n变为len/2;
   * n在(后半部分的)的第一位的后缀,n变为 n-len-1;

2. 当n在字符串的前半部分时 把len/2,以保证n在len的后半部分

## 三目运算符简化

只要是$if()\{\}else\{\}$ 情况都可以变为$( ? :)$ 

### 第一个简化

```c++
if (n == len + 1) {
	n = len;
}
else {
    n -= len + 1;
}
```

简化为

```c++
n = (n == len - 1 ? n = len : n - len - 1);
```

### 第二个简化

```c++
if (n > len){
	n = (n == len - 1 ? n = len : n - len - 1);//第一个简化
}
else {
	continue;
}
```

简化为

```c++
n = (n > len ? (n == len - 1 ? n = len : n - len - 1) : n);
```

### 最终为

```c++
#include<iostream>
#include<string>
using namespace std;

int main() {
	string s; cin >> s;
	long long n; cin >> n;
	long long len = s.length();
	while (len < n) len <<= 1;
	while (n > s.length()) {
		len >>= 1;
		n = (n > len ? (n == len - 1 ? n = len : n - len - 1) : n);
	}
	cout << s[n - 1];
	return 0;
}
```

