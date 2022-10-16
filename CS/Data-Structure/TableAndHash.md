# ✅ Table(테이블)
키와 값으로 이루어진 테이블 형태의 자료구조
- key와 value가 한쌍을 이룬다.
- key가 존재하지 않는 value는 저장할 수 없으며 모든 key는 중복이 허용되지 않는다.
- 시간 복잡도 : `O(1)`
- 슬롯은 테이블에 데이터를 저장할 수 있는 각각의 공간을 말한다.
- 슬롯의 상태는 `EMPTY`, `DELETED`, `INUSE`로 나뉜다.

## ✅ 해쉬 함수

- 해쉬 함수는 넓은 범위의 키를 좁은 범위의 키로 변경하는 역할을 한다.
- 해쉬가 같은 경우 `충돌`이 발생한다.
- 좋은 해쉬 함수는 데이터가 테이블 전체에 골고루 분포되어있어 충돌을 덜 이르키게하는 함수이다.
- 좋은 해쉬 함수는 키의 일부분을 참조하여 해쉬 값을 만드는 것이 아닌 키 전체를 참조하여 만들어야 한다.
- 해쉬 함수를 디자인할 때는 키의 특성과 저장공간의 크기를 고려하는 것이 우선이다.

### 자릿수 선택 방법 (Digit Selection)
키의 특정 위치에서 중복의 비율이 높거나, 공통으로 들어가는 값의 제외한 나머지를 이용해 해쉬 값을 생성하는 방법

### 자릿수 폴딩 방법 (Digit Folding)
종이를 접듯이 숫자를 겹치게 하여 더한 결과를 해쉬 값으로 결정하는 방법

## ✅ 충돌

### ✅ open addressing method

#### 1. 선형 조사법 (Linear Probing)
충돌이 발생할 경우 한칸씩 옆으로 이동하여 비었는지 확인하는 방법
- 충돌 횟수가 증가함에 따라 `클러스터 현상`이 발생한다는 단점이있다.

#### 2. 이차 조사법 (Quadratic Probing)
충돌이 발생할 경우 1^2, 2^2, ..., n^2 씩 옆으로 이동하여 비었는지 확인하는 방법
- 해쉬 값이 같으면, 충돌 발생시 빈 슬롯을 찾기 위해서 접근하는 위치가 늘 동일하다는 단점이 있다.

#### 3. 이중 해쉬 (Double Hash)
두 개의 해쉬 함수를 이용하는 방법
- 1차 해쉬 함수는 키를 근거로 저장위치를 결정한다.
- 2차 해쉬 함수는 충돌 발생시 몇 칸 뒤를 살필지 결정한다.
</br>

#### Example :   
- 1차 해쉬 함수 : h1(k) = k % 15   
- 2차 해쉬 함수 : h2(k) = 1 + (k % 7)  
    + 2차 해쉬 값이 0이 되는 것을 막기위해 1을 더한다.
    + 가급적 2차 해쉬 값이 1차 해쉬 값을 넘지 않기위해 15보다 작은 소수를 선택한다.
    + 소수로 선택할 경우 클러스터 현상을 낮춘다.


### ✅ closed addressing method

#### 체이닝(Chaining)
슬롯을 생성하여 `연결 리스트`의 모델로 연결해나가는 방식으로 충돌을 해결하는 방법
- 배열을 사용하지 않는 이유는 충돌이 발생하지 않을 경우 메모리 낭비가 심하고, 충돌의 최대 횟수를 결정해야 하는 부담이 있기 때문이다.
- 연결리스트의 단점이기도한 탐색을 위해서는 동일한 해쉬 값으로 묶여있는 연결된 슬롯을 처음부터 모두 조사해야한다는 단점이 있지만 해쉬 함수를 잘 정의하여 충돌 확률이 높지 않다면 연결된 슬롯의 길이는 부담스럽지 않다.

## ✅ main

```c
#include <stdio.h>
#include <stdlib.h>
#include "Person.h"
#include "Table.h"

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
대상 정의

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

## ✅ Slot.h
테이블의 슬롯 정의

```c
#ifndef Slot_h
#define Slot_h

#include "Person.h"

typedef int Key;
typedef Person* Value;

enum SlotStatus {EMPTY, DELETED, INUSE};

// 테이블의 슬롯
typedef struct _slot {
    Key key;
    Value val;
    enum SlotStatus status;
} Slot;

#endif /* Slot_h */
```

## ✅ Table.h
테이블 배열 정의

```c
#ifndef Table_h
#define Table_h

#include "Slot.h"
#include "LinkedList.h"

#define MAX_TBL 100

typedef int HashFunc(Key k);

typedef struct _table {
    Slot tb[MAX_TBL];
    HashFunc* hf;
} Table;

void initTable(Table* pt, HashFunc* hf);

void insertInTable(Table* pt, Key key, Value val);

Value deleteInTable(Table* pt, Key key);

Value searchInTable(Table* pt, Key key);

#endif /* Table_h */
```

## ✅ Table.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "Table.h"

void initTable(Table* pt, HashFunc* hf) {
    for(int i = 0; i<MAX_TBL; i++) {
        pt->tb[i].status = EMPTY;
    }
    pt->hf = hf;
}

void insertInTable(Table* pt, Key key, Value val) {
    int hIdx = pt->hf(key);
    pt->tb[hIdx].key = key;
    pt->tb[hIdx].val = val;
    pt->tb[hIdx].status = INUSE;
}

Value deleteInTable(Table* pt, Key key) {
    int hIdx = pt->hf(key);
    
    if(pt->tb[hIdx].status != INUSE)
        return NULL;
    else {
        pt->tb[hIdx].status = DELETED;
        return pt->tb[hIdx].val;
    }
}

Value searchInTable(Table* pt, Key key) {
    int hIdx = pt->hf(key);
    
    if(pt->tb[hIdx].status != INUSE)
        return NULL;
    else
        return pt->tb[hIdx].val;
}

```

## Reference
- 윤성우의 열혈 C 자료구조
