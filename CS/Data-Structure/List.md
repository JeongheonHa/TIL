# ✅ 배열기반 리스트

## ✅ main

```c
#include <stdio.h>
#include <stdlib.h>
#include "ArrayList.h"
#include "Point.h"

int main(void) {
    List list;
    Point compPos;
    Point* ppos;

    ListInit(&list);

    ppos = (Point*)malloc(sizeof(Point));
    SetPointPos(ppos, 2, 1);
    LInsert(&list, ppos);

    ppos = (Point*)malloc(sizeof(Point));
    SetPointPos(ppos, 2, 2);
    LInsert(&list, ppos);

    ppos = (Point*)malloc(sizeof(Point));
    SetPointPos(ppos, 3, 1);
    LInsert(&list, ppos);

    ppos = (Point*)malloc(sizeof(Point));
    SetPointPos(ppos, 3, 2);
    LInsert(&list, ppos);

    printf("현재 데이터의 수: %d \n", LCount(&list));

    if(LFirst(&list, &ppos)) {
        ShowPointPos(ppos);

        while(LNext(&list, &ppos))
            ShowPointPos(ppos);
    }
    printf("\n");

    compPos.xpos = 2;
    compPos.ypos = 0;

    if(LFirst(&list, &ppos)) {
        ppos=LRemove(&list);
        free(ppos);
    }

    while(LNext(&list, &ppos)) {
        if(PointComp(ppos, &compPos)==1) {
            ppos=LRemove(&list);
            free(ppos);
        }
    }
    printf("현재 데이터의 수: %d \n", LCount(&list));

    if(LFirst(&list, &ppos)) {
        ShowPointPos(ppos);

        while(LNext(&list, &ppos))
            ShowPointPos(ppos);
    }
    printf("\n");

    return 0;
}
```

## ✅ ArrayList header

```c
#ifndef __ARRAY_LIST_H__
#define __ARRAY_LIST_H__

#include "Point.h"

#define TRUE 1
#define FALSE 0

#define LIST_LEN 100
typedef Point* LData;

typedef struct __ArrayList {
    LData arr[LIST_LEN];
    int numOfData;
    int curPosition;
} ArrayList;

typedef ArrayList List;

void ListInit(List* plist);
void LInsert(List* plist, LData data);

int LFirst(List* plist, LData* pdata);
int LNext(List* plist, LData* pdata);

LData LRemove(List* plist);
int LCount(List* plist);

#endif
```
```c
#include <stdio.h>
#include "ArrayList.h"

void ListInit(List* plist) {
    (plist->numOfData) = 0;
    (plist->curPosition) = -1;
}

void LInsert(List* plist, LData data) {
    if(plist->numOfData >= LIST_LEN) {
        puts("저장이 불가능합니다.");
        return;
    }
    plist->arr[plist->numOfData] = data;
    (plist->numOfData)++;
}

int LFirst(List* plist, LData* pdata) {
    if(plist->numOfData == 0)
        return FALSE;
    
    (plist->curPosition) = 0;
    *pdata = plist->arr[0];
    return TRUE;
}

int LNext(List* plist, LData* pdata) {
    if(plist->curPosition >= (plist->numOfData)-1)
        return FALSE;
    (plist->curPosition)++;
    *pdata = plist->arr[plist->curPosition];
    return TRUE;
}

LData LRemove(List* plist) {
    int rpos = plist->curPosition;
    int num = plist->numOfData;
    int i;
    LData rdata = plist->arr[rpos];
    
    for(i=rpos; i<num-1; i++)
        plist->arr[i] = plist->arr[i+1];
    
    (plist->numOfData)--;
    (plist->curPosition)--;
    return rdata;
}

int LCount(List* plist) {
    return plist->numOfData;
}

```

## ✅ Point header

```c
#ifndef __POINT_H__
#define __POINT_H__

typedef struct _point {
    int xpos;
    int ypos;
} Point;

void SetPointPos(Point* ppos, int xpos, int ypos);

void ShowPointPos(Point* ppos);

int PointComp(Point* pos1, Point* pos2);

#endif /* Point_h */
```
```c
#include <stdio.h>
#include "Point.h"

void SetPointPos(Point* ppos, int xpos, int ypos) {
    ppos->xpos = xpos;
    ppos->ypos = ypos;
}

void ShowPointPos(Point* ppos) {
    printf("[%d, %d] \n", ppos->xpos, ppos->ypos);
}

int PointComp(Point* pos1, Point* pos2) {
    if (pos1->xpos == pos2->xpos && pos1->ypos == pos2->ypos)
        return 0;
    else if (pos1->xpos == pos2->xpos)
        return 1;
    else if (pos1->ypos == pos2->ypos)
        return 2;
    else
        return -1;
}
```

## References
- 윤성우의 열혈 자료구조

