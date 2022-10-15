# ✅ Deque
양방향으로 삽입/삭제가 가능한 큐 자료구조
</br>

<img src = "https://user-images.githubusercontent.com/108064146/195986240-ef70967c-b08f-4e84-9b14-82bf0bac72cc.PNG">
</br>

## ✅ main

```c
#include <stdio.h>
#include "Deque.h"

int main(void) {
    Deque deq;
    dequeInit(&deq);
    
    addFirst(&deq, 3);
    addFirst(&deq, 2);
    addFirst(&deq, 1);
    
    addLast(&deq, 4);
    addLast(&deq, 5);
    addLast(&deq, 6);
    
    while(!isEmpty(&deq))
        printf("%d ", removeFirst(&deq));
    
    printf("\n");
    
    addFirst(&deq, 3);
    addFirst(&deq, 2);
    addFirst(&deq, 1);
    
    addLast(&deq, 4);
    addLast(&deq, 5);
    addLast(&deq, 6);
    
    while(!isEmpty(&deq))
        printf("%d ", removeLast(&deq));
    
    printf("\n");
    
    return 0;
}
/*
1 2 3 4 5 6 
6 5 4 3 2 1 
*/
```

## ✅ Deque.h

```c
#ifndef Deque_h
#define Deque_h

#define TRUE 1
#define FALSE 0

typedef int Data;

typedef struct _node {
    Data data;
    struct _node* next;
    struct _node* prev;
} Node;

typedef struct _doublyLinkedDeque {
    Node* head;
    Node* tail;
} DLDeque;

typedef DLDeque Deque;

void dequeInit(Deque* pdeq);

int isEmpty(Deque* pdeq);

void addFirst(Deque* pdeq, Data data);
void addLast(Deque* pdeq, Data data);

Data removeFirst(Deque* pdeq);
Data removeLast(Deque* pdeq);

Data getFirst(Deque* pdeq);
Data getLast(Deque* pdeq);

#endif /* Deque_h */
```

## ✅ Deque.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "Deque.h"

void dequeInit(Deque* pdeq) {
    pdeq->head = NULL;
    pdeq->tail = NULL;
}

int isEmpty(Deque* pdeq) {
    if(pdeq->head == NULL)
        return TRUE;
    else
        return FALSE;
}

void addFirst(Deque* pdeq, Data data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = pdeq->head;
    
    if(isEmpty(pdeq))
        pdeq->tail = newNode;
    else
        pdeq->head->prev = newNode;
    
    newNode->prev = NULL;
    pdeq->head = newNode;
}
void addLast(Deque* pdeq, Data data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->prev = pdeq->tail;
    
    if(isEmpty(pdeq))
        pdeq->head = newNode;
    else
        pdeq->tail->next = newNode;
    
    newNode->next = NULL;
    pdeq->tail = newNode;
}

Data removeFirst(Deque* pdeq) {
    if(isEmpty(pdeq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    Node* rnode = pdeq->head;
    Data rdata = pdeq->head->data;
    
    pdeq->head = pdeq->head->next;
    free(rnode);
    
    if(pdeq->head == NULL)
        pdeq->tail = NULL;
    else
        pdeq->head->prev = NULL;
    
    return rdata;
}
Data removeLast(Deque* pdeq) {
    if(isEmpty(pdeq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    Node* rnode = pdeq->tail;
    Data rdata = pdeq->tail->data;
    
    pdeq->tail = pdeq->tail->prev;
    free(rnode);
    
    if(pdeq->tail == NULL)
        pdeq->head = NULL;
    else
        pdeq->tail->next = NULL;
    
    return rdata;
}

Data getFirst(Deque* pdeq) {
    if(isEmpty(pdeq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    
    return pdeq->head->data;
}
Data getLast(Deque* pdeq) {
    if(isEmpty(pdeq)) {
        printf("Queue Memory Error.");
        exit(-1);
    }
    
    return pdeq->tail->data;
}
```

## References
- 윤성우의 열혈 자료구조
