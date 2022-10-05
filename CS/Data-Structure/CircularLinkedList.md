# ✅ 단방향 원형 연결 리스트 구현

더미가 없는 단방향 원형 리스트를 구현한다.

## ✅ main

```c
#include <stdio.h>
#include "CLinkedList.h"

int main(void) {
    List list;
    int data;
    ListInit(&list);
    
    LInsert(&list, 3);
    LInsert(&list, 4);
    LInsert(&list, 5);
    LInsertFront(&list, 2);
    LInsertFront(&list, 1);
    
    printf("LinkedList 3번 반복 반환 \n");
    
    if(LFirst(&list, &data)) {
        printf("%d ", data);
        
        for(int i = 0; i < LCount(&list)*3-1; i++) {
            if(LNext(&list, &data))
                printf("%d ", data);
        }
    }
    printf("\n\n");
    
    printf("2의 배수 모두 제거 \n");
    
    if(LCount(&list) != 0) {
        LFirst(&list, &data);
        if(data%2 == 0)
            LRemove(&list);
        
        for(int i = 0; i < LCount(&list)-1; i++) {
            LNext(&list, &data);
            if(data%2 == 0)
                LRemove(&list);
        }
    }
    
    if(LFirst(&list, &data)) {
        printf("%d ", data);
        
        for(int i = 0; i < LCount(&list)-1; i++) {
            if(LNext(&list, &data))
                printf("%d ", data);
        }
    }
    printf("\n\n");
    
    return 0;
}

/*
현재 데이터의 수: 5 
11 11 22 22 33 

현재 데이터의 수: 3 
11 11 33 
*/
```

## ✅ CLinkedList.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "CLinkedList.h"

#define TRUE 1
#define FALSE 0

void ListInit(List* plist) {
    plist->tail = NULL;
    plist->cur = NULL;
    plist->before = NULL;
    plist->numOfData = 0;
}

void LInsertFront(List* plist, LData data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    
    if(plist->tail == NULL) {
        plist->tail = newNode;
        newNode->next = newNode;
    } else {
        newNode->next = plist->tail->next;
        plist->tail->next = newNode;
    }
    plist->numOfData++;
}


void LInsert(List* plist, LData data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    
    if(plist->tail == NULL) {
        plist->tail = newNode;
        newNode->next = newNode;
    } else {
        newNode->next = plist->tail->next;
        plist->tail->next = newNode;
        plist->tail = newNode;
    }
    plist->numOfData++;
}

int LFirst(List* plist, LData* pdata) {
    if(plist->tail == NULL)
        return FALSE;
    plist->before = plist->tail;
    plist->cur = plist->tail->next;
    
    *pdata = plist->cur->data;
    return TRUE;
}

int LNext(List* plist, LData* pdata) {
    if(plist->tail == NULL)
        return FALSE;
    plist->before = plist->cur;
    plist->cur = plist->cur->next;
    
    *pdata = plist->cur->data;
    return TRUE;
}

LData LRemove(List* plist) {
    Node* rpos = plist->cur;
    LData rdata = plist->cur->data;
    
    if(rpos == plist->tail) {
        if(plist->tail == plist->tail->next)
            plist->tail = NULL;
        else
            plist->tail = plist->before;
    }
    plist->before->next = plist->cur->next;
    plist->cur = plist->before;
    plist->numOfData--;
    free(rpos);
    return rdata;
}

int LCount(List* plist) {
    return plist->numOfData;
}
```

## ✅ header file

```c
#ifndef CLinkedList_h
#define CLinkedList_h

#include <stdio.h>

typedef int LData;

typedef struct _node {
    LData data;
    struct _node* next;
} Node;

typedef struct _clinkedlist {
    Node* tail;
    Node* cur;
    Node* before;
    int numOfData;
} CLinkedList;

typedef CLinkedList List;

void ListInit(List* plist);

void LInsert(List* plist, LData data);

void LInsertFront(List* plist, LData data);

int LFirst(List* plist, LData* pdata);

int LNext(List* plist, LData* pdata);

LData LRemove(List* plist);

int LCount(List* plist);

#endif /* CLinkedList_h */
```

## References
- 윤성우의 열혈 자료구조




