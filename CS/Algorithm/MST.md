# ✅ 최소 비용 신장 트리(MST)
신장 트리의 모든 간선의 가중치 합이 최소인 그래프

<img src = "https://user-images.githubusercontent.com/108064146/196733495-a71db77b-f176-4b09-9a4e-13d879243bdd.PNG">
</br>
<img src = "https://user-images.githubusercontent.com/108064146/196733594-e7f3496b-643f-4e46-a567-09178918ac26.PNG">
</br>

## ✅ 용어 정리
- 경로 : 두개의 정점을 잇는 간선을 순서대로 나열한것
- 단순 경로 : 동일한 간선을 중복하여 포함하지 않는 경로
- 사이클 : 단순 경로이면서 시작과 끝이 같은 경로
- 신장 트리 : 사이클을 형성하지 않는 그래프
    + 그래프의 모든 정점이 간선에 의해서 하나로 연결
    + 그래프 내에서 사이클을 형성하지 않는다.

## ✅ MST 구성 알고리즘

### 크루스칼 알고리즘
가중치를 기준으로 간선을 정렬한 후 MST가 될 때까지 간선을 하나씩 선택 or 삭제해 나가는 방식 (오름차순, 내림차순)

### 프림 알고리즘
하나의 정점을 시작으로 MST가 될 때까지 트리를 확장해 나가는 방식

## ✅ 가중치 오름차순 정렬을 이용한 크루스칼 알고리즘 과정
- 가중치를 기준으로 간선을 `오름차순`으로 정렬
- 낮은 가중치의 간선부터 하나씩 그래프에 추가
- 사이클을 형성하는 간선은 추가하지 않는다.
- 간선의 수가 정점의 수보다 하나 적을 때 MST 완성

## ✅ 가중치 내림차순 정렬을 이용한 크루스칼 알고리즘 과정
- 가중치를 기준으로 간선을 `내림차순` 정렬
- 높은 가중치의 간선부터 하나씩 그래프에서 제거
- 두 정점을 연결하는 다른 경로가 없을 경우 해당 간은은 제거하지 않는다.
- 간선의 수가 정점의 수보다 하나 적을 때 MST 완성

## ✅ main

```c
#include <stdio.h>
#include "ALGraphKruskal.h"

int main(void) {
    ALGraph graph;
    graphInit(&graph, 6);
    
    addEdge(&graph, A, B, 9);
    addEdge(&graph, B, C, 2);
    addEdge(&graph, A, C, 12);
    addEdge(&graph, A, D, 8);
    addEdge(&graph, D, C, 6);
    addEdge(&graph, A, F, 11);
    addEdge(&graph, F, D, 4);
    addEdge(&graph, D, E, 3);
    addEdge(&graph, E, C, 7);
    addEdge(&graph, F, E, 13);
    
    conKruskalMST(&graph);
    showGraphEdgeInfo(&graph);
    showGraphEdgeWeightInfo(&graph);
    
    graphDestroy(&graph);
    
    return 0;
}

/*
A와 연결된 정점 : D 
B와 연결된 정점 : C 
C와 연결된 정점 : B D 
D와 연결된 정점 : A C E F 
E와 연결된 정점 : D 
F와 연결된 정점 : D 
(A-D), w:73 
(D-C), w:71 
(F-D), w:69 
(D-E), w:68 
(B-C), w:67 
*/
```

## ✅ ALGraphKruskal.h

```c
#ifndef ALGraphKruskal_h
#define ALGraphKruskal_h

#include "LinkedList.h"
#include "PriorityQueue.h"
#include "ALEdge.h"

enum {A, B, C, D, E, F, G, H, I, J};

typedef struct _ual {
    int numV;   // 정점의 수 (vertex)
    int numE;   // 간선의 수 (edge)
    List* adjList;  // 간선의 정보
    int* visitInfo; // 방문 정보 기록 배열
    PQueue pqueue;  // 간선의 가중치 정보 저장
} ALGraph;

void graphInit(ALGraph* pg, int nv);    // 그래프 초기화

void graphDestroy(ALGraph* pg);         // 그래프 리소스 해제

void addEdge(ALGraph* pg, int fromV, int toV, int weight);  // 간선의 추가

void showGraphEdgeInfo(ALGraph* pg);    // 간선의 정보 출력

void showGraphVertex(ALGraph* pg, int startV);  // 정점의 정보 출력 (BFS 기반)

void conKruskalMST(ALGraph* pg);    // 최소 비용 신장 트리 구성

void showGraphEdgeWeightInfo(ALGraph* pg);  // 가중치 정보 출력

#endif /* ALGraphKruskal_h */

```

## ✅ ALGraphKruskal.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "LinkedList.h"
#include "ArrayBaseStack.h"
#include "PriorityQueue.h"
#include "ALGraphKruskal.h"

int whoIsPrecede(int d1, int d2);
int weightComp(Edge d1, Edge d2);

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
    
    // 간선의 가중치 정보를 우선순위 큐에 저장
    PQueueInit(&pg->pqueue, weightComp);
}

void graphDestroy(ALGraph* pg) {// 그래프 리소스 해제
    if(pg->adjList != NULL)
        free(pg->adjList);
    if(pg->visitInfo != NULL)
        free(pg->visitInfo);
}

void addEdge(ALGraph* pg, int fromV, int toV, int weight) {  // 간선의 추가
    // 간선의 가중치 정보를 담는다.
    Edge edge = {fromV, toV, weight};
    
    LInsert(&pg->adjList[fromV], toV);
    LInsert(&pg->adjList[toV], fromV);
    pg->numE++;
    
    // 간선의 가중치 정보를 우선순위 큐에 저장
    PEnqueue(&pg->pqueue, edge);
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

int weightComp(Edge d1, Edge d2) {
    return d1.weight - d2.weight;
}

int visitVertex(ALGraph* pg, int visitV) {
    if(pg->visitInfo[visitV] == 0) {    // 배열이 비어있다면
        pg->visitInfo[visitV] = 1;      // 해당 정점을 방문했다는 의미로 1저장
        // printf("%c ", visitV+65);       // 상수를 아스키 코드로 출력
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

void removeWayEdge(ALGraph* pg, int fromV, int toV) {
    int edge;
    
    if(LFirst(&pg->adjList[fromV], &edge)) {
        if(edge == toV) {
            LRemove(&pg->adjList[fromV]);
            return ;
        }
        while(LNext(&pg->adjList[fromV], &edge))
            if(edge == toV) {
                LRemove(&pg->adjList[fromV]);
                return ;
            }
    }
}

void removeEdge(ALGraph* pg, int fromV, int toV) {
    removeWayEdge(pg, fromV, toV);
    removeWayEdge(pg, toV, fromV);
    pg->numE--;
}

void recoverEdge(ALGraph* pg, int fromV, int toV, int weight) {
    LInsert(&pg->adjList[fromV], toV);
    LInsert(&pg->adjList[toV], fromV);
    pg->numE++;
}

int isConnVertex(ALGraph* pg, int v1, int v2) {
    Stack stack;   // 스택 생성
    int visitV = v1;
    int nextV;
    
    stackInit(&stack);
    visitVertex(pg, visitV);    // 시작 정점 방문
    push(&stack, visitV);
    
    while(LFirst(&(pg->adjList[visitV]), &nextV) == TRUE) {   // 첫번째 정점이 존재한다면
        int visitFlag = FALSE;
        
        // 정점을 돌아다니는 동중에 목표를 찾으면 TRUE 반환
        if(nextV == v2) {
            memset(pg->visitInfo, 0, sizeof(int)*pg->numV);
            return TRUE;
        }
        
        if(visitVertex(pg, nextV) == TRUE) {    // 해당 정점이 방문한 적이 없다면
            push(&stack, visitV);   // 스택에 해당 정점을 추가
            visitV = nextV;         // while문을 다시 돌리기 위해 visitV로 저장
            visitFlag = TRUE;       // 정점 방문 성공
        } else {    // 해당 정점이 방문한 적이 있다면
            while(LNext(&(pg->adjList[visitV]), &nextV) == TRUE) {    // 다음 연결된 정점을 찾는다
                // 정점을 돌아다니는 동중에 목표를 찾으면 TRUE 반환
                if(nextV == v2) {
                    memset(pg->visitInfo, 0, sizeof(int)*pg->numV);
                    return TRUE;
                }
                
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
    return FALSE;   // 목표를 찾지 못했다면 FALSE 반환
}

void conKruskalMST(ALGraph* pg) {    // 최소 비용 신장 트리 구성
    Edge recvEdge[20];
    Edge edge;
    int eIdx = 0;
    
    while(pg->numE+1 > pg->numV) {
        edge = PDequeue(&pg->pqueue);
        removeEdge(pg, edge.v1, edge.v2);
        
        if(!isConnVertex(pg, edge.v1, edge.v2)) {
            recoverEdge(pg, edge.v1, edge.v2, edge.weight);
            recvEdge[eIdx++] = edge;
        }
    }
    
    for(int i=0; i<eIdx; i++)
        PEnqueue(&pg->pqueue, recvEdge[i]);
}

void showGraphEdgeWeightInfo(ALGraph* pg) {  // 가중치 정보 출력
    PQueue copyPQ = pg->pqueue;
    Edge edge;
    
    while(!PQIsEmpty(&copyPQ)) {
        edge = PDequeue(&copyPQ);
        printf("(%c-%c), w:%d \n", edge.v1+65, edge.v2+65, edge.weight+65);
    }
}

```

## ✅ ALEdge.h

```c
#ifndef ALEdge_h
#define ALEdge_h

typedef struct _edge {
    int v1;     // 간선이 연결하는 첫 번째 정점
    int v2;     // 간선이 연결하는 두 번째 정점
    int weight; // 간선의 가중치
} Edge;

#endif /* ALEdge_h */
```

## Reference
- 윤성우의 열혈 C 자료구조
