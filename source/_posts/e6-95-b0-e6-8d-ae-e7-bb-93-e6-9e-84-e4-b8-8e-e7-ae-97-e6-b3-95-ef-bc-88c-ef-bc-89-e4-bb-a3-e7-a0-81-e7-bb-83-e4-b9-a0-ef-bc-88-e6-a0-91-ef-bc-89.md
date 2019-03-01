---
title: 数据结构与算法（C++）代码练习（树）
url: 153.html
id: 153
categories:
  - 算法
date: 2018-03-10 22:23:45
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 递归法实现二叉树的三种遍历

 

#include<iostream>
#include<iomanip>
using namespace std;
class treeNode
{
public:
    int data;
    treeNode* left;
    treeNode* right;
};
int max(int a, int b)
{
    return (a > b ? a : b);
}
int height(treeNode* root)
{
    if (!root) {
        return -1;
    }
    else {
        return max(height(root->left), height(root->right)) + 1;
    }
}
void singleRotateLeft(treeNode*& root)
{
    treeNode* temp = root;
    root = root->left;
    temp->left = root->right;
    root->right = temp;
}
void singleRotateRight(treeNode*& root)
{
    treeNode* temp = root;
    root = root->right;
    temp->right = root->left;
    root->left = temp;
}
void doubleRotateLeft(treeNode*& root)
{
    singleRotateRight(root->left);
    singleRotateLeft(root);
}
void doubleRotateRight(treeNode*& root)
{
    singleRotateLeft(root->right);
    singleRotateRight(root);
}
treeNode* findMin(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->left) {
        root = root->left;
    }
    return root;
}
treeNode* findMax(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->right) {
        root = root->right;
    }
    return root;
}
int incertNode(int a, treeNode*& root)
{
    int res;
    if (!root) {
        root = new treeNode;
        root->data = a;
        root->left = root->right = NULL;
        return 1;
    }
    if (a < root->data) {
        res = incertNode(a, root->left);
        if (height(root->left) - height(root->right) >= 2) {
            if (a < root->left->data) {
                singleRotateLeft(root);
            }
            else {
                doubleRotateLeft(root);
            }
        }
        return res;
    }
    else if (a > root->data) {
        res = incertNode(a, root->right);
        if (height(root->right) - height(root->left) >= 2) {
            if (a > root->right->data) {
                singleRotateRight(root);
            }
            else {
                doubleRotateRight(root);
            }
        }
        return res;
    }
    else {
        return 0;
    }
}
void preorderTraverse(treeNode* root)
{
    if (root) {
        cout << root->data;
        preorderTraverse(root->left);
        preorderTraverse(root->right);
    }
}
void inorderTraverse(treeNode* root)
{
    if (root) {
        inorderTraverse(root->left);
        cout << root->data;
        inorderTraverse(root->right);
    }
}
void postorderTraverse(treeNode* root)
{
    if (root) {
        postorderTraverse(root->left);
        postorderTraverse(root->right);
        cout << root->data;
    }
}
void main()
{
    int n, a, res;
    treeNode* root = NULL;
    cout << "请输入二叉树结点总数：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = incertNode(a, root);
        cout << (res ? "插入成功\\n" : "数据重复\\n");
    }
    cout << "二叉树构建完毕，高度为：" << height(root) << '\\n';
    cout << "\\n前序遍历：\\n";
    preorderTraverse(root);
    cout << "\\n中序遍历：\\n";
    inorderTraverse(root);
    cout << "\\n后序遍历：\\n";
    postorderTraverse(root);
    system("pause");
}

   

#### 非递归法实现二叉树的中序遍历

 

#include<iostream>
#include<iomanip>
using namespace std;
class treeNode
{
public:
    int data;
    treeNode* left;
    treeNode* right;
};
class Node
{
public:
    treeNode* p;
    Node* next;
};
void push(Node* &top, treeNode* p)
{
    Node* temp = new Node;
    temp->next = top;
    top = temp;
    top->p = p;
}
treeNode* pop(Node* &top)
{
    Node* temp = top;
    top = top->next;
    temp->next = NULL;
    return temp->p;
}
int max(int a, int b)
{
    return (a > b ? a : b);
}
int height(treeNode* root)
{
    if (!root) {
        return -1;
    }
    else {
        return max(height(root->left), height(root->right)) + 1;
    }
}
void singleRotateLeft(treeNode*& root)
{
    treeNode* temp = root;
    root = root->left;
    temp->left = root->right;
    root->right = temp;
}
void singleRotateRight(treeNode*& root)
{
    treeNode* temp = root;
    root = root->right;
    temp->right = root->left;
    root->left = temp;
}
void doubleRotateLeft(treeNode*& root)
{
    singleRotateRight(root->left);
    singleRotateLeft(root);
}
void doubleRotateRight(treeNode*& root)
{
    singleRotateLeft(root->right);
    singleRotateRight(root);
}
treeNode* findMin(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->left) {
        root = root->left;
    }
    return root;
}
treeNode* findMax(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->right) {
        root = root->right;
    }
    return root;
}
int incertNode(int a, treeNode*& root)
{
    int res;
    if (!root) {
        root = new treeNode;
        root->data = a;
        root->left = root->right = NULL;
        return 1;
    }
    if (a < root->data) {
        res = incertNode(a, root->left);
        if (height(root->left) - height(root->right) >= 2) {
            if (a < root->left->data) {
                singleRotateLeft(root);
            }
            else {
                doubleRotateLeft(root);
            }
        }
        return res;
    }
    else if (a > root->data) {
        res = incertNode(a, root->right);
        if (height(root->right) - height(root->left) >= 2) {
            if (a > root->right->data) {
                singleRotateRight(root);
            }
            else {
                doubleRotateRight(root);
            }
        }
        return res;
    }
    else {
        return 0;
    }
}
void inorderTraverse(treeNode* root)
{
    Node* stack = new Node;
    stack->next = NULL;
    while (root||stack->next) {
        while (root) {
            push(stack, root);
            root = root->left;
        }
        root = pop(stack);
        cout << root->data;
        root = root->right;
    }
}
void main()
{
    int n, a, res;
    treeNode* root = NULL;
    cout << "请输入二叉树结点总数：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = incertNode(a, root);
        cout << (res ? "插入成功\\n" : "数据重复\\n");
    }
    cout << "二叉树构建完毕，高度为：" << height(root) << '\\n';
    cout << "\\n中序遍历：\\n";
    inorderTraverse(root);
    system("pause");
}

   

#### 平衡二叉树的插入、删除、查找

 

#include<iostream>
#include<iomanip>
using namespace std;
class treeNode
{
public:
    int data;
    treeNode* left;
    treeNode* right;
};
int max(int a, int b)
{
    return (a > b ? a : b);
}
int height(treeNode* root)
{
    if (!root) {
        return -1;
    }
    else {
        return max(height(root->left), height(root->right)) + 1;
    }
}
void singleRotateLeft(treeNode*& root)
{
    treeNode* temp = root;
    root = root->left;
    temp->left = root->right;
    root->right = temp;
}
void singleRotateRight(treeNode*& root)
{
    treeNode* temp = root;
    root = root->right;
    temp->right = root->left;
    root->left = temp;
}
void doubleRotateLeft(treeNode*& root)
{
    singleRotateRight(root->left);
    singleRotateLeft(root);
}
void doubleRotateRight(treeNode*& root)
{
    singleRotateLeft(root->right);
    singleRotateRight(root);
}
treeNode* find(int a, treeNode* root)
{
    if (!root) {
        return NULL;
    }
    if (a < root->data) {
        return find(a, root->left);
    }
    else if (a > root->data) {
        return find(a, root->right);
    }
    else {
        return root;
    }
}
treeNode* findMin(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->left) {
        root = root->left;
    }
    return root;
}
treeNode* findMax(treeNode* root)
{
    if (!root) {
        return NULL;
    }
    while (root->right) {
        root = root->right;
    }
    return root;
}
int incertNode(int a, treeNode*& root)
{
    int res;
    if (!root) {
        root = new treeNode;
        root->data = a;
        root->left = root->right = NULL;
        return 1;
    }
    if (a < root->data) {
        res = incertNode(a, root->left);
        if (height(root->left) - height(root->right) >= 2) {
            if (a < root->left->data) {
                singleRotateLeft(root);
            }
            else {
                doubleRotateLeft(root);
            }
        }
        return res;
    }
    else if (a > root->data) {
        res = incertNode(a, root->right);
        if (height(root->right) - height(root->left) >= 2) {
            if (a > root->right->data) {
                singleRotateRight(root);
            }
            else {
                doubleRotateRight(root);
            }
        }
        return res;
    }
    else {
        return 0;
    }
}
int deleteNode(int a, treeNode*& root)
{
    if (!root) {
        return 0;
    }
    if (a < root->data) {
        return deleteNode(a, root->left);
    }
    else if (a > root->data) {
        return deleteNode(a, root->right);
    }
    else if (root->left&&root->right) {
        treeNode* p;
        if (height(root->left) < height(root->right)) {
            p = findMin(root->right);
            root->data = p->data;
            return deleteNode(p->data, root->right);
        }
        else {
            p = findMax(root->left);
            root->data = p->data;
            return deleteNode(p->data, root->left);
        }
    }
    else if (root->left || root->right) {
        treeNode* p = (root->left ? root->left : root->right);
        delete root;
        root = p;
        return 1;
    }
    else {
        delete root;
        root = NULL;
        return 1;
    }
}
void main()
{
    int n, a, res;
    treeNode* root = NULL, *resP = NULL;
    cout << "请输入二叉树结点总数：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = incertNode(a, root);
        cout << (res ? "插入成功\\n" : "数据重复\\n");
    }
    cout << "二叉树构建完毕，高度为：" << height(root) << '\\n';
    cout << "请输入要删除的结点个数：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        res = deleteNode(a, root);
        cout << (res ? "删除成功\\n" : "查无此数\\n");
    }
    cout << "删除完毕，高度为：" << height(root) << '\\n';
    cout << "请输入要查找的结点个数：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        resP = find(a, root);
        cout << (resP ? "已找到\\n" : "未找到\\n");
    }
    system("pause");
}

   

#### 最小堆的插入与删除

 

#include<iostream>
#include<iomanip>
#define N 100
#define MAX 100000
using namespace std;
class Heap
{
public:
    int size;
    int a\[N\];
};
void checkOdd(Heap& minHeap)
{
    if (!(minHeap.size % 2)) {
        minHeap.a\[minHeap.size + 1\] = MAX;
    }
}
void incertMinHeap(int a, Heap& minHeap)
{
    int hole = ++minHeap.size;
    while (hole != 1 && a < minHeap.a\[hole / 2\]) {
        minHeap.a\[hole\] = minHeap.a\[hole / 2\];
        hole = hole / 2;
    }
    minHeap.a\[hole\] = a;
    checkOdd(minHeap);
}
int deleteMin(Heap& minHeap)
{
    int min = minHeap.a\[1\];
    int hole = 1;
    while (hole <= minHeap.size / 2 && (minHeap.a\[minHeap.size\] > minHeap.a\[hole * 2 + 1\] || minHeap.a\[minHeap.size\] > minHeap.a\[hole * 2\])) {
        if (minHeap.a\[hole * 2 + 1\] < minHeap.a\[hole * 2\]) {
            minHeap.a\[hole\] = minHeap.a\[hole * 2 + 1\];
            hole = hole * 2 + 1;
        }
        else {
            minHeap.a\[hole\] = minHeap.a\[hole * 2\];
            hole = hole * 2;
        }
    }
    minHeap.a\[hole\] = minHeap.a\[minHeap.size\];
    minHeap.size--;
    checkOdd(minHeap);
    return min;
}
void main()
{
    int n, a;
    Heap minHeap;
    minHeap.size = 0;
    cout << "请输入最小堆结点个数（100以内）：\\n";
    cin >> n;
    cout << "请依次输入结点数字：\\n";
    for (int i = 0; i < n; i++) {
        cin >> a;
        incertMinHeap(a, minHeap);
        cout << "已插入第" << i + 1 << "个数\\n";
    }
    cout << "请输入要查看从小到大排列的前几个数：\\n";
    cin >> n;
    for (int i = 0; i < n; i++) {
        a = deleteMin(minHeap);
        cout << "第" << i + 1 << "小的数为：" << a << "\\n";
    }
    system("pause");
}

      2016年3月