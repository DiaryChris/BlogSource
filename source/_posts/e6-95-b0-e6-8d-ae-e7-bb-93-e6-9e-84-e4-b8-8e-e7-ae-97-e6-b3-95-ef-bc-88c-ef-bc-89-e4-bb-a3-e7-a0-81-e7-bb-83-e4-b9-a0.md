---
title: 数据结构与算法（C++）代码练习（图）
url: 137.html
id: 137
categories:
  - 算法
date: 2017-11-27 23:30:21
tags:
---

_所有代码均由Microsoft Visual Studio 2015编译通过_      

#### 图的遍历DFS和BFS

 

#include<iostream>
#include<iomanip>
#define N 16
using namespace std;
class listNode
{
public:
    int data;
    listNode* next;
};
class QueueNode
{
public:
    int a;
    QueueNode* next;
};
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
void inputLesson(listNode* head\[\])
{
    head\[0\] = NULL;
    head\[1\] = new listNode;
    head\[1\]->data = 3;
    head\[1\]->next = NULL;
    head\[2\] = new listNode;
    head\[2\]->data = 3;
    head\[2\]->next = new listNode;
    head\[2\]->next->data = 13;
    head\[2\]->next->next = NULL;
    head\[3\] = new listNode;
    head\[3\]->data = 7;
    head\[3\]->next = NULL;
    head\[4\] = new listNode;
    head\[4\]->data = 5;
    head\[4\]->next = NULL;
    head\[5\] = new listNode;
    head\[5\]->data = 6;
    head\[5\]->next = NULL;
    head\[6\] = new listNode;
    head\[6\]->data = 15;
    head\[6\]->next = NULL;
    head\[7\] = new listNode;
    head\[7\]->data = 10;
    head\[7\]->next = new listNode;
    head\[7\]->next->data = 11;
    head\[7\]->next->next = new listNode;
    head\[7\]->next->next->data = 12;
    head\[7\]->next->next->next = NULL;
    head\[8\] = new listNode;
    head\[8\]->data = 9;
    head\[8\]->next = NULL;
    head\[9\] = new listNode;
    head\[9\]->data = 10;
    head\[9\]->next = new listNode;
    head\[9\]->next->data = 11;
    head\[9\]->next->next = NULL;
    head\[10\] = new listNode;
    head\[10\]->data = 14;
    head\[10\]->next = NULL;
    head\[11\] = NULL;
    head\[12\] = NULL;
    head\[13\] = NULL;
    head\[14\] = NULL;
    head\[15\] = NULL;
}
void DFS(listNode* list\[\], int index, int visited\[\])
{
    listNode* p;
    if (visited\[index\]) {
        return;
    }
    cout << index;
    visited\[index\] = 1;
    p = list\[index\];
    while (p) {
        DFS(list, p->data, visited);
        p = p->next;
    }
}
void BFS(listNode* list\[\], int index, int visited\[\])
{
    int v;
    listNode* p;
    QueueNode* front,* rear;
    front = rear = new QueueNode;
    rear->next = NULL;
    enqueue(index, rear);
    visited\[index\] = 1;
    while (front != rear) {
        v = dequeue(front);
        cout << v;
        p = list\[v\];
        while (p) {
            if (!(visited\[p->data\])) {
                enqueue(p->data, rear);
                visited\[p->data\] = 1;
            }
            p = p->next;
        }
    }
}
void resetVisited(int visited\[\])
{
    for (int i = 0; i < N; i++) {
        visited\[i\] = 0;
    }
}
void main()
{
    listNode* list\[N\];
    int n;
    int visited\[N\];
    inputLesson(list);
    cout << "请输入深度优先搜索起始点序号：\\n";
    cin >> n;
    resetVisited(visited);
    DFS(list, n, visited);
    cout << '\\n';
    cout << "请输入广度优先搜索起始点序号：\\n";
    cin >> n;
    resetVisited(visited);
    BFS(list, n, visited);
    cout << '\\n';
    system("pause");
}

   

#### 最小生成树Prim算法

 

#include<iostream>
#include<iomanip>
#define N 8
#define MAX 100000
using namespace std;
class AdjList
{
public:
    int data;
    int dist;
    AdjList* next;
    AdjList() {
        next = NULL;
    }
};
class Vertex
{
public:
    int id;
    AdjList* adj;
    int dist;
    int visited;
    int path;
    Vertex() {
        adj = NULL;
        path = 0;
        visited = 0;
        dist = MAX;
    }
};
class Edge
{
public:
    int v;
    int w;
    void printEdge()
    {
        cout << v << '-' << w << "\\n";
    }
};
class Heap
{
public:
    int size;
    Vertex a\[N\];
};
void checkOdd(Heap& minHeap)
{
    if (!(minHeap.size % 2)) {
        minHeap.a\[minHeap.size + 1\].dist = MAX;
    }
}
void incertMinHeap(Vertex v, Heap& minHeap)
{
    int hole = ++minHeap.size;
    while (hole != 1 && v.dist < minHeap.a\[hole / 2\].dist) {
        minHeap.a\[hole\] = minHeap.a\[hole / 2\];
        hole = hole / 2;
    }
    minHeap.a\[hole\] = v;
    checkOdd(minHeap);
}
Vertex deleteMin(Heap& minHeap)
{
    Vertex min = minHeap.a\[1\];
    int hole = 1;
    while (hole <= minHeap.size / 2 && (minHeap.a\[minHeap.size\].dist > minHeap.a\[hole * 2 + 1\].dist || minHeap.a\[minHeap.size\].dist > minHeap.a\[hole * 2\].dist)) {
        if (minHeap.a\[hole * 2 + 1\].dist < minHeap.a\[hole * 2\].dist) {
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
void input(Vertex &v, int data, int dist)
{
    AdjList*& p = v.adj;
    if (data == 0 && dist == 0) {
        p = NULL;
        return;
    }
    if (!p) {
        p = new AdjList;
        p->data = data;
        p->dist = dist;
    }
    else {
        AdjList* temp = p;
        while (p->next) {
            p = p->next;
        }
        p->next = new AdjList;
        p->next->data = data;
        p->next->dist = dist;
        p = temp;
    }
}
void inputSample(Vertex v\[\])
{
    for (int i = 0; i < N; i++)
    {
        v\[i\].id = i;
    }
    input(v\[1\], 2, 2);
    input(v\[1\], 3, 4);
    input(v\[1\], 4, 1);
    input(v\[2\], 1, 2);
    input(v\[2\], 4, 3);
    input(v\[2\], 5, 10);
    input(v\[3\], 1, 4);
    input(v\[3\], 4, 2);
    input(v\[3\], 6, 5);
    input(v\[4\], 1, 1);
    input(v\[4\], 2, 3);
    input(v\[4\], 3, 2);
    input(v\[4\], 5, 7);
    input(v\[4\], 6, 8);
    input(v\[4\], 7, 4);
    input(v\[5\], 2, 10);
    input(v\[5\], 4, 7);
    input(v\[5\], 7, 6);
    input(v\[6\], 3, 5);
    input(v\[6\], 4, 8);
    input(v\[6\], 7, 1);
    input(v\[7\], 4, 4);
    input(v\[7\], 5, 6);
    input(v\[7\], 6, 1);
}
void Prim(Vertex &s, Vertex g\[\], Edge MST\[\])
{
    int i = 0;
    Vertex v;
    AdjList* p;
    Heap minHeap;
    minHeap.size = 0;
    s.dist = 0;
    incertMinHeap(s, minHeap);
    while (minHeap.size) {
        v = deleteMin(minHeap);
        while (g\[v.id\].visited) {
            if (!minHeap.size) {
                return;
            }
            v = deleteMin(minHeap);
        }
        g\[v.id\].visited = 1;
        g\[v.id\].dist = 0;
        if (v.path) {
            i++;
            MST\[i\].v = v.path;
            MST\[i\].w = v.id;
        }
        p = v.adj;
        while (p) {
            if (g\[p->data\].visited == 0 && p->dist < g\[p->data\].dist) {
                g\[p->data\].dist = p->dist ;
                g\[p->data\].path = v.id;
                incertMinHeap(g\[p->data\], minHeap);
            }
            p = p->next;
        }
    }
}
void check(Vertex v\[\])
{
    AdjList* p;
    for (int i = 1; i < N; i++)
    {
        cout << i << '\\n';
        p = v\[i\].adj;
        while (p) {
            cout << p->data << ' ' << p->dist << ' ' << p->next << '\\n';
            p = p->next;
        }
    }
    system("pause");
}
void main()
{

    Vertex v\[N\];
    Edge MST\[N - 1\];
    int n;
    inputSample(v);
    cout << "请输入源点序号：\\n";
    cin >> n;
    Prim(v\[n\], v, MST);
    cout << "最小生成树为：\\n";
    for (int i = 1; i < N - 1; i++)
    {
        MST\[i\].printEdge();
    }
    system("pause");
}

   

#### 最小生成树Kruskal算法

 

#include<iostream>
#include<iomanip>
#define N 8
#define MAX 100000
using namespace std;
class AdjList
{
public:
    int data;
    int dist;
    AdjList* next;
    AdjList() {
        next = NULL;
    }
};
class Vertex
{
public:
    int id;
    AdjList* adj;
    int root;
    int visited;

    Vertex() {
        adj = NULL;
        root = -1;
        visited = 0;
    }
};
class Edge
{
public:
    int v;
    int w;
    int l;
    Edge(int v = 0, int w = 0, int l = 0) :v(v), w(w), l(l) {}
    void printEdge()
    {
        cout << v << '-' << w << "\\n";
    }
};
class Heap
{
public:
    int size;
    Edge a\[N*(N - 1)\];
    Heap() {
        size = 0;
    }
};
void checkOdd(Heap& minHeap)
{
    if (!(minHeap.size % 2)) {
        minHeap.a\[minHeap.size + 1\].l = MAX;
    }
}
void incertMinHeap(Edge e, Heap& minHeap)
{
    int hole = ++minHeap.size;
    while (hole != 1 && e.l < minHeap.a\[hole / 2\].l) {
        minHeap.a\[hole\] = minHeap.a\[hole / 2\];
        hole = hole / 2;
    }
    minHeap.a\[hole\] = e;
    checkOdd(minHeap);
}
Edge deleteMin(Heap& minHeap)
{
    Edge min = minHeap.a\[1\];
    int hole = 1;
    while (hole <= minHeap.size / 2 && (minHeap.a\[minHeap.size\].l > minHeap.a\[hole * 2 + 1\].l || minHeap.a\[minHeap.size\].l > minHeap.a\[hole * 2\].l)) {
        if (minHeap.a\[hole * 2 + 1\].l < minHeap.a\[hole * 2\].l) {
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
int find(Vertex v, Vertex g\[\])
{
    if (v.root == -1) {
        return v.id;
    }
    return find(g\[v.root\], g);
}
void unionSet(int r1, int r2, Vertex g\[\])
{
        g\[r2\].root = g\[r1\].id;
}
void input(Vertex &v, int data, int dist)
{
    AdjList*& p = v.adj;
    if (data == 0 && dist == 0) {
        p = NULL;
        return;
    }
    if (!p) {
        p = new AdjList;
        p->data = data;
        p->dist = dist;
    }
    else {
        AdjList* temp = p;
        while (p->next) {
            p = p->next;
        }
        p->next = new AdjList;
        p->next->data = data;
        p->next->dist = dist;
        p = temp;
    }
}
void inputSample(Vertex v\[\])
{
    for (int i = 0; i < N; i++)
    {
        v\[i\].id = i;
    }
    input(v\[1\], 2, 2);
    input(v\[1\], 3, 4);
    input(v\[1\], 4, 1);
    input(v\[2\], 1, 2);
    input(v\[2\], 4, 3);
    input(v\[2\], 5, 10);
    input(v\[3\], 1, 4);
    input(v\[3\], 4, 2);
    input(v\[3\], 6, 5);
    input(v\[4\], 1, 1);
    input(v\[4\], 2, 3);
    input(v\[4\], 3, 2);
    input(v\[4\], 5, 7);
    input(v\[4\], 6, 8);
    input(v\[4\], 7, 4);
    input(v\[5\], 2, 10);
    input(v\[5\], 4, 7);
    input(v\[5\], 7, 6);
    input(v\[6\], 3, 5);
    input(v\[6\], 4, 8);
    input(v\[6\], 7, 1);
    input(v\[7\], 4, 4);
    input(v\[7\], 5, 6);
    input(v\[7\], 6, 1);
}
void Kruskal(Vertex g\[\], Edge MST\[\])
{
    Heap minHeap;
    Edge e;
    int i = 0;
    for (int i = 1; i < N; i++) {
        AdjList* p = g\[i\].adj;
        while (p) {
            if (!g\[p->data\].visited) {
                Edge e(i, p->data, p->dist);
                incertMinHeap(e, minHeap);
            }
            p = p->next;
        }
        g\[i\].visited = 1;
    }
    while (minHeap.size) {
        e = deleteMin(minHeap);
        int r1 = find(g\[e.v\], g);
        int r2 = find(g\[e.w\], g);
        if (r1 != r2) {
            MST\[++i\] = e;
            unionSet(r1, r2, g);
        }
    }

}
void main()
{
    Vertex v\[N\];
    Edge MST\[N - 1\];
    inputSample(v);
    Kruskal(v, MST);
    cout << "最小生成树为：\\n";
    for (int i = 1; i < N - 1; i++)
    {
        MST\[i\].printEdge();
    }
    system("pause");
}

   

#### 基于最小堆的有权图单源最短路径Dijkstra算法

 

#include<iostream>
#include<iomanip>
#define N 16
#define MAX 100000
using namespace std;
class AdjList
{
public:
    int data;
    int dist;
    AdjList* next;
    AdjList() {
        next = NULL;
    }
};
class Vertex
{
public:
    int id;
    AdjList* adj;
    int dist;
    int visited;
    int path;
    Vertex() {
        adj = NULL;
        path = 0;
        visited = 0;
        dist = MAX;
    }
};
class Heap
{
public:
    int size;
    Vertex a\[N\];
};
void checkOdd(Heap& minHeap)
{
    if (!(minHeap.size % 2)) {
        minHeap.a\[minHeap.size + 1\].dist = MAX;
    }
}
void incertMinHeap(Vertex v, Heap& minHeap)
{
    int hole = ++minHeap.size;
    while (hole != 1 && v.dist < minHeap.a\[hole / 2\].dist) {
        minHeap.a\[hole\] = minHeap.a\[hole / 2\];
        hole = hole / 2;
    }
    minHeap.a\[hole\] = v;
    checkOdd(minHeap);
}
Vertex deleteMin(Heap& minHeap)
{
    Vertex min = minHeap.a\[1\];
    int hole = 1;
    while (hole <= minHeap.size / 2 && (minHeap.a\[minHeap.size\].dist > minHeap.a\[hole * 2 + 1\].dist || minHeap.a\[minHeap.size\].dist > minHeap.a\[hole * 2\].dist)) {
        if (minHeap.a\[hole * 2 + 1\].dist < minHeap.a\[hole * 2\].dist) {
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
void input(Vertex &v, int data, int dist)
{
    AdjList*& p = v.adj;
    if (data == 0 && dist == 0){
        p = NULL;
        return;
    }
    if (!p) {
        p = new AdjList;
        p->data = data;
        p->dist = dist;
    }
    else {
        AdjList* temp = p;
        while (p->next) {
            p = p->next;
        }
        p->next = new AdjList;
        p->next->data = data;
        p->next->dist = dist;
        p = temp;
    }
}
void inputSample(Vertex v\[\])
{
    for (int i = 0; i < N; i++)
    {
        v\[i\].id = i;
    }
    input(v\[1\], 2, 2);
    input(v\[1\], 4, 1);
    input(v\[2\], 4, 3);
    input(v\[2\], 5, 10);
    input(v\[3\], 1, 4);
    input(v\[3\], 6, 5);
    input(v\[4\], 3, 2);
    input(v\[4\], 5, 2);
    input(v\[4\], 6, 8);
    input(v\[4\], 7, 4);
    input(v\[5\], 7, 6);
    input(v\[7\], 6, 1);
}
void Dijkstra(Vertex &s, Vertex g\[\])
{
    Vertex v;
    AdjList* p;
    Heap minHeap;
    minHeap.size = 0;
    s.dist = 0;
    incertMinHeap(s, minHeap);
    while (minHeap.size) {
        v = deleteMin(minHeap);
        while (g\[v.id\].visited) {
            if (!minHeap.size) {
                return;
            }
            v = deleteMin(minHeap);
        }
        g\[v.id\].visited = 1;
        p = v.adj;
        while (p) {
            if (g\[p->data\].visited == 0 && p->dist + v.dist < g\[p->data\].dist) {
                g\[p->data\].dist = p->dist + v.dist;
                g\[p->data\].path = v.id;
                incertMinHeap(g\[p->data\], minHeap);
            }
            p = p->next;
        }
    }
}
void printPath(Vertex v, Vertex g\[\])
{
    if (v.path) {
        printPath(g\[v.path\], g);
        cout << "->";
    }
    cout << v.id;
}
void check(Vertex v\[\])
{
    AdjList* p;
    for (int i = 1; i < N; i++)
    {
        cout << i << '\\n';
        p = v\[i\].adj;
        while (p) {
            cout << p->data << ' ' << p->dist << ' ' << p->next << '\\n';
            p = p->next;
        }
    }
    system("pause");
}
void main()
{

    Vertex v\[N\];
    int n;
    inputSample(v);
    cout << "请输入源点序号：\\n";
    cin >> n;
    Dijkstra(v\[n\], v);
    cout << "请输入终点序号：\\n";
    cin >> n;
    cout << "最短路径为：\\n";
    printPath(v\[n\], v);
    cout << "路径权重：" << v\[n\].dist;
    system("pause");
}

   

#### 邻接表存储有向无环图以队列法拓扑排序

 

#include<iostream>
#include<iomanip>
#define N 16
using namespace std;
class listNode
{
public:
    int data;
    listNode* next;
};
class QueueNode
{
public:
    int a;
    QueueNode* next;
};
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
void inputLesson(listNode* head\[\])
{
    head\[0\] = NULL;
    head\[1\] = new listNode;
    head\[1\]->data = 3;
    head\[1\]->next = NULL;
    head\[2\] = new listNode;
    head\[2\]->data = 3;
    head\[2\]->next = new listNode;
    head\[2\]->next->data = 13;
    head\[2\]->next->next = NULL;
    head\[3\] = new listNode;
    head\[3\]->data = 7;
    head\[3\]->next = NULL;
    head\[4\] = new listNode;
    head\[4\]->data = 5;
    head\[4\]->next = NULL;
    head\[5\] = new listNode;
    head\[5\]->data = 6;
    head\[5\]->next = NULL;
    head\[6\] = new listNode;
    head\[6\]->data = 15;
    head\[6\]->next = NULL;
    head\[7\] = new listNode;
    head\[7\]->data = 10;
    head\[7\]->next = new listNode;
    head\[7\]->next->data = 11;
    head\[7\]->next->next = new listNode;
    head\[7\]->next->next->data = 12;
    head\[7\]->next->next->next = NULL;
    head\[8\] = new listNode;
    head\[8\]->data = 9;
    head\[8\]->next = NULL;
    head\[9\] = new listNode;
    head\[9\]->data = 10;
    head\[9\]->next = new listNode;
    head\[9\]->next->data = 11;
    head\[9\]->next->next = NULL;
    head\[10\] = new listNode;
    head\[10\]->data = 14;
    head\[10\]->next = NULL;
    head\[11\] = NULL;
    head\[12\] = NULL;
    head\[13\] = NULL;
    head\[14\] = NULL;
    head\[15\] = NULL;
}
void scanAll(listNode* list\[\], int inDegree\[\])
{
    for (int i = 1; i < N; i++)
    {
        listNode* p;
        p = list\[i\];
        while (p)
        {
            inDegree\[p->data\]++;
            p = p->next;
        }
    }
}
void topSort(listNode* list\[\])
{
    int v;
    int inDegree\[N\];
    listNode* p;
    QueueNode* front, *rear;
    front = rear = new QueueNode;
    for (int i = 0; i < N; i++) {
        inDegree\[i\] = 0;
    }
    scanAll(list, inDegree);
    for (int i = 1; i < N; i++) {
        if (!inDegree\[i\]) {
            enqueue(i, rear);
        }
    }
    while (front != rear) {
        v = dequeue(front);
        cout << v << '\\n';
        p = list\[v\];
        while (p) {
            if (!(--inDegree\[p->data\])) {
                enqueue(p->data, rear);
            }
            p = p->next;
        }
    }

}
void main()
{
    listNode* list\[N\];
    inputLesson(list);
    topSort(list);
    system("pause");
}

        2016年3月