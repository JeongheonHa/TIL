# ✅ 퀵 정렬
`O(nlog n)`의 사간복잡도를 갖는 다른 정렬 알고리즘보다 평균적으로 가장 빠른 정렬의 속도를 보이는 알고리즘   
- 전체 데이터를 기준으로 중간에 해당하는 값을 피벗으로 할 때 좋은 성능을 보인다.   
- 데이터의 이동이 데이터의 비교보다 현저히 적게 일어나고, 병합정렬과 같이 별도의 메모리 공간을 요구하지 않는다.
- 비교연산 횟수 : 최선 : `O(nlog n)`, 평균 : 최선에 가까운 성능을 평균적으로 낸다. 최악 : `O(n^2)` (이미 정렬된 배열을 퀵 정렬할 경우 최악의 시간복잡도를 갖는다.)

<img src = "https://user-images.githubusercontent.com/108064146/195350249-696253b7-5705-4527-a148-27998a0f9d17.PNG">
</br>

## ✅ main

```c
#include <stdio.h>

void swap(int arr[], int idx1, int idx2) {
    int temp;
    temp = arr[idx1];
    arr[idx1] = arr[idx2];
    arr[idx2] = temp;
}

int medianOfThree(int arr[], int left, int right) {
    int samples[3] = {left, (left+right)/2, right};
    
    if(arr[samples[0]] > arr[samples[1]])
        swap(samples, 0, 1);
    if(arr[samples[1]] > arr[samples[2]])
        swap(samples, 1, 2);
    if(arr[samples[0]] > arr[samples[1]])
        swap(samples, 0, 1);
    
    return samples[1];  // 3개 중 중간 값 반환
}

int partition(int arr[], int left, int right) {
    int pIdx = medianOfThree(arr, left, right); // pivot을 중간값으로 시작
    int pivot = arr[pIdx];
    // int pivot = arr[left];
    int low = left+1;
    int high = right;
    
    swap(arr, left, pIdx);
    
    printf("pivot: %d \n", pivot);
    
    while(low<=high) {
        while(arr[low]<=pivot && low <= right)
            low++;
        while(arr[high]>=pivot && high >= left+1)
            high--;
        if(low<=high)
            swap(arr, low, high);
    }
    swap(arr, left, high);
    return high;
}



void quickSort(int arr[], int left, int right) {
    if(left<=right) {
        int pivot = partition(arr, left, right);
        
        quickSort(arr, left, pivot-1);
        quickSort(arr, pivot+1, right);
    }
}

int main(void) {
    int arr[9] = {3, 3, 3, 2, 4, 1, 7, 6, 5};
    
    int len = sizeof(arr)/sizeof(int);
    int i;
    
    quickSort(arr, 0, sizeof(arr)/sizeof(int)-1);
    
    for(i=0; i<len; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
pivot: 3 
pivot: 6 
pivot: 5 
pivot: 7 
1 2 3 3 3 4 5 6 7 
*/
```

## Reference
- 윤성우의 열혈 C 자료구조
