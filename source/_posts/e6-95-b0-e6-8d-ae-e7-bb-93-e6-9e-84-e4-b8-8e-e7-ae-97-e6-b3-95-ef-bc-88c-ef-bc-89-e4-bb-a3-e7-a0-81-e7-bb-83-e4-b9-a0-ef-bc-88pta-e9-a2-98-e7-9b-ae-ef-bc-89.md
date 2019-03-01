---
title: 数据结构与算法（C++）代码练习（PTA题目）
url: 163.html
id: 163
categories:
  - 算法
date: 2018-03-10 23:03:03
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 在线算法输出N个整数的最大子列和

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
int maxSubSum(int a\[\], int n)
{
    int thisSum = 0, maxSum = 0;
    for (int i = 0; i < n; i++) {
        thisSum += a\[i\];
        if (thisSum > maxSum) {
            maxSum = thisSum;
        }
        if (thisSum < 0) {
            thisSum = 0;
        }
    }
    return maxSum;
}
void main()
{
    int n, *a, sum;
    cout << "请输入数列长度：\\n";
    cin >> n;
    a = new int\[n\];
    cout << "请输入数列：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a\[i\];
    }
    sum = maxSubSum(a, n);
    cout << "最大子列和为：" << sum << "\\n";
    system("pause");
}
```

   

#### 在线算法输出N个整数的最大子列和及其首尾项

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
int maxSubSumPrint(int a\[\], int n)
{
    int thisSum = 0, maxSum = -1;
    int head, rear, flag = 0;
    int maxHead = 0, maxRear = n - 1;
    for (int i = 0; i < n; i++) {
        thisSum += a\[i\];
        if (flag == 0) {
            if (a\[i\] >= 0) {
                head = rear = i;
                flag = 1;
            }
        }
        else {
            rear++;
        }
        if (thisSum > maxSum) {
            maxSum = thisSum;
            maxHead = head;
            maxRear = rear;
        }
        if (thisSum < 0) {
            thisSum = 0;
            flag = 0;
        }
    }
    if (maxSum == -1) {
        maxSum = 0;
    }
    cout << "最大子列和为：" << maxSum << "\\n";
    cout << "最大子列头数字为" << a\[maxHead\] << "  尾数字为" << a\[maxRear\] << "\\n";
    return maxSum;
}
void main()
{
    int n, *a;
    cout << "请输入数列长度：\\n";
    cin >> n;
    a = new int\[n\];
    cout << "请输入数列：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a\[i\];
    }
    maxSubSumPrint(a, n);
    system("pause");
}
```

   

#### 使用链表输出两个一元多项式的乘积与和

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
class PolyNode
{
public:
    int coef;
    int expo;
    PolyNode *nextNode;
};
PolyNode* inputPoly(int n) {
    if (n <= 0) {
        return NULL;
    }
    PolyNode *poly = new PolyNode;
    PolyNode *p = NULL;
    cout << "请以指数递降顺序输入各项系数与指数（以空格分开）：\\n";
    p = poly;
    cin >> p->coef;
    cin >> p->expo;
    while (--n) {
        p->nextNode = new PolyNode;
        p = p->nextNode;
        cin >> p->coef;
        cin >> p->expo;
    }
    p->nextNode = NULL;
    return poly;
}
PolyNode* add(PolyNode* p1, PolyNode* p2)
{
    PolyNode* p = new PolyNode;
    p->nextNode = new PolyNode;
    PolyNode* p0 = NULL;
    if (p1 || p2) {
        p0 = p->nextNode;
    }
    while (p1&&p2) {
        p = p->nextNode;
        if (p1->expo > p2->expo) {
            p->expo = p1->expo;
            p->coef = p1->coef;
            p1 = p1->nextNode;
        }
        else if (p1->expo < p2->expo) {
            p->expo = p2->expo;
            p->coef = p2->coef;
            p2 = p2->nextNode;
        }
        else {
            p->expo = p1->expo;
            p->coef = p1->coef + p2->coef;
            p1 = p1->nextNode;
            p2 = p2->nextNode;
        }
        p->nextNode = new PolyNode;
    }
    while (p1) {
        p = p->nextNode;
        p->expo = p1->expo;
        p->coef = p1->coef;
        p1 = p1->nextNode;
        p->nextNode = new PolyNode;
    }
    while (p2) {
        p = p->nextNode;
        p->expo = p2->expo;
        p->coef = p2->coef;
        p2 = p2->nextNode;
        p->nextNode = new PolyNode;
    }
    p->nextNode = NULL;
    return p0;
}
PolyNode* mult(PolyNode* p1, PolyNode* p2)
{
    PolyNode *p = NULL;
    PolyNode *t = new PolyNode;
    t->nextNode = NULL;
    PolyNode \*i = p1, \*j = p2;
    while (i) {
        if (i->coef) {
            j = p2;
            while (j) {
                if (j->coef) {
                    t->coef = i->coef*j->coef;
                    t->expo = i->expo + j->expo;
                    p = add(p, t);
                }
                j = j->nextNode;
            }
        }
        i = i->nextNode;
    }
    return p;
}
void printPoly(PolyNode* p)
{
    int flag = 0;
    while (p) {
        if (p->coef) {
            cout << p->coef << ' ' << p->expo << ' ';
            flag = 1;
        }
        p = p->nextNode;
    }
    if (!flag) {
        cout << "0 0";
    }
    cout << "\\n";
}
void main()
{
    int n1, n2;
    PolyNode \*p1, \*p2, \*padd, \*pmult;
    cout << "请输入第一个多项式非零项项数：\\n";
    cin >> n1;
    p1 = inputPoly(n1);
    cout << "请输入第二个多项式非零项项数：\\n";
    cin >> n2;
    p2 = inputPoly(n2);
    padd = add(p1, p2);
    pmult = mult(p1, p2);
    cout << "和多项式的非零项为：\\n";
    printPoly(padd);
    cout << "乘积多项式的非零项为：\\n";
    printPoly(pmult);
    system("pause");
}
```

   

#### 以k为单位反转抽象链表

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
class Node
{
public:
    int Data;
    int Next;
};
void input(Node a\[\], int n)
{
    int i;
    for (int j = 0; j < n; j++) {
        cin >> i;
        cin >> a\[i\].Data >> a\[i\].Next;
    }
}
int count(Node a\[\], int h)
{
    int i = 0, p = h;
    while (p != -1)
    {
        i++;
        p = a\[p\].Next;
    }
    return i;
}
void reverse(Node a\[\], int& h, int k)
{
    int n, i = 0;
    n = count(a, h);
    int p, h1, h2, t1, t2;
    p = h1 = h2 = h;
    t1 = a\[p\].Next;
    t2 = a\[t1\].Next;
    while (++i)
    {
        if (i > n / k) {
            break;
        }
        for (int j = 0; j < k - 1; j++)
        {
            a\[t1\].Next = p;
            p = t1;
            t1 = t2;
            t2 = a\[t2\].Next;
        }
        if (i == 1)
        {
            h = p;
        }
        else {
            a\[h1\].Next = p;
            h1 = h2;
        }
        p = t1;
        t1 = t2;
        t2 = a\[t2\].Next;
        h2 = p;
    }
    a\[h1\].Next = p;
}
void print(Node a\[\], int h)
{
    int p = h;
    while (p != -1 && a\[p\].Next != -1) {
        cout << setfill('0') << setw(5) << p << ' ' << a\[p\].Data << ' ' << setw(5) << a\[p\].Next << '\\n';
        p = a\[p\].Next;
    }
    if (a\[p\].Next == -1) {
        cout << setfill('0') << setw(5) << p << ' ' << a\[p\].Data << ' ' << a\[p\].Next << '\\n';
    }
}
void main()
{
    int head, n, k;
    Node Memory\[100000\];
    cout << "请输入头地址、节点总数、反转单位长度：\\n";
    cin >> head >> n >> k;
    input(Memory, n);
    reverse(Memory, head, k);
    cout << "反转后结果：\\n";
    print(Memory, head);
    system("pause");
}

   
```



#### 判断一组数是否为可能的出栈结果

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
class Node
{
public:
    int Data;
    Node* Next;
};
void push(Node* &top, int& sum, int a)
{
    Node* temp = new Node;
    temp->Next = top;
    top = temp;
    top->Data = a;
    sum++;
}
int pop(Node* &top, int& sum)
{
    Node* temp = top;
    top = top->Next;
    temp->Next = NULL;
    sum--;
    return temp->Data;
}
int check(int m, int n)
{
    int temp = 0, sum = 0;
    int x;
    int result = 1;
    Node* top = new Node;
    top->Next = NULL;
    for (int i = 0; i < n; i++) {
        cin >> x;
        if (x < temp) {
            if (top->Data != x) {
                result = 0;
            }
            else {
                pop(top, sum);
            }
            continue;
        }
        while (++temp != x) {
            push(top, sum, temp);
            if (sum > m - 1) {
                result = 0;
                break;
            }
        }
    }
    return result;
}
void main()
{
    int m, n, k;
    cin >> m >> n >> k;
    int *result;
    result = new int\[k\];
    for (int i = 0; i < k; i++)
    {
        result\[i\] = check(m, n);
    }
    for (int i = 0; i < k; i++)
    {
        switch (result\[i\])
        {
        case 0:cout << "NO\\n";
            break;
        case 1:cout << "YES\\n";
            break;
        }
    }
    system("pause");
}

   
```



#### 递归法判断树的同构

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
class Node
{
public:
    char data;
    int left;
    int right;
    int isRoot;
};
int input(Node *&a)
{
    int n;
    int root;
    char left, right;
    cin >> n;
    if (!n) {
        a = NULL;
        return -1;
    }
    a = new Node\[n\];
    for (int i = 0; i < n; i++)
    {
        cin >> a\[i\].data >> left >> right;
        a\[i\].isRoot = 1;
        if (left == '-') {
            a\[i\].left = -1;
        }
        else {
            a\[i\].left = int(left) - 48;
        }
        if (right == '-') {
            a\[i\].right = -1;
        }
        else {
            a\[i\].right = int(right) - 48;
        }
    }
    for (int i = 0; i < n; i++)
    {
        if (a\[i\].left != -1) {
            a\[a\[i\].left\].isRoot = 0;
        }
        if (a\[i\].right != -1) {
            a\[a\[i\].right\].isRoot = 0;
        }
    }
    for (int i = 0; i < n; i++)
    {
        if (a\[i\].isRoot) {
            root = i;
        }
    }
    return root;
}
int compare(Node* a, Node* b, int aRoot, int bRoot)
{
    if (!a&&!b) {
        return 1;
    }
    else if (!a || !b) {
        return 0;
    }
    else if (aRoot == -1 && bRoot == -1) {
        return 1;
    }
    else if (aRoot == -1 || bRoot == -1) {
        return 0;
    }
    else if (a\[aRoot\].data != b\[bRoot\].data) {
        return 0;
    }
    else if (compare(a, b, a\[aRoot\].left, b\[bRoot\].left)) {
        return compare(a, b, a\[aRoot\].right, b\[bRoot\].right);
    }
    else if (compare(a, b, a\[aRoot\].left, b\[bRoot\].right)) {
        return compare(a, b, a\[aRoot\].right, b\[bRoot\].left);
    }
    else {
        return 0;
    }
}
void main()
{
    Node \*a, \*b;
    int aRoot, bRoot;
    int result;
    aRoot = input(a);
    bRoot = input(b);
    result = compare(a, b, aRoot, bRoot);
    if (result == 1) {
        cout << "Yes";
    }
    else {
        cout << "No";
    }
    system("pause");
}

   
```



#### 层序遍历打印叶子结点

 

```c++
#include<iostream>
#include<iomanip>
using namespace std;
class Node
{
public:
    int left;
    int right;
    int isRoot;
    int isLeaf;
};
class QueueNode
{
public:
    int a;
    QueueNode* next;
};
int input(Node *&a)
{
    int n;
    int root;
    char left, right;
    cin >> n;
    if (!n) {
        a = NULL;
        return -1;
    }
    a = new Node\[n\];
    for (int i = 0; i < n; i++)
    {
        cin >> left >> right;
        a\[i\].isRoot = 1;
        a\[i\].isLeaf = 1;
        if (left == '-') {
            a\[i\].left = -1;
        }
        else {
            a\[i\].left = int(left) - int('0');
        }
        if (right == '-') {
            a\[i\].right = -1;
        }
        else {
            a\[i\].right = int(right) - int('0');
        }
    }
    for (int i = 0; i < n; i++)
    {
        if (a\[i\].left != -1) {
            a\[a\[i\].left\].isRoot = 0;
            a\[i\].isLeaf = 0;
        }
        if (a\[i\].right != -1) {
            a\[a\[i\].right\].isRoot = 0;
            a\[i\].isLeaf = 0;
        }
    }
    for (int i = 0; i < n; i++)
    {
        if (a\[i\].isRoot) {
            root = i;
        }
    }
    return root;
}
void enqueue(int a, QueueNode*& rear) {
    rear->a = a;
    rear->next = new QueueNode;
    rear = rear->next;
    rear->next = NULL;
}
int dequeue(QueueNode*& front) {
    QueueNode* temp = front;
    front = front->next;
    temp->next = NULL;
    return temp->a;
}
void printLeaf(Node *tree, int root) {
    QueueNode \*front, \*rear;
    int a, flag = 0;
    front = rear = new QueueNode;
    rear->next = NULL;
    enqueue(root, rear);
    while (front != rear) {
        a = dequeue(front);
        if (tree\[a\].isLeaf) {
            if (!flag) {
                cout << a;
                flag = 1;
            }
            else {
                cout << ' ' << a;
            }
        }
        if (tree\[a\].left != -1) {
            enqueue(tree\[a\].left, rear);
        }
        if (tree\[a\].right != -1) {
            enqueue(tree\[a\].right, rear);
        }
    }
}
void main()
{
    Node *tree;
    int root;
    root = input(tree);
    printLeaf(tree, root);
    system("pause");
}
```

      



2016年3月