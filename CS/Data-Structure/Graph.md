# ✅ 그래프(Graph)
`정점`과 `간선`으로 이루어진 자료구조

- `정점`(Vertex) : 연결 대상이 되는 개체 또는 위치
- `간선`(edge) : 정점 사이의 연결

<img src = "https://user-images.githubusercontent.com/108064146/196456377-f7cf582a-3ce0-4dd8-a85b-feac6b1038b9.PNG">
</br>

## ✅ 그래프의 종류

### 방향 그래프(다이그래프)
- 간선에 방향 정보가 포함된 그래프

### 완전 그래프
- 각각의 정점에서 다른 모든 정점을 연결한 그래프

### 가충치 그래프
- 간선에 가중치 정보를 표현한 그래프

### 구분 그래프
- 부분 집합과 유사하게 원 그래프의 일부 정점 및 간선으로 이루어진 그래프

## ✅ 그래프의 집합 표현
- 정접 집합은 `V(G)`로 표시
- 간선 집합은 `E(G)`로 표시

### 무방향 그래프의 집합 표현
- V(G) = {A, B, C, D}, E(G) = {(A, B), (A, C), (A,D), (B, C), (C, D)}

### 방향 그래프의 집합 표현
- V(G) = {A, B, C, D}, E(G) = {<A, B>, <A, C>, <D, A>}

## ✅ 그래프 구현 방법

### 인접 행렬 기반 그래프
2차원 배열을 이용한 그래프

### 인접 리스트 기반 그래프
연결 리스트를 이용한 그래프

## ✅ main

```c
#include <stdio.h>
#include "ALGraph.h"

int main(void) {
    ALGraph graph;
    graphInit(&graph, 5);
    
    addEdge(&graph, A, B);
    addEdge(&graph, A, D);
    addEdge(&graph, B, C);
    addEdge(&graph, C, D);
    addEdge(&graph, D, E);
    addEdge(&graph, E, A);
    
    showGraphEdgeInfo(&graph);
    graphDestroy(&graph);
    
    return 0;
}

/*
A와 연결된 정점 : B D E 
B와 연결된 정점 : A C 
C와 연결된 정점 : B D 
D와 연결된 정점 : A C E 
E와 연결된 정점 : A D 
*/
```

## ✅ Graph.h
인접 리스트 기반의 그래프를 구현한다.

```c
#ifndef ALGraph_h
#define ALGraph_h

#include "LinkedList.h"

enum {A, B, C, D, E, F, G, H, I, J};

typedef struct _ual {
    int numV;   // 정점의 수 (vertex)
    int numE;   // 간선의 수 (edge)
    List* adjList;  // 간선의 정보
} ALGraph;

void graphInit(ALGraph* pg, int nv);    // 그래프 초기화

void graphDestroy(ALGraph* pg);         // 그래프 리소스 해제

void addEdge(ALGraph* pg, int fromV, int toV);  // 간선의 추가

void showGraphEdgeInfo(ALGraph* pg);    // 간선의 정보 출력

#endif /* ALGraph_h */
```

## ✅ Graph.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "ALGraph.h"
#include "LinkedList.h"

int whoIsPrecede(int d1, int d2);

void graphInit(ALGraph* pg, int nv) {    // 그래프 초기화
    pg->adjList = (List*)malloc(sizeof(List)*nv);
    pg->numV = nv;
    pg->numE = 0;
    
    for(int i=0; i<nv; i++) {
        ListInit(&pg->adjList[i]);
        SetSortRule(&pg->adjList[i], whoIsPrecede);
    }
}

void graphDestroy(ALGraph* pg) {// 그래프 리소스 해제
    if(pg->adjList != NULL)
        free(pg->adjList);
}

void addEdge(ALGraph* pg, int fromV, int toV) {  // 간선의 추가
    LInsert(&pg->adjList[fromV], toV);
    LInsert(&pg->adjList[toV], fromV);
    pg->numE++;
}

void showGraphEdgeInfo(ALGraph* pg) {    // 간선의 정보 출력
    int vx;
    
    for(int i=0; i<pg->numV; i++) {
        printf("%c와 연결된 정점 : ", i+65);
        
        if(LFirst(&pg->adjList[i], &vx)) {
            printf("%c ", vx+65);
            
            while(LNext(&(pg->adjList[i]), &vx))
                printf("%c ", vx+65);
        }
        printf("\n");
    }
}
int whoIsPrecede(int d1, int d2) {
    if(d1 < d2)
        return 0;
    else
        return 1;
}

```

## ✅ LinkedList.h

```c
#ifndef LinkedList_h
#define LinkedList_h

#define TRUE 1
#define FALSE 0

typedef int LData;

typedef struct _node {
    LData data;
    struct _node* next;
} Node;

typedef struct _linkedlist {
    Node* head;
    Node* cur;
    Node* before;
    int len;
    int (*comp)(LData d1, LData d2);
} LinkedList;

typedef LinkedList List;

void ListInit(List* plist);

void LInsert(List* plist, LData data);

int LFirst(List* plist, LData* pdata);

int LNext(List* plist, LData* pdata);

LData LRemove(List* plist);

int LCount(List* plist);

void SetSortRule(List* plist, int (*comp)(LData d1, LData d2));

#endif /* LinkedList_h */
```

## ✅ LinkedList.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "LinkedList.h"

void ListInit(List* plist) {
    plist->head = (Node*)malloc(sizeof(Node));
    plist->head->next = NULL;
    plist->len = 0;
    plist->comp = NULL;
}

void FInsert(List* plist, LData data) {
    Node* new_node = (Node*)malloc(sizeof(Node));
    new_node->data = data;
    new_node->next = plist->head->next;
    plist->head->next = new_node;
    plist->len++;
}

void SInsert(List* plist, LData data) {
    Node* new_node = (Node*)malloc(sizeof(Node));
    Node* pred = plist->head;
    new_node->data = data;
    
    while(pred->next != NULL && plist->comp(data, pred->next->data) != 0) {
        pred = pred->next;
    }
    new_node->next = pred->next;
    pred->next = new_node;
    
    plist->len++;
}

void LInsert(List* plist, LData data) {
    if(plist->comp == NULL) {
        FInsert(plist, data);
    } else {
        SInsert(plist, data);
    }
}

int LFirst(List* plist, LData* pdata) {
    if(plist->head->next == NULL)
        return FALSE;
    plist->before = plist->head;
    plist->cur = plist->head->next;
    *pdata = plist->cur->data;
    
    return TRUE;
}

int LNext(List* plist, LData* pdata) {
    if(plist->cur->next == NULL)
        return FALSE;
    plist->before = plist->cur;
    plist->cur = plist->cur->next;
    *pdata = plist->cur->data;
    
    return TRUE;
}

LData LRemove(List* plist) {
    Node* rpos = plist->cur;
    LData rdata = plist->cur->data;
    
    plist->before->next = plist->cur->next;
    plist->cur = plist->before;
    
    free(rpos);
    plist->len--;
    return rdata;
}

int LCount(List* plist) {
    return plist->len;
}

void SetSortRule(List* plist, int (*comp)(LData d1, LData d2)) {
    plist->comp = comp;
}

```

## Reference
- 윤성우의 열혈 C 자료구조
