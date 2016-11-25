title: codeforce652B
date: 2016-02-19 02:11:10
categories:
  - ACM/ICPC
tags:
  - KMP
  - 字符串匹配
---

寒假颓废了好久，准备回归正轨，奈何算法早已生疏，于是随便拿到水题来练练手orz。

<!--more-->
题目[戳这里][link1]

##题意

题目大意是让你求B串在A串中的出现次数，毫不犹豫的想到了**KMP**

##思路

众所周知，KMP是解决字符串匹配的一个优秀算法，如果还未了解，可以参考这篇博客**[从头到尾彻底理解KMP][link2]**。

在该算法中，返回的是第一次匹配到的B串在A串中的首地址。如果要求一共出现了几次，很容易想到，将匹配时的比较稍作更改，增加一个变量记录次数即可。

##解题

``` cpp
#include <cstdio>
#include <cstring>
#include <iostream>

using namespace std;

int Next[35];
char s[100000], p[35];

void GetNext()  
{  
    int pLen = strlen(p);  
    Next[0] = -1;  
    int k = -1;  
    int j = 0;  
    while (j < pLen - 1)  
    {  
        if (k == -1 || p[j] == p[k]) ++k, ++j, Next[j] = k;  
        else k = Next[k];      
    }  
}  

int KmpSearch() {  
    int i = 0;  
    int j = 0;  
    int sLen = strlen(s);  
    int pLen = strlen(p);
	int cnt = 0;  
    while (i < sLen)  
    {  
        if (j == -1 || s[i] == p[j]) i++, j++;      
        else j = Next[j];    
        if(j == pLen) ++cnt,j=0;
    }  
 	return cnt;
}  

int main(){
	scanf("%s%s", s, p);
	GetNext();
  	printf("%d\n",KmpSearch());

}

```

##小结
该题为kmp的一个变种，但是通过该题，不难想到更多的变种，例如求A串所有子串在B中的次数，只需利用A串的next数组进行dp即可[HDOJ3336 Count the string ][link3],求串的最短周期串等等，在此就不一一例举了。

[link1]:http://codeforces.com/problemset/problem/625/B
[link2]:http://blog.csdn.net/v_july_v/article/details/7041827
[link3]:http://acm.hdu.edu.cn/showproblem.php?pid=3336
