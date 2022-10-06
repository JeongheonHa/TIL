# ✅ 배열 기반 스택

## ✅ main

```c
#include <stdio.h>
#include "ArrayBaseStack.h"

int main(void) {
    Stack stack;
    stackInit(&stack);
    
    push(&stack, 1); push(&stack, 2);
    push(&stack, 3); push(&stack, 4);
    push(&stack, 5); push(&stack, 6);
    push(&stack, 7);
    
    while(!isEmpty(&stack)) {
        printf("%d ", pop(&stack));
    }
    
    return 0;
}

/*
7 6 5 4 3 2 1
*/
```

## ✅ ArrayBaseStack.h

```c
#ifndef ArrayBaseStack_h
#define ArrayBaseStack_h

#define TRUE 1
#define FALSE 0
#define STACK_LEN 100

typedef int Data;

typedef struct _arrayStack {
    Data arr[STACK_LEN];
    int topIndex;
} ArrayStack;

typedef ArrayStack Stack;

void stackInit(Stack* pstack);

int isEmpty(Stack* pstack);

void push(Stack* pstack, Data data);

Data pop(Stack* pstack);

Data peek(Stack* pstack);

#endif /* ArrayBaseStack_h */
```
## ✅ ArrayBaseStack.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "ArrayBaseStack.h"

void stackInit(Stack* pstack) {
    pstack->topIndex = -1;
}

int isEmpty(Stack* pstack) {
    if(pstack->topIndex == -1)
        return TRUE;
    else
        return FALSE;
}

void push(Stack* pstack, Data data) {
    pstack->topIndex++;
    pstack->arr[pstack->topIndex] = data;
}

Data pop(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("Stack is empty.");
        exit(-1);
    }
    int pIndex = pstack->topIndex;
    
    pstack->topIndex--;
    return pstack->arr[pIndex];
}

Data peek(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("Stack is empty.");
        exit(-1);
    }
    
    return pstack->arr[pstack->topIndex];
}
```



# ✅ 리스트 기반 스택

## ✅ main

```c
#include <stdio.h>
#include "ListBaseStack.h"

int main(void) {
    Stack stack;
    stackInit(&stack);
    
    push(&stack, 1); push(&stack, 2);
    push(&stack, 3); push(&stack, 4);
    push(&stack, 5); push(&stack, 6);
    
    while(!isEmpty(&stack)) {
        printf("%d ", pop(&stack));
    }
    
    return 0;
}

/*
6 5 4 3 2 1
*/
```

## ✅ ListBaseStack.h

```c
#ifndef ListBaseStack_h
#define ListBaseStack_h

#define TRUE 1
#define FALSE 0

typedef int Data;

typedef struct _node {
    Data data;
    struct _node* next;
} Node;

typedef struct _listStack {
    Node* head;
} ListStack;

typedef ListStack Stack;

void stackInit(Stack* pstack);

int isEmpty(Stack* pstack);

void push(Stack* pstack, Data data);

Data pop(Stack* pstack);

Data peek(Stack* pstack);

#endif /* ListBaseStack_h */
```

## ✅ ListBaseStack.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "ListBaseStack.h"

void stackInit(Stack* pstack) {
    pstack->head = NULL;
}

int isEmpty(Stack* pstack) {
    if(pstack->head == NULL)
        return TRUE;
    else
        return FALSE;
}

void push(Stack* pstack, Data data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = pstack->head;
    pstack->head = newNode;
}

Data pop(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("stack is empty.");
        exit(-1);
    }
    Node* rnode = pstack->head;
    Data rdata = pstack->head->data;
    
    pstack->head = pstack->head->next;
    free(rnode);
    return rdata;
}

Data peek(Stack* pstack) {
    if(isEmpty(pstack)) {
        printf("Stack is empty.");
        exit(-1);
    }
    return pstack->head->data;
}
```

## References
- 윤성우의 열혈 자료구조







