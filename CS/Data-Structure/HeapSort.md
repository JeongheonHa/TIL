# ✅ 힙 정렬

- 힙 구조의 데이터의 저장 시간 복잡도 : `O(log n)`
- 힙 구조의 데이터의 삭제 시간 복잡도 : `O(log n)`
- 정렬 과정의 시간 복잡도 : `O(nlog n)` (정렬 대상 : n, 데이터 삽입/삭제 : log n)

## ✅ main

```c
#include <stdio.h>
#include "MaxHeap.h"

int priorityComp(int d1, int d2) {
    return d2-d1;
}

void heapSort(int arr[], int n, PriorityComp pc) {
    Heap heap;
    int i;
    heapInit(&heap, pc);
    
    for(i=0; i<n; i++)
        insert(&heap, arr[i]);
    
    for(i=0; i<n; i++)
        arr[i] = delete(&heap);
}

int main(void) {
    int arr[4] = {3, 4, 2, 1};
    int i;
    
    heapSort(arr, sizeof(arr)/sizeof(int), priorityComp);
    
    for(i=0; i<4; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
1 2 3 4
*/
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
        if(ph->comp(ph->heapArr[getLChildIdx(idx)], ph->heapArr[getRChildIdx(idx)]) < 0) // 두 node의 우선순위 비교
            return getRChildIdx(idx);
        else
            return getLChildIdx(idx);
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

    while(childIdx = getHiPriChildIdx(ph, parentIdx)) {
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
