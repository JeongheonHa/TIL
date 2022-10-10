# ✅ Priority Queue (우선순위 큐)

<img src = "https://user-images.githubusercontent.com/108064146/194876097-b33adea4-161d-44c8-a78a-47c8583f4cf1.PNG"> </br>

## ✅ main

```c
#include <stdio.h>
#include "PriorityQueue.h"

int DataPriorityComp(PQData d1, PQData d2) {
    return d1-d2; // 아스키 코드가 작을수록 우선순위가 높다. 양수면 d1이 우선순위가 높다.
}

int main(void) {
    PQueue pq;
    PQueueInit(&pq, DataPriorityComp);
    
    PEnqueue(&pq, 13);
    PEnqueue(&pq, 12);
    PEnqueue(&pq, 8);
    printf("%d \n", PDequeue(&pq));
    
    PEnqueue(&pq, 7);
    PEnqueue(&pq, 4);
    PEnqueue(&pq, 2);
    printf("%d \n", PDequeue(&pq));
    
    printf("\n");
    
    while(!PQIsEmpty(&pq))
        printf("%d \n", PDequeue(&pq));
    
    return 0;
}

/*
13 
12 

8 
7 
4 
*/
```

## ✅ PriorityQueue.h

```c
#ifndef PriorityQueue_h
#define PriorityQueue_h

#include "MaxHeap.h"

typedef Heap PQueue;
typedef HData PQData;

void PQueueInit(PQueue* ppq, PriorityComp comp);

int PQIsEmpty(PQueue* ppq);

void PEnqueue(PQueue* ppq, PQData data);

PQData PDequeue(PQueue* ppq);

#endif /* PriorityQueue_h */
```

## ✅ PriorityQueue.c

```c
#include "PriorityQueue.h"

void PQueueInit(PQueue* ppq, PriorityComp comp) {
    return heapInit(ppq, comp);
}

int PQIsEmpty(PQueue* ppq) {
    return isEmpty(ppq);
}

void PEnqueue(PQueue* ppq, PQData data) {
    insert(ppq, data);
}

PQData PDequeue(PQueue* ppq) {
    return delete(ppq);
}
```

## ✅ MaxHeap.h

```c
#ifndef MaxHeap_h
#define MaxHeap_h

#define TRUE 1
#define FALSE 0

#define HEAP_LEN 100

typedef int HData;
typedef int PriorityComp(HData d1, HData d2);

typedef struct _heap {
    int numOfData;
    HData heapArr[HEAP_LEN];
    PriorityComp* comp;
} Heap;

void heapInit(Heap* ph, PriorityComp comp);

int isEmpty(Heap* ph);

void insert(Heap* ph, HData data);

HData delete(Heap* ph);

#endif /* MaxHeap_h */
```

## ✅ MaxHeap.c

```c
#include "MaxHeap.h"

void heapInit(Heap* ph, PriorityComp comp) {
    ph->numOfData = 0;
    ph->comp = comp;
}

int isEmpty(Heap* ph) {
    if(ph->numOfData == 0)
        return TRUE;
    else
        return FALSE;
}

int getParentIdx(int idx) {
    return idx/2;
}

int getLChildIdx(int idx) {
    return 2*idx;
}

int getRChildIdx(int idx) {
    return 2*idx+1;
}

int getHiPriChildIdx(Heap* ph, int idx) {
    if(getLChildIdx(idx) > ph->numOfData)   // leaf node가 둘다 없는 경우
        return 0;
    else if(getLChildIdx(idx) == ph->numOfData) // leaf node가 left child node인 경우
        return getLChildIdx(idx);
    else {                                  // leaf node가 둘다 있는 경우
        if(ph->comp(ph->heapArr[getLChildIdx(idx)], ph->heapArr[getRChildIdx(idx)]) > 0) // 두 node의 우선순위 비교
            return getLChildIdx(idx);
        else
            return getRChildIdx(idx);
    }
}


void insert(Heap* ph, HData data) {
    int idx = ph->numOfData+1;
    
    while(idx != 1) {   // idx가 root node가 될 때까지
        if(ph->comp(data, ph->heapArr[getParentIdx(idx)]) > 0) {  // data의 우선순위가 부모 노드보다 우선순위가 높을 경우
            ph->heapArr[idx] = ph->heapArr[getParentIdx(idx)];
            idx = getParentIdx(idx);
        }
        else
            break;
    }
    
    ph->heapArr[idx] = data;
    ph->numOfData++;
}

HData delete(Heap* ph) {
    HData retData = ph->heapArr[1];
    HData lastElem = ph->heapArr[ph->numOfData];
    
    int parentIdx = 1;
    int childIdx;
    
    while(1) {
        childIdx = getHiPriChildIdx(ph, parentIdx);
        if(ph->comp(lastElem, ph->heapArr[childIdx]) >= 0)
            break;
        ph->heapArr[parentIdx] = ph->heapArr[childIdx];
        parentIdx = childIdx;
    }
    
    ph->heapArr[parentIdx] = lastElem;
    ph->numOfData--;
    return retData;
}
```

## References
- 윤성우의 열혈 C 자료구조
