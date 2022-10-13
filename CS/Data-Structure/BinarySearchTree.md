# ✅ 이진 탐색 트리
이진 트리에 데이터의 저장 규칙을 더해 놓은 트리
- 노드에 저장된 키는 유일하다.
- 루트 노드의 키가 왼쪽 서브 트리를 구성하는 어떠한 노드의 키보다 크다.
- 루트 노드의 키가 오른쪽 서브 트리를 구성하는 어떤한 노드의 키보다 작다.
- 왼쪽과 오른쪽 서브 트리도 이진 탐색 트리이다.
- 왼쪽 자식 노드의 키 < 부모 노드의 키 < 오른쪽 자식 노드의 키 (항상 만족)

<img src = "https://user-images.githubusercontent.com/108064146/195615518-43cfddeb-55d1-4244-b14e-6073d22421b4.PNG">
<img src = "https://user-images.githubusercontent.com/108064146/195605604-cf817a03-c1bf-4281-986c-ed2afe798970.PNG">
<img src = "https://user-images.githubusercontent.com/108064146/195605710-c8b7bddb-5f83-4d25-a7aa-196a3b1e735a.PNG">
</br>

## ✅ main

```c
#include <stdio.h>
#include <stdlib.h>
#include "BinarySearchTree.h"

int main(void) {
    BTreeNode* bstRoot;
    BTreeNode* sNode;
    makeAndInitBST(&bstRoot);
    
    insertBST(&bstRoot, 5); insertBST(&bstRoot, 8);
    insertBST(&bstRoot, 1); insertBST(&bstRoot, 6);
    insertBST(&bstRoot, 4); insertBST(&bstRoot, 9);
    insertBST(&bstRoot, 3); insertBST(&bstRoot, 2);
    insertBST(&bstRoot, 7);
    
    showALLBST(bstRoot); printf("\n");
    sNode = removeBST(&bstRoot, 3);
    free(sNode);
    
    showALLBST(bstRoot); printf("\n");
    sNode = removeBST(&bstRoot, 8);
    free(sNode);
    
    showALLBST(bstRoot); printf("\n");
    sNode = removeBST(&bstRoot, 1);
    free(sNode);
    
    showALLBST(bstRoot); printf("\n");
    sNode = removeBST(&bstRoot, 6);
    free(sNode);
    
    showALLBST(bstRoot); printf("\n");
    
    return 0;
}

/*
1 2 3 4 5 6 7 8 9 
1 2 4 5 6 7 8 9 
1 2 4 5 6 7 9 
2 4 5 6 7 9 
2 4 5 7 9 
*/
```

## ✅ BinarySearchTree.h

```c
#ifndef BinarySearchTree_h
#define BinarySearchTree_h

#include "BinaryTree.h"

typedef BTData BSTData;

void makeAndInitBST(BTreeNode** pRoot); // 이진 탐색 트리의 생성 및 초기화

BSTData getNodeDataBST(BTreeNode* bst); // 노드에 저장된 데이터 반환

void insertBST(BTreeNode** pRoot, BSTData data);    // 이진 탐색 트리를 대상으로 데이터 저장

BTreeNode* searchBST(BTreeNode* bst, BSTData target);   // 이진 탐색 트리를 대상으로 데이터 탐색

BTreeNode* removeBST(BTreeNode** pRoot, BSTData target);    // 트리에서 노드를 제거하고 제거된 노드의 주소 값을 반환

void showALLBST(BTreeNode* bst);    // 이진 탐색 트리에 저장된 모든 노드의 데이터를 출력

void showIntData(int data);

#endif /* BinarySearchTree_h */
```

## ✅ BinarySearchTree.c

```c
#include <stdio.h>
#include <stdlib.h>
#include "BinaryTree.h"
#include "BinarySearchTree.h"

void makeAndInitBST(BTreeNode** pRoot) {
    *pRoot = NULL;
}

BSTData getNodeDataBST(BTreeNode* bst) {
    return getData(bst);
}

void insertBST(BTreeNode** pRoot, BSTData data) {
    BTreeNode* pNode = NULL;
    BTreeNode* nNode = NULL;
    BTreeNode* cNode = *pRoot;
    
    while(cNode != NULL) {
        if(data == getData(cNode))
            return ;
        pNode = cNode;
        if(data < getData(cNode))
            cNode = getLeftSubTree(cNode);
        else
            cNode = getRightSubTree(cNode);
    }
    
    nNode = makeBTreeNode();
    nNode->data = data;
    
    if(pNode != NULL) {
        if(data < getData(pNode))
            makeLeftSubTree(pNode, nNode);
        else
            makeRightSubTree(pNode, nNode);
    } else {
        *pRoot = nNode;
    }
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
