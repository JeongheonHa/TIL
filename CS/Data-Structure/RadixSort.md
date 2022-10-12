# ✅ 기수 정렬
데이터를 구성하는 기본요소인 기수를 이용해서 정렬하는 알고리즘
- 비교연산을 하지않고 정렬 알고리즘의 시간복잡도인 O(nlog n)을 넘을 수 있는 유일한 알고리즘
- 정렬 대상이 제한적이라는 단점이 있다.
- 데이터의 길이가 같은 데이터들을 대상으로 정렬을 한다.
- 기수 정렬의 시간 복잡도는 삽입과 추출의 빈도수에 의해 결정된다.
- 시간 복잡도 : `O(n)`

<img src = "https://user-images.githubusercontent.com/108064146/195353713-294a4894-b853-4605-ab2d-8a5c80e5ecbe.PNG">
</br>

## ✅ LSD vs MSD

### LSD (Least Significant Digit)
- 덜 중요한 자릿수부터 정렬을 진행하는 방식
- 보통 기수 정렬이라하면 `LSD 기수 정렬`을 말한다.
- 가장 영향력이 큰 자릿수를 마지막에 비교하여 마지막까지 결과를 알 수 없다.

### MSD (Most Significant Digit)
- 가장 중요한 지릿수부터 정렬을 진행하는 방식
- 반드시 끝까지 가지 않아도 중간에 정렬이 완료될 수 있다.
- 대신, 중간에 데이터를 점검하고 일정 부분은 정렬을 진행하고 어떤 부분은 정렬을 진행하면 안되는지 구분해야하기 때문에 구현 난이도가 높다.
- 데이터를 점검하는 과정에서 추가적인 연산과 별도의 메모리가 요구된다.

## ✅ main

```c
#include <stdio.h>
#include "ListBaseQueue.h"

#define BUCKET_NUM 10

void radixSort(int arr[], int num, int maxLen) {
    Queue buckets[BUCKET_NUM];
    int bi;     // bucket의 index
    int pos;    // 자릿수의 위치
    int di;     // 데이터의 index
    int divfac = 1;
    int radix;  // N번째 자리의 수
    
    for(bi=0; bi<BUCKET_NUM; bi++) {
        queueInit(&buckets[bi]);
    }
    
    for(pos=0; pos<maxLen; pos++) {
        for(di=0; di<num; di++) {
            radix = (arr[di]/divfac)%10;
            enqueue(&buckets[radix], arr[di]);
        }
        for(bi=0, di=0; bi<BUCKET_NUM; bi++) {
            while(!isEmpty(&buckets[bi]))
                arr[di++] = dequeue(&buckets[bi]);
        }
        divfac *= 10;
    }
}

int main(void) {
    int arr[7] = {13, 212, 14, 7141, 10987, 6, 15};
    
    int len = sizeof(arr)/sizeof(int);
    int i;
    
    radixSort(arr, len, 5);
    
    for(i=0; i<len; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
6 13 14 15 212 7141 10987 
*/
```

## ✅ ListBaseQueue.h

```c
#ifndef ListBaseQueue_h
#define ListBaseQueue_h

#define TRUE 1
#define FALSE 0

typedef int Data;

typedef struct _node {
    Data data;
    struct _node* next;
} Node;

typedef struct _listQueue {
    Node* front;
    Node* rear;
} ListQueue;

typedef ListQueue Queue;

void queueInit(Queue* pq);

int isEmpty(Queue* pq);

void enqueue(Queue* pq, Data data);

Data dequeue(Queue* pq);

Data peek(Queue* pq);

#endif /* ListBaseQueue_h */
```

## ✅ LisBaseQueue.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "ListBaseQueue.h"

void queueInit(Queue* pq) {
    pq->front = NULL;
    pq->rear = NULL;
}

int isEmpty(Queue* pq) {
    if(pq->front == NULL)
        return TRUE;
    else
        return FALSE;
}

void enqueue(Queue* pq, Data data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->next = NULL;
    newNode->data = data;
    if(isEmpty(pq)) {
        pq->front = newNode;
        pq->rear = newNode;
    } else {
        pq->rear->next = newNode;
        pq->rear = newNode;
    }
}

Data dequeue(Queue* pq) {
    Node* delNode;
    Data retData;
    
    if(isEmpty(pq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    
    delNode = pq->front;
    retData = pq->front->data;
    pq->front = pq->front->next;
    
    free(delNode);
    return retData;
}

Data peek(Queue* pq) {
    if(isEmpty(pq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    
    return pq->front->data;
}
```

## Reference
- 윤성우의 열혈 C 자료구조
