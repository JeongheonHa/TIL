# ✅ 균형 잡힌 이진 탐색 트리 (AVL Tree)
노드가 추가/삭제될 때 트리의 균형상태를 파악해서 스스로 그 구조를 변경하여 균형을 잡는 트리
- 균형 인수(balance factor) = 왼쪽 서브 트리의 높이 - 오른쪽 서브 트리의 높이
- 리밸런싱 : 균형을 잡기 위한 트리 구조의 재조정
- 이진 탐색 트리는 저장 순서에 따라 탐색의 성능에 큰 차이를 보이며 균형이 맞지 않을수록 `O(n)`에 가까운 시간복잡도를 갖는다는 단점을 가진다.

<img src = "https://user-images.githubusercontent.com/108064146/195874504-309c18da-2d29-4461-a0de-164f2b22292d.png">
<img src = "https://user-images.githubusercontent.com/108064146/195874598-b8f2c21e-b757-4ff4-9c53-387ad50927a3.png">
<img src = "https://user-images.githubusercontent.com/108064146/195874693-e05b72c2-6b08-47f9-989a-9661d66784e0.png">

## ✅ main

```c
#include <stdio.h>
#include "BinaryTree.h"
#include "BinarySearchTree.h"

int main(void)
{
    BTreeNode* avlRoot;
    BTreeNode* clNode;        // current left node
    BTreeNode* crNode;        // current right node

    BTreeNode* clNode2;
    BTreeNode* crNode2;

    BTreeNode* clNode3;
    BTreeNode* crNode3;

    makeAndInitBST(&avlRoot);

    insertBST(&avlRoot, 1);
    insertBST(&avlRoot, 2);
    insertBST(&avlRoot, 3);
    insertBST(&avlRoot, 4);
    insertBST(&avlRoot, 5);
    insertBST(&avlRoot, 6);
    insertBST(&avlRoot, 7);
    insertBST(&avlRoot, 8);
    insertBST(&avlRoot, 9);

    printf("루트 노드 : %d \n", getData(avlRoot));

    clNode = getLeftSubTree(avlRoot);
    crNode = getRightSubTree(avlRoot);
    printf("%d, %d \n", getData(clNode), getData(crNode));

    clNode2 = getLeftSubTree(clNode);
    crNode2 = getRightSubTree(clNode);
    printf("%d, %d \n", getData(clNode2), getData(crNode2));

    clNode2 = getLeftSubTree(crNode);
    crNode2 = getRightSubTree(crNode);
    printf("%d, %d \n", getData(clNode2), getData(crNode2));

    clNode3 = getLeftSubTree(crNode2);
    crNode3 = getRightSubTree(crNode2);
    printf("%d, %d \n", getData(clNode3), getData(crNode3));
    return 0;
}

/*
루트 노드 : 4 
2, 6 
1, 3 
5, 8 
7, 9 
*/
```

## ✅ AVLRebalance.h

```c
#ifndef AVLRebalance_h
#define AVLRebalance_h

#include <stdio.h>

BTreeNode* rebalance(BTreeNode ** pRoot);

#endif /* AVLRebalance_h */
```

## ✅ AVLRebalance.c

```c
#include <stdio.h>
#include "BinaryTree.h"


BTreeNode* rotateLL(BTreeNode* bst)
{
    BTreeNode* pNode;
    BTreeNode* cNode;

    pNode = bst;
    cNode = getLeftSubTree(pNode);

    changeLeftSubTree(pNode, getRightSubTree(cNode));
    changeRightSubTree(cNode, pNode);
    return cNode;
}


BTreeNode* rotateRR(BTreeNode * bst)
{
    BTreeNode* pNode;
    BTreeNode* cNode;

    pNode = bst;
    cNode = getRightSubTree(pNode);

    changeRightSubTree(pNode, getLeftSubTree(cNode));
    changeLeftSubTree(cNode, pNode);
    return cNode;
}


BTreeNode* rotateRL(BTreeNode * bst)
{
    BTreeNode* pNode;
    BTreeNode* cNode;

    pNode = bst;
    cNode = getRightSubTree(pNode);

    changeRightSubTree(pNode, rotateLL(cNode));
    return rotateRR(pNode);
}


BTreeNode* rotateLR(BTreeNode* bst)
{
    BTreeNode* pNode;
    BTreeNode* cNode;
    
    pNode = bst;
    cNode = getLeftSubTree(pNode);
    
    changeLeftSubTree(pNode, rotateRR(cNode));
    return rotateLL(pNode);
}

int getHeight(BTreeNode* bst) {
    int leftH;        // left height
    int rightH;        // right height

    if(bst == NULL)
        return 0;
    
    leftH = getHeight(getLeftSubTree(bst));
    rightH = getHeight(getRightSubTree(bst));

    if(leftH > rightH)
        return leftH + 1;
    else
        return rightH + 1;
}

int getHeightDiff(BTreeNode* bst) {
    int lsh;    // left sub tree height
    int rsh;    // right sub tree height

    if(bst == NULL)
        return 0;

    lsh = getHeight(getLeftSubTree(bst));
    rsh = getHeight(getRightSubTree(bst));

    return lsh - rsh;
}

BTreeNode * rebalance(BTreeNode** pRoot) {
    int hDiff = getHeightDiff(*pRoot);

    if(hDiff > 1)
    {
        if(getHeightDiff(getLeftSubTree(*pRoot)) > 0)
            *pRoot = rotateLL(*pRoot);
        else
            *pRoot = rotateLR(*pRoot);
    }
    
    if(hDiff < -1)
    {
        if(getHeightDiff(getRightSubTree(*pRoot)) < 0)
            *pRoot = rotateRR(*pRoot);
        else
            *pRoot = rotateRL(*pRoot);
    }
    
    return *pRoot;
}

```

## ✅ BinarySearchTree.h

```c
#ifndef BinarySearchTree_h
#define BinarySearchTree_h

#include "BinaryTree.h"

typedef BTData BSTData;

void makeAndInitBST(BTreeNode** pRoot); // 이진 탐색 트리의 생성 및 초기화

BSTData getNodeDataBST(BTreeNode* bst); // 노드에 저장된 데이터 반환

BTreeNode* insertBST(BTreeNode** pRoot, BSTData data);    // 이진 탐색 트리를 대상으로 데이터 저장

BTreeNode* searchBST(BTreeNode* bst, BSTData target);   // 이진 탐색 트리를 대상으로 데이터 탐색

BTreeNode* removeBST(BTreeNode** pRoot, BSTData target);    // 트리에서 노드를 제거하고 제거된 노드의 주소 값을 반환

void showALLBST(BTreeNode* bst);    // 이진 탐색 트리에 저장된 모든 노드의 데이터를 출력

void showIntData(int data);
```

## ✅ BinarySearchTree.c
insert 함수와 remove 함수에 리벨런싱 적용   
기존의 insert함수를 사용할 경우 루트 노드가 변경될 우려가 있으며 모든 상황에서 리벨런싱이 적용되는 것은 아니다.   
이에 모든 상황에서 적용이 가능하게 하려면 재귀적으로 올라가면서 각각 리벨런싱을 적용해줘야 한다.
```c
#include <stdio.h>
#include <stdlib.h>
#include "BinaryTree.h"
#include "BinarySearchTree.h"
#include "AVLRebalance.h"

void makeAndInitBST(BTreeNode** pRoot) {
    *pRoot = NULL;
}

BSTData getNodeDataBST(BTreeNode* bst) {
    return getData(bst);
}

BTreeNode* insertBST(BTreeNode** pRoot, BSTData data) {
    if(*pRoot == NULL) {
        *pRoot = makeBTreeNode();
        setData(*pRoot, data);
    }
    else if(data < getData(*pRoot)) {
        insertBST(&((*pRoot)->left), data);
        *pRoot = rebalance(pRoot);
    }
    else if(data > getData(*pRoot)) {
        insertBST(&((*pRoot)->right), data);
        *pRoot = rebalance(pRoot);
    }
    else
        return NULL;
    
    return *pRoot;
}

BTreeNode* searchBST(BTreeNode* bst, BSTData target) {
    BTreeNode* cNode = bst;
    BSTData cdata;
    
    while(cNode != NULL) {
        cdata = getData(cNode);
        
        if(cdata == target)
            return cNode;
        else if(cdata > target)
            cNode = getLeftSubTree(cNode);
        else
            cNode = getRightSubTree(cNode);
    }
    return NULL;
}

BTreeNode* removeBST(BTreeNode** pRoot, BSTData target) {
    BTreeNode* pVRoot = makeBTreeNode();    // 가상 root node
    BTreeNode* pNode = pVRoot;              // parent node
    BTreeNode* cNode = *pRoot;              // current node
    BTreeNode* dNode;                       // delete node
    
    // 루트 노드를 pVRoot가 가리키는 노드의 오른쪽 자식 노드가 되게 한다.
    changeRightSubTree(pVRoot, *pRoot);
    
    // 삭제 대상인 노드 탐색
    while(cNode != NULL && getData(cNode) != target) {
        pNode = cNode;
        if(target < getData(cNode))
            cNode = getLeftSubTree(cNode);
        else
            cNode = getRightSubTree(cNode);
    }
    
    if(cNode == NULL)   // 삭제 대상이 존재하지 않는다면
        return NULL;
    
    dNode = cNode;      // 삭제 대상을 dNode가 가리키게 한다.
    
    // 첫 번째 경우 : 삭제 대상이 단말 노드인 경우
    if(getLeftSubTree(dNode) == NULL && getRightSubTree(dNode) == NULL) {
        if(getLeftSubTree(pNode) == dNode)
            removeLeftSubTree(pNode);
        else
            removeRightSubTree(pNode);
    }
    // 두 번째 경우 : 삭제 대상이 하나의 자식 노드를 갖는 경우
    else if(getLeftSubTree(dNode) == NULL || getRightSubTree(dNode) == NULL) {
        BTreeNode* dcNode;  // 삭제 대상의 자식 노드 가리킴
        
        if(getLeftSubTree(dNode) != NULL)
            dcNode = getLeftSubTree(dNode);
        else
            dcNode = getRightSubTree(dNode);
        
        if(getLeftSubTree(pNode) == dNode)
            changeLeftSubTree(pNode, dcNode);
        else
            changeRightSubTree(pNode, dcNode);
    }
    // 세 번째 경우 : 두 개의 자식 노드를 모두 갖는 경우
    else {
        BTreeNode* mNode = getRightSubTree(dNode);  // 대체 노드 가리킴
        BTreeNode* mpNode = dNode;                  // 대체 노드의 부모 노드 가리킴
        int delData;
        
        // 삭제 대상의 대체 노드를 찾는다.
        while(getLeftSubTree(mNode) != NULL) {
            mpNode = mNode;
            mNode = getLeftSubTree(mNode);
        }
        
        // 대체 노드에 저장된 값을 삭제할 노드에 대입한다.
        delData = getData(dNode);       // 대입 전에 데이터 백업
        setData(dNode, getData(mNode)); // 대입
        
        // 대체 노드의 부모 노드와 자식 노드를 연결
        if(getLeftSubTree(mpNode) == mNode)
            changeLeftSubTree(mpNode, getRightSubTree(mNode));
        else
            changeRightSubTree(mpNode, getRightSubTree(mNode));
        
        dNode = mNode;
        setData(dNode, delData);    // 백업 데이터 복원
    }
    
    // 삭제된 노드가 루트 노드인 경우에 대한 추가적인 처리
    if(getRightSubTree(pVRoot) != *pRoot)
        *pRoot = getRightSubTree(pVRoot);   // 루트 노드의 변경을 반영
    
    free(pVRoot);   // 가상 루트 노드의 소멸
    *pRoot = rebalance(pRoot);
    return dNode;   // 삭제 대상의 반환
}

void showALLBST(BTreeNode* bst) {
    inorderTraverse(bst, showIntData);
}

void showIntData(int data) {
    printf("%d ", data);
}

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


BTreeNode* removeLeftSubTree(BTreeNode* bt);    // 왼쪽 자식 노드를 트리에서 제거, 제거된 노드의 주소 값이 반환
BTreeNode* removeRightSubTree(BTreeNode* bt);   // 오른쪽 자식 노드를 트리에서 제거, 제거된 노드의 주소 값이 반환
void changeLeftSubTree(BTreeNode* main, BTreeNode* sub);    // 메모리 소멸을 수반하지 않고 main의 왼쪽 자식 노드를 변경
void changeRightSubTree(BTreeNode* main, BTreeNode* sub);   // 메모리 소멸을 수반하지 않고 main의 오른쪽 자식 노드를 변경

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
        free(main->left);   // 해당 노드만 free해주는 것이므로 메모리 누수가 발생할 수 있다.
    
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

BTreeNode* removeLeftSubTree(BTreeNode* bt) {
    BTreeNode* delNode;
    
    if(bt == NULL)
        return NULL;
    
    delNode = bt->left;
    bt->left = NULL;

    return delNode;
}

BTreeNode* removeRightSubTree(BTreeNode* bt) {
    BTreeNode* delNode;
    
    if(bt == NULL)
        return NULL;
    
    delNode = bt->right;
    bt->right = NULL;

    return delNode;
}

void changeLeftSubTree(BTreeNode* main, BTreeNode* sub) {
    main->left = sub;
}
void changeRightSubTree(BTreeNode* main, BTreeNode* sub) {
    main->right = sub;
}
```

## Reference
- 윤성우의 열혈 C 자료구조
