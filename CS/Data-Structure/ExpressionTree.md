# ✅ 수식 트리

## ✅ main

```c
#include <stdio.h>
#include "ExpressionTree.h"

int main(void) {
    char exp[] = "12+7*";
    BTreeNode* eTree = makeExpTree(exp);
    
    printf("전위 표기법의 수식 : ");
    showPrefixTypeExp(eTree);
    printf("\n");
    
    printf("중위 표기법의 수식 : ");
    showInfixTypeExp(eTree);
    printf("\n");
    
    printf("후위 표기법의 수식 : ");
    showPostfixTypeExp(eTree);
    printf("\n");
    
    printf("연산의 결과: %d \n", evalExpTree(eTree));
    
    return 0;
}

/*
전위 표기법의 수식 : * + 1 2 7 
중위 표기법의 수식 : ( ( 1 + 2 )* 7 )
후위 표기법의 수식 : 1 2 + 7 * 
연산의 결과: 21 
*/
```

## ✅ ExpressionTree.h

```c
#ifndef ExpressionTree_h
#define ExpressionTree_h

#include "BinaryTree.h"

BTreeNode* makeExpTree(char exp[]);     // 후위 표기법의 수식을 매개변수로 받으면 이진 트리로 만든다.

int evalExpTree(BTreeNode* bt);         // 후위 표기법으로 만들어진 수식 트리를 계산

void showPrefixTypeExp(BTreeNode* bt);  // 수식 트리를 전위 표기법의 수식으로 표현
void showInfixTypeExp(BTreeNode* bt);   // 수식 트리를 중위 표기법의 수식으로 표현
void showPostfixTypeExp(BTreeNode* bt); // 수식 트리를 후위 표기법의 수식으로 표현

#endif /* ExpressionTree_h */

```

## ✅ ExpressionTree.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include "BinaryTree.h"
#include "ListBaseStack.h"

BTreeNode* makeExpTree(char exp[]) {
    Stack stack;
    BTreeNode* pnode;
    
    int expLen = (int)strlen(exp);
    int i;
    
    stackInit(&stack);
    
    for(i = 0; i < expLen; i++) {
        pnode = makeBTreeNode();
        
        if(isdigit(exp[i])) {
            setData(pnode, exp[i]-'0');
        } else {
            makeRightSubTree(pnode, pop(&stack));
            makeLeftSubTree(pnode, pop(&stack));
            setData(pnode, exp[i]);
        }
        push(&stack, pnode);
    }
    return pop(&stack);
}

int evalExpTree(BTreeNode* bt) {
    if(getLeftSubTree(bt) == NULL && getRightSubTree(bt) == NULL)
        return getData(bt);
    
    int op1 = evalExpTree(getLeftSubTree(bt));
    int op2 = evalExpTree(getRightSubTree(bt));
    
    switch(getData(bt)) {
        case '+':
            return op1+op2;
            break;
        case '-':
            return op1-op2;
            break;
        case '*':
            return op1*op2;
            break;
        case '/':
            return op1/op2;
            break;
    }
    return 0;
}

void showNodeData(int data) {
    if(0<=data && data<=9)
        printf("%d ", data);
    else
        printf("%c ", data);
}

void showPrefixTypeExp(BTreeNode* bt) {
    preorderTraverse(bt, showNodeData);
}
void showInfixTypeExp(BTreeNode* bt) {
    if(bt == NULL)
        return ;
    if(bt->left != NULL || bt->right != NULL)   // 괄호 넣어 보기
        printf("( ");
    
    showInfixTypeExp(bt->left);
    showNodeData(bt->data);
    showInfixTypeExp(bt->right);
    
    if(bt->left != NULL || bt->right != NULL)   // 괄호 넣어 보기
        printf(")");
    
}
void showPostfixTypeExp(BTreeNode* bt) {
    postorderTraverse(bt, showNodeData);
}

```

## References
- 윤성우의 열혈 C 자료구조



