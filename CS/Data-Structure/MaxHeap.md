## ✅ MaxHeap 구현

```python
class MaxHeap:
    def __init__(self, L = []):
        self.queue = L
        self.make_heap()
        
    def heapify_down(self, k ,n):
        while 2*k+1 < n:    # left 자손 노드가 있는 동안 (leaf노드 까지), left노드는 있지만 right노드는 없는 경우 존재
            left = 2*k+1
            right = 2*k+2
            if self.queue[left] > self.queue[k]:
                max_index = left
            else:
                max_index = k
            if right < n and self.queue[right] > self.queue[max_index]: # right 자손노드가 있다면
                max_index = right
            if max_index != k:
                self.swap(max_index, k)
                k = max_index
            else:
                break   # make_heap에서 마지막 노드부터 역순으로 올라가기 때문에 break
                
    def make_heap(self):
        n = len(self.queue)
        for k in range(n-1, -1, -1):
            self.heapify_down(k, n)
            
    def heap_sort(self):
        n = len(self.queue)
        for k in range(n - 1, -1, -1):
            self.swap(0, k)
            self.heapify_down(0, k)
    
    def heapify_up(self, k):
        while k > 0:
            parent = (k-1)//2
            if self.queue[k] > self.queue[parent]:
                self.swap(k, parent)
                k = parent
            
    def insert(self, key):
        self.queue.append(key)
        self.heapify_up(len(self.queue)-1)
    
    def delete_max(self):
        n = len(self.queue)
        if n == 0: return None
        key = self.queue[0]
        self.swap(0, n-1)
        self.queue.pop()
        self.heapify_down(0, n-1)
        return key
            
        
    def swap(self, a, b):
        self.queue[a], self.queue[b] = self.queue[b], self.queue[a]
        
if __name__ == "__main__":
    h = MaxHeap([1,4,5,6,78,23,2,13,17])
    print(h.queue)
    h.delete_max()
    print(h.queue)
    h.insert(40)
    print(h.queue)
    h.heap_sort()
    print(h.queue)
```
