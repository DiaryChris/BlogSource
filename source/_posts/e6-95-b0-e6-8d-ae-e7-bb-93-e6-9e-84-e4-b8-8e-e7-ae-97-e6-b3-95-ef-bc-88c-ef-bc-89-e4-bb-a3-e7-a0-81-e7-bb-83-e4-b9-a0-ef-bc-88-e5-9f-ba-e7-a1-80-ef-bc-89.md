---
title: 数据结构与算法（C++）代码练习（基础）
url: 159.html
id: 159
categories:
  - 算法
date: 2018-03-10 22:52:01
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 递推法求sinx

 

```c++
#include<iostream>
#include<math.h>
using namespace std;
float sinx(float x);
void main()
{
    float x;
    cout << "请输入x：";
    cin >> x;
    if (fabs(x) >= 1)
    {
        cout << "x大于等于1，数值非法";
    }
    else
    {
    cout << "sin " << x << " = " << sinx(x);
    }
    system("pause");
}
float sinx(float x)
{
    float term = x, result = x;
    for (int n = 1;; n++)
    {
        term = -term\*x\*x / ((2 * n)*(2 * n + 1));
        if (fabs(term)<0.0001)
        {
            break;
        }
        result += term;
    }
    return result;
}
```

   

#### 递推法输出斐波那契数列前20项

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
void main()
{
    long int f1,f2,f3;
    f1 = f2 = 1;
    cout << setw(10) << f1 << setw(10) << f2;
    for (int i = 3; i <= 20; i++)
    {
        f3 = f1 + f2;
        cout << setw(10) << f3;
        f1 = f2;
        f2 = f3;
        if (i % 5 == 0)
        {
            cout << "\n";
        }
    }
    system("pause");
}
```

   

#### 欧几里得算法求两正整数最大公约数

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
void main()
{
    int a, b,a0,b0;
    int temp;
    cout << "请输入两整数的值";
    cin >> a >> b;
    a0 = a, b0 = b;
    if (a <= 0 || b <= 0) {
        cout << "只能计算正整数！\n";
    }
    else {
        while (a%b != 0) {
            temp = a%b;
            a = b;
            b = temp;
        }
        cout << a0 << "与" << b0 << "的最大公约数为" << b << "\n";
    }
    system("pause");
}
```

   

#### 递推法求n阶乘

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
void main()
{
    int n;
    int result=1;
    cout << "请输入正整数n\n";
    cin >> n;
    if (n <= 0) {
        cout << "不是正整数！";
    }
    else {
        for (; n > 0; n--) {
            result *= n;
        }
        cout << "n！=" << result << "\n";
    }
    system("pause");
}
```

   

#### 递归法求n阶乘

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
int x(int n)
{
    if (n != 0) {
        return n*x(n - 1);
    }
    return 1;
}
void main()
{
    int n;
    int result;
    cout << "请输入正整数n\n";
    cin >> n;
    if (n <= 0) {
        cout << "不是正整数！";
    }
    else {
        result=x(n);
        cout << "n！=" << result << "\n";
    }
    system("pause");
}
```

   

#### 递归法输出斐波那契数列前20项

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
int fib(int n)
{
    if (n == 1 || n == 2) {
        return 1;
    }
    return fib(n - 1) + fib(n - 2);
}
void main()
{
    for (int i = 1; i <= 20; i++)
    {
        cout << setw(10) << fib(i);
        if (i % 5 == 0) {
            cout << "\n";
        }
    }
    system("pause");
}
```

   

#### 汉诺塔问题

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
void hanio(int n ,char a , char b , char c ,int &i)
{
    if (n == 0) {
        return;
    }
    hanio(n - 1, a, c, b, i);
    cout << "移动第" << n << "个盘 从" << a << "柱到" << c << "柱\n";
    i++;
    hanio(n - 1, b, a, c, i);
    return;
}
void main()
{
    int n, steps=0;
    cout << "请输入汉诺塔总盘数：";
    cin >> n;
    hanio(n,'A','B','C',steps);
    cout << "总步数：" << steps << "步\n";
    system("pause");
}
```

   

#### 筛法求3-100以内素数

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
void main()
{
    int prime[49];
    for (int i = 0; i < 49; i++) {
        prime[i] = 3 + 2 * i;
    }
    for (int i = 0; i < 4; i++) {
        if (prime[i]) {
            for (int j = i + 1; j < 49; j++) {
                if (prime[j] % prime[i] == 0) {
                    prime[j] = 0;
                }
            }
        }
    }
    int j = 0;
    for (int i = 0; i < 49; i++) {
        if (prime[i] != 0) {
            cout << prime[i] << setw(10);
            j++;
        }
        if (j != 0 && j % 5 == 0) {
            cout << "\n";
        }
    }
    system("pause");
}
```

   

#### 输入成绩并输出成绩低于平均成绩的学生

 

```c++
#include<iostream>
#include<iomanip>
#define N 100
using namespace std;
void main()
{
    float x[N], sum = 0, ave;
    int i = 0;
    cout << "请输入学生成绩，以负数结束\n";
    do {
        cin >> x[i];
        sum += x[i];
    } while (x[i++] >= 0 && i < N);
    if (i == N) {
        ave = sum / i;
    }
    else {
        ave = (sum - x[--i]) / i;
    }
    cout << "共输入" << i << "个学生    平均成绩" << ave << "分\n\n";
    cout << "其中低于平均成绩的学生有：";
    for (int j = 0; j < i; j++) {
        if (x[j] < ave) {
            cout << "第" << j+1 << "个学生    " << x[j] << "分\n";
        }
    }
    system("pause");
}
```

   

#### 打印10层杨辉三角形

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void main()
{
    int a[N][N];
    for (int i = 0; i < N; i++) {
        for (int j = 0; j <= i; j++) {
            if (j == 0 || j == i) {
                a[i][j] = 1;
            }
            else {
                a[i][j] = a[i - 1][j] + a[i - 1][j - 1];
            }
            cout << a[i][j] << setw(10);
        }
        cout << "\n";
    }
    system("pause");
}
```

      



2016年3月