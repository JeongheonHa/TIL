# ✅ 정렬

## ✅ 버블 정렬
- 데이터의 비교횟수 : 최악 - `O(n^2)`, 최선 - `O(n^2)`
- 데이터의 이동횟수 : 최악 - `O(n^2)`, 최선 - x    
(최악의 경우 for문 안에서 이동이 이루어지므로 비교횟수보다 3배 많다.)

<img src = "https://user-images.githubusercontent.com/108064146/195115638-4ce7e64f-95b8-4a22-8620-4abdfc30b95c.jpeg">
</br>

```c
#include <stdio.h>

void bubbleSort(int arr[], int n) {
    int i, j;
    int temp;
    
    for(i=0; i<n-1; i++) {
        for(j=0; j<n-j-1; j++) {
            if(arr[j] > arr[j+1]) {
                temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

int main(void) {
    int arr[4] = {3, 2, 1, 4};
    int i;
    
    bubbleSort(arr, sizeof(arr)/sizeof(int));
    
    for(i=0; i<4; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
1 2 3 4
*/
```

## ✅ 선택 정렬
- 데이터의 비교횟수 : 최악 - `O(n^2)` , 최선 - `O(n^2)`
- 데이터의 연산횟수 : 최악 - `O(n)`   , 최선 - `O(n)`(for문 밖에서 연산이 이루어지므로 `3(n-1)`)

<img src = "https://user-images.githubusercontent.com/108064146/195115933-c94f7bf6-4834-4225-9e32-3c9cee53ffb0.jpeg">
</br>

```c
#include <stdio.h>

void selectionSort(int arr[], int n) {
    int i, j;
    int maxIdx;
    int temp;
    
    for(i=0; i<n-1; i++) {
        maxIdx = i;
        for(j=i+1; j<n; j++) {
            if(arr[j] < arr[maxIdx])
                maxIdx = j;
        }
        
        temp = arr[i];
        arr[i] = arr[maxIdx];
        arr[maxIdx] = temp;
    }
}

int main(void) {
    int arr[4] = {3, 2, 1, 4};
    int i;
    
    selectionSort(arr, sizeof(arr)/sizeof(int));
    
    for(i=0; i<4; i++)
        printf("%d ", arr[i]);
    
    printf("\n");
    return 0;
}

/*
1 2 3 4
*/
```

## ✅ 삽입 정렬
정렬대상이 대부분 정렬되어있는 경우 빠르게 작동한다.   
- 데이터의 비교횟수 : 최악 - `O(n^2)`, 최선 - x
- 데이터의 이동횟수 : 최악 - `O(n^2)`, 최선 - x

<img src = "https://user-images.githubusercontent.com/108064146/195116063-517dfd00-e916-48f8-b498-7ca0dec6de08.jpeg">
</br>

```c
#include <stdio.h>

void insertSort(int arr[], int n) {
    int i, j;
    int insData;
    
    for(i=1; i<n; i++) {
        insData = arr[i];
        
        for(j=i-1; i>=0 ; j--) {
            if(arr[j] > insData)
                arr[j+1] = arr[j];
            else
                break;
        }
        arr[j+1] = insData;
    }
}
```

## References
- 윤성우의 열혈 C 자료구조
