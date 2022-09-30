## ✅ MaxHeap 구현

```python
class MaxHeap:
    def __init__(self, L = []):
        self.A = L
        self.make_heap()
        
    def heapify_down(self, k, n):   # k : 인덱스, n : 힙의 전체 노드의 수
        while k*2+1 < n:
            L, R = 2*k + 1, 2*k + 2
            if self.A[L] > self.A[k]:
                m = L
            else:
                m = k
            if R < n and self.A[R] > self.A[m]:
                m = R
            if m != k:
                self.A[k], self.A[m] = self.A[m], self.A[k]
                k = m
            else:
                break
            
    def make_heap(self):
        n = len(self.A)
        for k in range(n-1, -1, -1):
            self.heapify_down(k, n)
            
    def heap_sort(self):
        n = len(self.A)
        for k in range(len(self.A)-1, -1, -1):
            self.A[0], self.A[k] = self.A[k], self.A[0]
            n = n - 1
            self.heapify_down(0, n)
            
    def heapify_up(self, k):
        while k > 0 and self.A[k] > self.A[(k-1)//2]:
            self.A[k], self.A[(k-1)//2] = self.A[(k-1)//2], self.A[k]
            k = (k-1)//2
            
    def insert(self, key):
        self.A.append(key)
        self.heapify_up(len(self.A)-1)
        
    def delete_max(self):
        if len(self.A) == 0: return None
        key = self.A[0]
        self.A[0], self.A[len(self.A)-1] = self.A[len(self.A)-1], self.A[0]
        self.A.pop()
        heapify_down(0, len(self.A))
        return key
    
if __name__ == "__main__":
    mh = MaxHeap([3, 5, 1, 2, 7, 9, 10, 14])
    mh.insert(4)
    mh.insert(41)
    print(mh.A)
    mh.heap_sort()
    print(mh.A)
```
