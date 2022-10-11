# ✅ 병합 정렬
`분할` -> `정복` -> `결합`의 단계를 거치는 `분할 정복법`에 근거한 알고리즘   
임시 메모리가 필요하다는 단점이 있지만 정렬 대상이 배열이 아닌 연결 리스트의 경우 단점이 되지 않는다. 따라서 연결 리스트에서 더 좋은 성능을 갖는다.
- 데이터의 비교연산 횟수 : `O(nlog n)`
- 데이터의 이동연산 횟수 : `O(nlog n)` (최악, 평균, 최선 상관없이)

<img src = "https://user-images.githubusercontent.com/108064146/195123797-39ef2b5d-3055-4fe4-9b2c-6a6f8558a1b4.PNG">
</br>

## ✅ main

```c
#include <stdio.h>
#include <stdlib.h>

void mergeTwoArea(int arr[], int left, int mid, int right) {
    int fIdx = left;
    int rIdx = mid+1;
    int i;
    
    int* sortArr = (int*)malloc(sizeof(int)*(right+1)); // 병합 한 결과를 담을 sortArr 배열 동적할당
    int sIdx = left;
    
    while(fIdx <= mid && rIdx <= right) {
        if(arr[fIdx] <= arr[rIdx])          // 병할할 두 데이터를 비교하여 정렬 순서대로 sortArr에 옮겨담는다.
            sortArr[sIdx] = arr[fIdx++];
        else
            sortArr[sIdx] = arr[rIdx++];
        
        sIdx++;
    }
    
    if(fIdx > mid) {    // 배열의 앞부분이 모두 sortArr에 옮겨졌을 경우
        for(i=rIdx; i<=right; i++, sIdx++)    // 배열의 뒷부분 모두 sortArr에 옮긴다.
            sortArr[sIdx] = arr[i];
    } else {
        for(i=fIdx; i<=mid; i++, sIdx++)
            sortArr[sIdx] = arr[i];
    }
    
    for(i=left; i<=right; i++)  // sortArr의 데이터를 arr에 옮긴다.
        arr[i] = sortArr[i];
    
    free(sortArr);
}

void mergeSort(int arr[], int left, int right) {
    int mid;
    
    if(left < right) {
        mid = (left+right)/2;
        
        mergeSort(arr, left, mid);
        mergeSort(arr, mid+1, right);
        
        mergeTwoArea(arr, left, mid, right);
    }
}

int main(void) {
    int arr[7] = {3, 2, 4, 1, 7, 6, 5};
    int i;
    
    mergeSort(arr, 0, sizeof(arr)/sizeof(int)-1);
    
    for(i=0; i<7; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
1 2 3 4 5 6 7 
*/
```

## References
- 윤성우의 열혈 C 자료구조
