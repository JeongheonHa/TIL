# ✅ 체이닝 (Chaining)

## ✅ main

```c
#include <stdio.h>
#include <stdlib.h>
#include "Person.h"
#include "Table2.h"

int hashFunc(int key) {
    return key % 100;
}

int main(void) {
    Table table;
    Person* p1;
    Person* p2;
    Person* p3;
    
    initTable(&table, hashFunc);
    
    p1 = makePersonData(20120003, "Lee", "Seoul");
    insertInTable(&table, getId(p1), p1);
    
    p2 = makePersonData(20130012, "KIM", "Jeju");
    insertInTable(&table, getId(p2), p2);
    
    p3 = makePersonData(20170049, "HAN", "Kangwon");
    insertInTable(&table, getId(p3), p3);
    
    if(searchInTable(&table, getId(p1)) != NULL)
        showPersonInfo(p1);
    if(searchInTable(&table, getId(p2)) != NULL)
        showPersonInfo(p2);
    if(searchInTable(&table, getId(p3)) != NULL)
        showPersonInfo(p3);
    
    if(deleteInTable(&table, getId(p1)) != NULL)
        free(p1);
    if(deleteInTable(&table, getId(p2)) != NULL)
        free(p2);
    if(deleteInTable(&table, getId(p3)) != NULL)
        free(p3);
    
    return 0;
}

```

## ✅ Person.h

```c
#ifndef Person_h
#define Person_h

#define STR_LEN 50

// 테이블에 저장할 대상
typedef struct _person {
    int id;
    char name[STR_LEN];
    char address[STR_LEN];
} Person;

int getId(Person* p);

void showPersonInfo(Person* p);

Person* makePersonData(int id, char* name, char* address);

#endif /* Person_h */
```

## ✅ Person.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "Person.h"

int getId(Person* p) {
    return p->id;
}

void showPersonInfo(Person* p) {
    printf("주민등록번호 : %d \n", p->id);
    printf("이름 : %s \n", p->name);
    printf("주소 : %s \n", p->address);
}

Person* makePersonData(int id, char* name, char* address) {
    Person* newP = (Person*)malloc(sizeof(Person));
    newP->id = id;
    strcpy(newP->name, name);
    strcpy(newP->address, address);
    return newP;
}
```

## ✅ Slot2.h

```c
#ifndef Slot2_h
#define Slot2_h

#include "Person.h"

typedef int Key;
typedef Person* Value;

// 테이블의 슬롯
typedef struct _slot {
    Key key;
    Value val;
} Slot;

#endif /* Slot2_h */

```

## ✅ Table2.h

```c
#ifndef Table2_h
#define Table2_h

#include "Slot2.h"
#include "LinkedList.h"

#define MAX_TBL 100

typedef int HashFunc(Key k);

typedef struct _table {
    List tb[MAX_TBL];
    HashFunc* hf;
} Table;

void initTable(Table* pt, HashFunc* hf);

void insertInTable(Table* pt, Key key, Value val);

Value deleteInTable(Table* pt, Key key);

Value searchInTable(Table* pt, Key key);

#endif /* Table2_h */

```

## ✅ Table2.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "Table2.h"

void initTable(Table* pt, HashFunc* hf) {
    for(int i=0; i<MAX_TBL; i++) {
        ListInit(&pt->tb[i]);
    }
    pt->hf = hf;
}

void insertInTable(Table* pt, Key key, Value val) {
    int hIdx = pt->hf(key);
    Slot entry = {.key = key, .val = val};
    
    // key가 중복일 경우
    if(searchInTable(pt, key) != NULL) {
        printf("Key is exist.");
        return;
    } else {
        LInsert(&pt->tb[hIdx], entry);
    }
}

Value deleteInTable(Table* pt, Key key) {
    int hIdx = pt->hf(key);
    Slot cSlot;
    
    if(LFirst(&pt->tb[hIdx], &cSlot)) {
        if(cSlot.key == key) {
            LRemove(&pt->tb[hIdx]);
            return cSlot.val;
        } else {
            while(LNext(&pt->tb[hIdx], &cSlot)) {
                LRemove(&pt->tb[hIdx]);
                return cSlot.val;
            }
        }
    }
    return NULL;
}

Value searchInTable(Table* pt, Key key) {
    int hIdx = pt->hf(key);
    Slot cSlot;
    
    if(LFirst(&pt->tb[hIdx], &cSlot)) {
        if(cSlot.key == key)
            return cSlot.val;
        else {
            while(LNext(&pt->tb[hIdx], &cSlot)) {
                if(cSlot.key == key)
                    return cSlot.val;
            }
        }
    }
    return NULL;
}

```

## Reference
- 윤성우의 열혈 C 자료구조
