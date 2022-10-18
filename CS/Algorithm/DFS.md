# ✅ 깊이 우선 탐색(DFS, Depth First Search)
그래프의 모든 정점을 탐색하기 위한 알고리즘으로 한길을 깊이 파고드는 방식이다.

<img src = "https://user-images.githubusercontent.com/108064146/196456474-82fdc7fc-4b0b-4e4f-8dce-2998b35410cd.PNG">
</br>


## ✅ DFS의 과정

- 연결된 하나의 정점에 방문한다. (방문할 순서는 오름차순으로 정했다.)
- 방문할 정점이 없으면 다시 돌아간다.
- 처음 시작 정점의 위치에서 끝이난다.

## ✅ 탐색에 필요한 추가 자료구조

- 스택 : 경로 정보의 추적을 위해 필요하다.
    + 방문할 정점이 없으면 다시 돌아가기 위해 사용
    + 처음 시작 정점에 도착했는지 알기위해 사용
- 배열 : 방문 정보의 기록을 위해 필요하다.
    + 방문한 정점인지 파악하기 위해 사용
    + 기본값이 0이고 해당 정점을 방문하면 1로 바꾼다.

## ✅ main

```c
#include <stdio.h>
#include "ALGraphDFS.h"

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
A B C D E F G 
C B A D E F G 
E D A B C F G 
G E D A B C F 
*/
```

## ✅ ALGraphDFS.h

```c
#ifndef ALGraphDFS_h
#define ALGraphDFS_h

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

void showGraphVertex(ALGraph* pg, int startV);  // 정점의 정보 출력 (DFS 기반)

#endif /* ALGraphDFS_h */

```

## ✅ ALGraphDFS.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ALGraphDFS.h"
#include "LinkedList.h"
#include "ArrayBaseStack.h"

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

void showGraphVertex(ALGraph* pg, int startV) {
    Stack stack;   // 스택 생성
    int visitV = startV;
    int nextV;
    
    stackInit(&stack);
    visitVertex(pg, visitV);    // 시작 정점 방문
    push(&stack, visitV);
    
    while(LFirst(&(pg->adjList[visitV]), &nextV) == TRUE) {   // 첫번째 정점이 존재한다면
        int visitFlag = FALSE;
        
        if(visitVertex(pg, nextV) == TRUE) {    // 해당 정점이 방문한 적이 없다면
            push(&stack, visitV);   // 스택에 해당 정점을 추가
            visitV = nextV;         // while문을 다시 돌리기 위해 visitV로 저장
            visitFlag = TRUE;       // 정점 방문 성공
        } else {    // 해당 정점이 방문한 적이 있다면
            while(LNext(&(pg->adjList[visitV]), &nextV) == TRUE) {    // 다음 연결된 정점을 찾는다
                if(visitVertex(pg, nextV) == TRUE) {    // 해당 정점이 방문한 적이 없다면
                    push(&stack, visitV);   // 스택에 해당 정점 추가
                    visitV = nextV;         // while문을 다시 돌리기 위해 visitV로 저장
                    visitFlag = TRUE;       // 정점 방문 성공
                    break;
                }
            }
        }
        
        if(visitFlag == FALSE) {    // 해당 정점과 연결된 모든 정점에 방문 실패
            if(isEmpty(&stack) == TRUE) // 스택이 비어있다면 시작위치로 돌아온 것이다.
                break;
            else
                visitV = pop(&stack);   // 스택이 아직 차있으면 가장 위에 것 빼기
        }
    }
    
    memset(pg->visitInfo, 0, sizeof(int)*pg->numV); // 방문 기록 배열 다시 0으로 초기화
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

## ✅ ArrayBaseStack.h

```c
#ifndef ArrayBaseStack_h
#define ArrayBaseStack_h

#define TRUE 1
#define FALSE 0
#define STACK_LEN 100

typedef int Data;

typedef struct _arrayStack {
    Data arr[STACK_LEN];
    int topIndex;
} ArrayStack;

typedef ArrayStack Stack;

void stackInit(Stack* pstack);

int isEmpty(Stack* pstack);

void push(Stack* pstack, Data data);

Data pop(Stack* pstack);

Data peek(Stack* pstack);

#endif /* ArrayBaseStack_h */
```

## ✅ ArrayBaseStack.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "ArrayBaseStack.h"

void stackInit(Stack* pstack) {
    pstack->topIndex = -1;
}

int isEmpty(Stack* pstack) {
    if(pstack->topIndex == -1)
        return TRUE;
    else
        return FALSE;
}

void push(Stack* pstack, Data data) {
    pstack->topIndex++;
    pstack->arr[pstack->topIndex] = data;
}

Data pop(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("Stack is empty.");
        exit(-1);
    }
    int pIndex = pstack->topIndex;
    
    pstack->topIndex--;
    return pstack->arr[pIndex];
}

Data peek(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("Stack is empty.");
        exit(-1);
    }
    
    return pstack->arr[pstack->topIndex];
}

```

## Reference
- 윤성우의 열혈 C 자료구조
