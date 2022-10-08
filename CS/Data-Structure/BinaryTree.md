# ✅ 이진 트리

## ✅ main

```c
#include <stdio.h>
#include "BinaryTree.h"

void showIntData(int data);

int main(void) {
    BTreeNode* bt1 = makeBTreeNode();
    BTreeNode* bt2 = makeBTreeNode();
    BTreeNode* bt3 = makeBTreeNode();
    BTreeNode* bt4 = makeBTreeNode();
    BTreeNode* bt5 = makeBTreeNode();
    BTreeNode* bt6 = makeBTreeNode();
    
    setData(bt1, 1);
    setData(bt2, 2);
    setData(bt3, 3);
    setData(bt4, 4);
    setData(bt5, 5);
    setData(bt6, 6);
    
    makeLeftSubTree(bt1, bt2);
    makeRightSubTree(bt1, bt3);
    makeLeftSubTree(bt2, bt4);
    makeRightSubTree(bt2, bt5);
    makeLeftSubTree(bt4, bt6);
    
    printf("%d \n", getData(getLeftSubTree(bt1)));
    printf("%d \n", getData(getLeftSubTree(getLeftSubTree(bt1))));
    
    preorderTraverse(bt1, showIntData);
    printf("\n\n");
    inorderTraverse(bt1, showIntData);
    printf("\n\n");
    postorderTraverse(bt1, showIntData);
    printf("\n\n");
    
    deleteTree(bt2);
    
    return 0;
}

void showIntData(int data) {
    printf("%d ", data);
}

/*
2 
4 
1 2 4 6 5 3 

6 4 2 5 1 3 

6 4 5 2 3 1 

delete tree data : 6 
delete tree data : 4 
delete tree data : 5 
delete tree data : 2 
*/
```

## ✅ BinaryTree.h

```c
#ifndef BinaryTree_h
#define BinaryTree_h

typedef int BTData;

typedef struct _bTreeNode {
    BTData data;
    struct _bTreeNode* left;
    struct _bTreeNode* right;
} BTreeNode;

BTreeNode* makeBTreeNode(void); // 이진 트리 노드 생성
BTData getData(BTreeNode* bt);  // 노드의 data 반환
void setData(BTreeNode* bt, BTData data);   // 노드의 data 저장

BTreeNode* getLeftSubTree(BTreeNode* bt);   // 왼쪽 자식 노드 반환
BTreeNode* getRightSubTree(BTreeNode* bt);  // 오른쪽 자식 노드 반환

void makeLeftSubTree(BTreeNode* main, BTreeNode* sub);  // 왼쪽 자식 노드 지정(삭제)
void makeRightSubTree(BTreeNode* main, BTreeNode* sub); // 오른쪽 자식 노드 지정(삭제)

void deleteTree(BTreeNode* bt); // 해당 노드의 모든 서브 노드 제거

typedef void (*VisitFuncPtr)(BTData data);

void preorderTraverse(BTreeNode* bt, VisitFuncPtr action);
void inorderTraverse(BTreeNode* bt, VisitFuncPtr action);
void postorderTraverse(BTreeNode* bt, VisitFuncPtr action);

#endif /* BinaryTree_h */
```

## ✅ BinaryTree.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "BinaryTree.h"

BTreeNode* makeBTreeNode(void) {    
    BTreeNode* node = (BTreeNode*)malloc(sizeof(BTreeNode));
    node->left = NULL;
    node->right = NULL;
    return node;
}

BTData getData(BTreeNode* bt) {     
    return bt->data;
}
void setData(BTreeNode* bt, BTData data) {  
    bt->data = data;
}

BTreeNode* getLeftSubTree(BTreeNode* bt) {  
    return bt->left;
}
BTreeNode* getRightSubTree(BTreeNode* bt) {
    return bt->right;
}

void makeLeftSubTree(BTreeNode* main, BTreeNode* sub) {     
    if(main->left != NULL)
        free(main->left);
    
    main->left = sub;
}
void makeRightSubTree(BTreeNode* main, BTreeNode* sub) {    
    if(main->right != NULL)
        free(main->right);
    
    main->right = sub;
}

void deleteTree(BTreeNode* bt) {
    if(bt == NULL)
        return;
    
    deleteTree(bt->left);
    deleteTree(bt->right);
    
    printf("delete tree data : %d \n", bt->data);
    free(bt);
}

void preorderTraverse(BTreeNode* bt, VisitFuncPtr action) {
    if(bt == NULL)
        return;
    
    action(bt->data);
    preorderTraverse(bt->left, action);
    preorderTraverse(bt->right, action);
    
}
void inorderTraverse(BTreeNode* bt, VisitFuncPtr action) {
    if(bt == NULL)
        return;
    
    inorderTraverse(bt->left, action);
    action(bt->data);
    inorderTraverse(bt->right, action);
}
void postorderTraverse(BTreeNode* bt, VisitFuncPtr action) {
    if(bt == NULL)
        return;
    
    postorderTraverse(bt->left, action);
    postorderTraverse(bt->right, action);
    action(bt->data);
}
```

## References
- 윤성우의 열혈 C 자료구조
