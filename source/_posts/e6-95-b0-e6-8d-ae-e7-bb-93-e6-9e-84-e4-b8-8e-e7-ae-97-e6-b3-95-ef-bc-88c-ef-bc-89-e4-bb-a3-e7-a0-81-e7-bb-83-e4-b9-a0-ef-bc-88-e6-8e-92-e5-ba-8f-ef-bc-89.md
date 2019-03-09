---
title: 数据结构与算法（C++）代码练习（排序）
url: 151.html
id: 151
categories:
  - 算法
date: 2018-03-10 21:40:13
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 插入排序（Insertion_Sort）

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void Insertion_Sort(int a[], int n)
{
    int temp;
    for (int i = 1; i < n; i++) {
        int j = i - 1;
        temp = a[i];
        while (j >= 0 && a[j] > temp) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = temp;
    }
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Insertion_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 希尔排序（Shell_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void Shell_Sort(int a[], int n)
{
    int temp;
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i += gap) {
            int j = i - gap;
            temp = a[i];
            while (j >= 0 && a[j] > temp) {
                a[j + gap] = a[j];
                j -= gap;
            }
            a[j + gap] = temp;
        }
    }
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Shell_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 冒泡排序（Bubble_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void Bubble_Sort(int a[], int n)
{
    int temp, flag = 1;
    for (int j = n - 1; j > 0 && flag; j--) {
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
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Bubble_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 快速排序（Quick_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void Insertion_Sort(int a[], int n)
{
    int temp;
    for (int i = 1; i < n; i++) {
        int j = i - 1;
        temp = a[i];
        while (j >= 0 && a[j] > temp) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = temp;
    }
}
void swap(int &a, int &b)
{
    int temp = a;
    a = b;
    b = temp;
}
int median(int a[], int l, int r)
{
    int c = (l + r) / 2;
    if (a[l] > a[c]) {
        swap(a[l], a[c]);
    }
    if (a[c] > a[r]) {
        swap(a[c], a[r]);
    }
    if (a[l] > a[c]) {
        swap(a[l], a[c]);
    }
    swap(a[c], a[r - 1]);
    return a[r - 1];
}
void quickSort(int a[], int left, int right)
{
    if (right - left + 1 > 3) {
        int pivot = median(a, left, right);
        int i, j;
        i = left;
        j = right - 1;
        while (1) {
            while (a[++i] < pivot) {}
            while (a[--j] > pivot) {}
            if (i < j) {
                swap(a[i], a[j]);
            }
            else {
                break;
            }
        }
        swap(a[i], a[right - 1]);
        quickSort(a, left, i - 1);
        quickSort(a, i + 1, right);
    }
    else {
        Insertion_Sort(a + left, right - left + 1);
    }
}
void Quick_Sort(int a[], int n)
{
    quickSort(a, 0, n - 1);
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Quick_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 选择排序（Select_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void Select_Sort(int a[], int n)
{
    int min, temp;
    for (int i = 0; i < n - 1; i++) {
        min = i;
        for (int j = i + 1; j < n; j++) {
            if (a[j] < a[min]) {
                min = j;
            }
        }
        temp = a[i];
        a[i] = a[min];
        a[min] = temp;
    }
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Select_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 堆排序（Heap_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
class Heap
{
public:
    int size;
    int* a;
    Heap() {
        size = 0;
    }
};
void incertMaxHeap(int a, Heap& maxHeap)
{
    int hole = maxHeap.size++;
    while (hole != 0 && a > maxHeap.a[(hole - 1) / 2]) {
        maxHeap.a[hole] = maxHeap.a[(hole - 1) / 2];
        hole = (hole - 1) / 2;
    }
    maxHeap.a[hole] = a;
}
void moveMaxRear(Heap& maxHeap)
{
    int max = maxHeap.a[0];
    int hole = 0;
    while (hole <= maxHeap.size / 2 - 1 && (maxHeap.a[maxHeap.size - 1] < maxHeap.a[hole * 2 + 1] || maxHeap.a[maxHeap.size - 1] < maxHeap.a[hole * 2 + 2])) {
        if (maxHeap.a[hole * 2 + 2] > maxHeap.a[hole * 2 + 1]) {
            maxHeap.a[hole] = maxHeap.a[hole * 2 + 2];
            hole = hole * 2 + 2;
        }
        else {
            maxHeap.a[hole] = maxHeap.a[hole * 2 + 1];
            hole = hole * 2 + 1;
        }
    }
    if (maxHeap.size % 2 == 0 && hole == maxHeap.size) {
        maxHeap.a[maxHeap.size] = maxHeap.a[(hole - 1) / 2];
        hole = (hole - 1) / 2;
    }
    maxHeap.a[hole] = maxHeap.a[maxHeap.size - 1];
    maxHeap.size--;
    maxHeap.a[maxHeap.size] = max;
}
void Heap_Sort(int a[], int n)
{
    Heap maxHeap;
    maxHeap.a = a;
    for (int i = 0; i < n; i++) {
        incertMaxHeap(a[i], maxHeap);
    }
    for (int i = 0; i < n; i++) {
        moveMaxRear(maxHeap);
    }
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Heap_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 归并排序（Merge_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
using namespace std;
void merge(int obj[], int src[], int left, int right, int rEnd)
{
    int lEnd = right - 1;
    int i = left;
    while (left <= lEnd&&right <= rEnd) {
        if (src[left] <= src[right]) {
            obj[i++] = src[left++];
        }
        else {
            obj[i++] = src[right++];
        }
    }
    while (left <= lEnd) {
        obj[i++] = src[left++];
    }
    while (right <= rEnd) {
        obj[i++] = src[right++];
    }
}
void mergeSort(int obj[],int src[],int left,int right)
{
    if (left >= right) {
        return;
    }
    int center = (left + right) / 2;
    mergeSort(src, obj, left, center);
    mergeSort(src, obj, center + 1, right);
    merge(obj, src, left, center + 1, right);
}
void Merge_Sort(int a[], int n)
{
    int* temp = new int[n];
    for (int i = 0; i < n; i++) {
        temp[i] = a[i];
    }
    mergeSort(a, temp, 0, n - 1);
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Merge_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

   

#### 桶排序（Bucket_Sort）

 

```c++
#include<iostream>
#include<iomanip>
#define N 10
#define SIZE 100
using namespace std;
class listNode
{
public:
    int data;
    listNode* next;
    listNode() {
        next = NULL;
    }
};
void addNode(listNode& node, int n)
{
    listNode* p = new listNode;
    p->data = n;
    p->next = node.next;
    node.next = p;
}
void readNode(listNode bucket[], int a[])
{
    int j = 0;
    for (int i = 0; i < SIZE; i++) {
        listNode* p = bucket[i].next;
        while (p) {
            a[j++] = p->data;
            p = p->next;
        }
    }
}
void Bucket_Sort(int a[], int n)
{
    listNode bucket[SIZE];
    for (int i = 0; i < n; i++) {
        addNode(bucket[a[i]], a[i]);
    }
    readNode(bucket, a);
}
void main()
{
    int a[N];
    cout << "请输入要排序的" << N << "个整数(0-99之间)：\n";
    for (int i = 0; i < N; i++) {
        cin >> a[i];
    }
    Bucket_Sort(a, N);
    cout << "排序后：";
    for (int i = 0; i < N; i++) {
        cout << a[i] << "    ";
    }
    system("pause");
}
```

      



2016年3月