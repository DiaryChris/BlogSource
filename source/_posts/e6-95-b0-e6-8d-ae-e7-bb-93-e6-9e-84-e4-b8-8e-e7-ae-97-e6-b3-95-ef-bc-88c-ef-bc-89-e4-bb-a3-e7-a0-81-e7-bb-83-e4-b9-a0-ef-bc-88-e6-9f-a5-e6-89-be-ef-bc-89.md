---
title: 数据结构与算法（C++）代码练习（查找）
url: 155.html
id: 155
categories:
  - 算法
date: 2018-03-10 22:30:24
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 20个随机数的数组中的顺序查找

 

```c++
#include<iostream>
#include<iomanip>
#include<stdlib.h>
#include<time.h>
#define N 20
using namespace std;
void Order_Search(int a[], int n)
{
    for (int i=0;i<N;i++){
        if (a[i] == n) {
            cout << "在第" <<i+1 << "个位置上找到" << n << "\n";
            return;
        }
    }
    cout << "未找到\n";
    return;
}
void main()
{
    int a[N],n;
    srand(unsigned(time));
    for (int i = 0; i < N; i++)
    {
        a[i] = rand();
        cout << a[i] << '\t';
    }
    cout << "\n请输入要查找的数：";
    cin >> n;
    cout << "\n\n";
    Order_Search(a,n);
    system("pause");
}

   
```



#### 20个随机数的数组中的二分查找

 

```c++
#include<iostream>
#include<iomanip>
#include<stdlib.h>
#include<time.h>
#define N 20
using namespace std;
void Bubble_Sort(int a[])
{
    int temp, flag = 1;
    for (int j = N - 1; j > 0 && flag; j--) {
        flag = 0;
        for (int i = 0; i < j; i++) {
            if (a[i] > a[i + 1]) {
                temp = a[i];
                a[i] = a[i + 1];
                a[i + 1] = temp;
                flag = 1;
            }
        }
    }
}
void Binary_Search(int a[], int n)
{
    int left = 0, right = N, mid;
    while (left <= right) {
        mid = (left + right) / 2;
        if (a[mid] > n) {
            right = mid-1;
        }
        else if (a[mid] < n) {
            left = mid+1;
        }
        else {
            cout << "在升序第" << mid + 1 << "个位置上找到" << n << "\n";
            return;
        }
    }
    cout << "未找到\n";
    return;
}
void main()
{
    int a[N], n;
    srand(unsigned(time));
    for (int i = 0; i < N; i++)
    {
        a[i] = rand();
        cout << a[i] << '\t';
    }
    cout << "\n请输入要查找的数：";
    cin >> n;
    cout << "\n\n";
    Bubble_Sort(a);
    Binary_Search(a, n);
    system("pause");
}
```

   

#### 使用平方探测的散列表的插入、删除、查找

 

```c++
#include<iostream>
#include<iomanip>
#define N 43
#define I -100000
#define HOLD 100000
using namespace std;
int Hash(int key)
{
    return key%N;
}
int findPos(int key, int table[])
{
    int n = 0;
    int pos, ipos;
    pos = ipos = Hash(key);
    while (table[pos]!=I && table[pos] != key) {
        if (++n % 2) {
            pos = ipos + (n + 1) / 2 * (n + 1) / 2;
            while (pos >= N) {
                pos -= N;
            }
        }
        else {
            pos = ipos - (n / 2)*(n / 2);
            while (pos < 0) {
                pos += N;
            }
        }
    }
    return pos;
}
int contain(int key, int table[])
{
    int pos=findPos(key, table);
    if (table[pos]==I) {
        return 0;
    }
    else {
        return 1;
    }
}
void insert(int key, int table[])
{
    int pos = findPos(key, table);
    table[pos] = key;
}
int remove(int key, int table[])
{
    int pos = findPos(key, table);
    if (table[pos]==I) {
        return 0;
    }
    else {
        table[pos] = HOLD;
        return 1;
    }
}
void main()
{
    int n, a, res;
    int hashTable[N];
    for (int i = 0; i < N; i++)
    {
        hashTable[i] = I;
    }
    cout << "请输入要插入的数据个数：\n";
    cin >> n;
    cout << "请依次输入数据：\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        insert(a, hashTable);
    }
    cout << "请输入要删除的数据个数：\n";
    cin >> n;
    cout << "请依次输入要删除的数据：\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = remove(a, hashTable);
        cout << (res ? "删除成功\n" : "查无此数\n");
    }
    cout << "请输入要查找的数据个数：\n";
    cin >> n;
    cout << "请依次输入要查找的数据：\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = contain(a, hashTable);
        cout << (res ? "已找到\n" : "未找到\n");
    }
    system("pause");
}
```

      



2016年3月