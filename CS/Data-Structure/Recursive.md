## ✅ 이진 탐색 재귀로 구현

```c
# include <stdio.h>

int RecursiveBinarySearch(int* arr, int first, int last, int target) {
    int mid;
    if(first > last) return -1;
    mid = (first+last)/2;
    if(arr[mid] == target) {
        return mid;
    } else if(target > arr[mid]) {
        return RecursiveBinarySearch(arr, mid+1, last, target);
    } else {
        return RecursiveBinarySearch(arr, first, mid-1, target);
    }
}

int main(void) {
    int arr[] = {1, 3, 5, 7, 9};
    int len = sizeof(arr)/sizeof(int);
    int idx = RecursiveBinarySearch(arr, 0, len-1, 7);
    if(idx == -1) {
        printf("탐색 실패 \n");
    } else {
        printf("타겟 저장 인덱스: %d\n", idx);
    }

    return 0;
}
```

## ✅ 피보나치 수열

```c
#include <stdio.h>

int Fibo(int n) {
    if(n == 1) {
        return 0;
    } else if(n == 2) {
        return 1;
    } else {
        return Fibo(n-1) + Fibo(n-2);
    }

}
int main(void) {
    for(int i = 1; i < 15; i++) {
        printf("%d ", Fibo(i));
    }
    printf("\n");
    return 0;
}
```

## ✅ 하노이 탑

```c
#include <stdio.h>

void hanoiMove(int n, char a, char b, char c) {
    if(n == 1) {
        printf("원반1을 %c에서 %c로 이동\n", a, c);
    } else {
        hanoiMove(n-1, a, c, b);
        printf("원반%d을(를) %c에서 %c로 이동\n", n, a, c);
        hanoiMove(n-1, b, a, c);
    }
}

int main(void) {
    hanoiMove(3, 'A', 'B', 'C');
    return 0;
}
```
