# ✅ ArrayBaseQueue

배열 기반의 큐 중에서 원형 큐를 구현한다. </br>

<img src = "https://user-images.githubusercontent.com/108064146/194569915-48867734-b977-4c7f-ac6e-99ccc9cee982.jpeg">
</br>

## ✅ main

```c
#include <stdio.h>
#include "CircularQueue.h"

int main(void) {
    Queue q;
    queueInit(&q);
    
    enqueue(&q, 1); enqueue(&q, 2);
    enqueue(&q, 3); enqueue(&q, 4);
    enqueue(&q, 5);
    
    while(!isEmpty(&q)) {
        printf("%d ", dequeue(&q));
    }
    return 0;
}

/*
1 2 3 4 5
*/
```

## ✅ ArrayBaseQueue.h

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

## ✅ ArrayBaseQueue.c

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

# ✅ ListBaseQueue
<img src = "https://user-images.githubusercontent.com/108064146/194570180-f994750a-d5ea-4fcd-94ee-4925f853d3b3.jpeg">
</br>

## ✅ main

```c
#include <stdio.h>
#include "ListBaseQueue.h"

int main(void) {
    Queue q;
    queueInit(&q);
    
    enqueue(&q, 1); enqueue(&q, 2);
    enqueue(&q, 3); enqueue(&q, 4);
    enqueue(&q, 5);
    
    while(!isEmpty(&q))
        printf("%d ",dequeue(&q));
    
    return 0;
}

/*
1 2 3 4 5
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

## ✅ ListBaseQueue.c

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

## References
- 윤성우의 열혈 자료구조
