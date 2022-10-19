# ✅ 너비 우선 탐색(BFS, Breadth First Search)
그래프의 모든 정점을 탐색하기위한 알고리즘으로 자신과 연결된 모든 정점을 먼저 탐색하는 방식이다.

<img src = "https://user-images.githubusercontent.com/108064146/196705392-264d9f0d-4f95-4b78-b2d3-6474283e8703.PNG">
</br>

## ✅ DFS의 과정

- 자신과 연결된 모든 정점을 차례대로 방문한다.
- 방문 기록이 있는 정점은 방문하지 않는다.
- 모든 정점을 방문하면 DFS가 끝난다.

## ✅ 탐색에 필요한 추가 자료구조

- 큐 : 방문 차례를 기록하기위해 필요하다.
    + 방문할 정점의 순서를 기록하기 위한것이다.
- 배열 : 방문 정보를 기록하기위해 필요하다.
    + 방문한 정점인지 파악하기 위해 사용
    + 기본값이 0이고 해당 정점을 방문하면 1로 바꾼다.

## ✅ main

```c
#include <stdio.h>
#include "ALGraphBFS.h"

int main(void) {
    ALGraph graph;
    graphInit(&graph, 7);
    
    addEdge(&graph, A, B);
    addEdge(&graph, A, D);
    addEdge(&graph, B, C);
    addEdge(&graph, D, C);
    addEdge(&graph, D, E);
    addEdge(&graph, E, F);
    addEdge(&graph, E, G);

    showGraphEdgeInfo(&graph);
    
    showGraphVertex(&graph, A); printf("\n");
    showGraphVertex(&graph, C); printf("\n");
    showGraphVertex(&graph, E); printf("\n");
    showGraphVertex(&graph, G); printf("\n");
    
    graphDestroy(&graph);
    
    return 0;
}
/*
A와 연결된 정점 : B D 
B와 연결된 정점 : A C 
C와 연결된 정점 : B D 
D와 연결된 정점 : A C E 
E와 연결된 정점 : D F G 
F와 연결된 정점 : E 
G와 연결된 정점 : E 
A B D C E F G 
C B D A E F G 
E D F G A C B 
G E D F A C B 
*/
```

## ✅ ALGraphBFS.h

```c
#ifndef ALGraphBFS_h
#define ALGraphBFS_h

#include "LinkedList.h"

enum {A, B, C, D, E, F, G, H, I, J};

typedef struct _ual {
    int numV;   // 정점의 수 (vertex)
    int numE;   // 간선의 수 (edge)
    List* adjList;  // 간선의 정보
    int* visitInfo; // 방문 정보 기록 배열
} ALGraph;

void graphInit(ALGraph* pg, int nv);    // 그래프 초기화

void graphDestroy(ALGraph* pg);         // 그래프 리소스 해제

void addEdge(ALGraph* pg, int fromV, int toV);  // 간선의 추가

void showGraphEdgeInfo(ALGraph* pg);    // 간선의 정보 출력

void showGraphVertex(ALGraph* pg, int startV);  // 정점의 정보 출력 (BFS 기반)

#endif /* ALGraphBFS_h */
```

## ✅ ALGraphBFS.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "LinkedList.h"
#include "CircularQueue.h"
#include "ALGraphBFS.h"

int whoIsPrecede(int d1, int d2);

void graphInit(ALGraph* pg, int nv) {    // 그래프 초기화
    pg->adjList = (List*)malloc(sizeof(List)*nv);
    pg->numV = nv;
    pg->numE = 0;
    
    for(int i=0; i<nv; i++) {
        ListInit(&(pg->adjList[i]));
        SetSortRule(&(pg->adjList[i]), whoIsPrecede);
    }
    
    pg->visitInfo = (int*)malloc(sizeof(int)*pg->numV);
    memset(pg->visitInfo, 0, sizeof(int)*pg->numV);
}

void graphDestroy(ALGraph* pg) {// 그래프 리소스 해제
    if(pg->adjList != NULL)
        free(pg->adjList);
    if(pg->visitInfo != NULL)
        free(pg->visitInfo);
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
        
        if(LFirst(&(pg->adjList[i]), &vx)) {
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

int visitVertex(ALGraph* pg, int visitV) {
    if(pg->visitInfo[visitV] == 0) {    // 배열이 비어있다면
        pg->visitInfo[visitV] = 1;      // 해당 정점을 방문했다는 의미로 1저장
        printf("%c ", visitV+65);       // 상수를 아스키 코드로 출력
        return TRUE;
    }
    return FALSE;
}

void showGraphVertex(ALGraph* pg, int startV) {  // 정점의 정보 출력 (BFS 기반)
    Queue queue;
    int visitV = startV;
    int nextV;
    
    queueInit(&queue);
    visitVertex(pg, visitV);
    
    while(LFirst(&pg->adjList[visitV], &nextV) == TRUE) {
        if(visitVertex(pg, nextV) == TRUE) {
            enqueue(&queue, nextV);
        }
        
        while(LNext(&pg->adjList[visitV], &nextV) == TRUE) {
            if(visitVertex(pg, nextV) == TRUE)
                enqueue(&queue, nextV);
        }
        if(isEmpty(&queue))
            break;
        else
            visitV = dequeue(&queue);
    }
    
    memset(pg->visitInfo, 0, sizeof(int)*pg->numV);
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

## ✅ CircularQueue.h

```c
#ifndef CircularQueue_h
#define CircularQueue_h

#define TRUE 1
#define FALSE 0

#define QUE_LEN 100
typedef int Data;

typedef struct _cQueue {
    Data arr[QUE_LEN];
    int front;
    int rear;
} CQueue;

typedef CQueue Queue;

void queueInit(Queue* pq);

int isEmpty(Queue* pq);

void enqueue(Queue* pq, Data data);

Data dequeue(Queue* pq);

Data peek(Queue* pq);

#endif /* CircularQueue_h */
```

## ✅ CurcularQueue.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "CircularQueue.h"

void queueInit(Queue* pq) {
    pq->front = 0;
    pq->rear = 0;
}

int isEmpty(Queue* pq) {
    if(pq->front == pq->rear)
        return TRUE;
    else
        return FALSE;
}

int nextPosIdx(int pos) {
    if(pos == QUE_LEN-1)    // 배열의 첫번째는 빈 공간으로 만든다.
        return 0;
    else
        return pos+1;
}
void enqueue(Queue* pq, Data data) {
    if(nextPosIdx(pq->rear) == pq->front) { // 데이터가 가득차면
        printf("Queue Memory Error.");
        exit(-1);
    }
    pq->rear = nextPosIdx(pq->rear);
    pq->arr[pq->rear] = data;
}

Data dequeue(Queue* pq) {
    if(isEmpty(pq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    pq->front = nextPosIdx(pq->front);
    return pq->arr[pq->front];
}

Data peek(Queue* pq) {
    if(isEmpty(pq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    return pq->arr[nextPosIdx(pq->front)];
}

```

## Reference
- 윤성우의 열혈 C 자료구조
