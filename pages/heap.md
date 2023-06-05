# 重新设计 heap 包

22 Jun 2020

```text
标准库 container/heap 包的设计比较特别，Api 并非面向对象的风格，
这和其他语言甚至 Go 自身的其他数据结构如 container/list 不一样
```

---

## 官方当前 Api 初体验

简单起见，我们先假设要管理一些整形数字；要同时用到大顶堆和小顶堆。怎么写呢？

- 定义
  需要先实现两个类型，MinHeap 和 MaxHeap，分别实现 heap.Interface 接口， 即 Len、Less、Swap、Push、Pop 五个方法：

  ```go
  // 小顶堆
  type MinHeap []int

  func (h MinHeap) Len() int { return len(h) }
  func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }
  func (h MinHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
  func (h *MinHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
  func (h *MinHeap) Pop() interface{} {
    x := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return x
  }
  ```

---

## 官方当前 Api 初体验

  ```go
  // 大顶堆
  type MaxHeap []int

  func (h MaxHeap) Len() int { return len(h) }
  func (h MaxHeap) Less(i, j int) bool { return h[i] > h[j] }
  func (h MaxHeap) Swap(i, j int) { h[i], h[j] = h[j], h[i] }
  func (h *MaxHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
  func (h *MaxHeap) Pop() interface{} {
    x := (*h)[len(*h)-1]
    *h = (*h)[:len(*h)-1]
    return x
  }
  ```

这里已经有明显的代码坏味道，重复代码太多；可以通过该自定义 heap 增加函数类型属性 cmp 来解决，这里不细说，后边重新设计后有类似实现。

---

## 官方当前 Api 初体验

- 使用

    ```go
    nums := []int{2, 9, 10, 7, 4, 3}
    minHeap := &MinHeap{}
    maxHeap := &MaxHeap{}
    for _, v := range nums {
    	heap.Push(minHeap, v)
    	heap.Push(maxHeap, v)
    }
    fmt.Println(heap.Pop(minHeap))
    fmt.Println(maxHeap[0])
    ```

  可以看到，没有 Peek 方法，直接取第 0 个元素即峰顶；push 和 pop 使用了 heap 包的 Push Pop 方法， 而不是直接这样：

    ```go
    minHeap.Push(5)
    maxHeap.Pop()
    ```

  综合看，步骤 1 里需要实现 heap.Interface 接口，并且步骤 2 使用的是 heap 包的 Push 和 Pop 方法，而不是类型本身的 Push 和 Pop 方法

---

## 分析修改 Api 设计

看起来标准库当前设计并不友好，尝试修改下，让 Api 更易用

1. 使用者只需关注比较逻辑

    ```text
    堆底层是一个切片， heap.Interface 里要求的五个方法 Len、Less、Swap、Push、Pop，
    有四个无需使用者关注，只有比较逻辑需要使用者确定
    这里可以定义一个只包含 Less 方法(改名为 Cmp 更好)的接口让使用者实现，
    或者直接给我们的结构体增加函数类型的 cmp 属性
    ```

2. Push 和 Pop 直接按照堆实例的方法调用

    ```text
    基于上条分析，Push 和 Pop 就可以直接按照堆实例的方法调用，而不用弄成一个包方法
    ```

---

## 分析修改 Api 设计
我们需要提供一个这样的 Heap：

```go
type Cmp func(i, j int) bool
type Any interface{}

type Heap interface {
    InitWithCmp(cmp Cmp)
    Get(i int) Any

    Push(x Any)
    Pop() Any
    Peek() Any
}
```

注意到有一些上面没有提到的方法，有的是为了方便使用，如 Peek、Len，有的是基于当前设计的实现需要。

---

## 分析修改 Api 设计

现在使用起来就是这样:

```go
func main()  {
    nums := []int{2, 9, 10, 7, 4, 3}
    minHeap := heap.NewWithCap(len(nums))
    minHeap.InitWithCmp(func(i, j int) bool {
        return minHeap.Get(i).(int) < minHeap.Get(j).(int)
    })
    maxHeap := heap.NewWithSlice(nil)
    maxHeap.InitWithCmp(func(i, j int) bool {
        return maxHeap.Get(i).(int) > maxHeap.Get(j).(int)
    })
    for _, v := range nums {
        minHeap.Push(v)
        maxHeap.Push(v)
    }
    fmt.Println(minHeap.Pop())
    fmt.Println(maxHeap.Peek())
}
```

先忽略 NewWithCap 和 NewWithSlice， 可以看到，使用者唯一需要确定的就是比较逻辑，即完成 InitWithCmp 方法就创建好了堆实例

---

## 分析修改 Api 设计

有两个实现方法

1. 包装标准库已有 Api
2. 从底层写起

---

## 分析修改 Api 设计

1. 包装标准库已有 Api

   ```go
   func New() Heap {
       return NewWithCap(0)
   }

   func NewWithCap(cap int) Heap {
       return &heapImp{inner: &helper{slice: make([]Any, 0, cap)}}
   }

   func NewWithSlice(slice []Any) Heap {
       return &heapImp{inner: &helper{slice: slice}}
   }

   type heapImp struct {
       inner *helper
   }

   func (h *heapImp) InitWithCmp(cmp Cmp) {
       h.inner.cmp = cmp
       heap.Init(h.inner)
   }
   ```

---

## 分析修改 Api 设计

  ```go
  func (h *heapImp) Get(i int) Any {
      return h.inner.slice[i]
  }

  func (h *heapImp) Len() int {
      return h.inner.Len()
  }

  func (h *heapImp) Peek() Any {
      return h.inner.slice[0]
  }

  func (h *heapImp) Push(x Any) {
      heap.Push(h.inner, x)
  }

  func (h *heapImp) Pop() Any {
      return heap.Pop(h.inner)
  }
  ```

---

## 分析修改 Api 

heapImp 的关键属性 inner 就是包装了标准库

```go
type helper struct {
    slice []Any
    cmp   Cmp
}

func (h *helper) Len() int           { return len(h.slice) }
func (h *helper) Less(i, j int) bool { return h.cmp(i, j) }
func (h *helper) Swap(i, j int)      { h.slice[i], h.slice[j] = h.slice[j], h.slice[i] }
func (h *helper) Push(x interface{}) {
    h.slice = append(h.slice, x)
}
func (h *helper) Pop() interface{} {
    n := len(h.slice)
    x := h.slice[n-1]
    h.slice = h.slice[:n-1]
    return x
}
```

---

## 分析修改 Api 设计

2. 从头实现

    详见[GitHub](https://github.com/zrcoder/dsGo/edit/master/base/heap/heap.go)

    ```text
    参考标准库核心方法 up 和 down ，从头写heapImp
    注意到还有些扩展 Api：
        Get用来获取指定索引元素，主要用在InitWithCmp方法中；
        Update 和 Remove 方法， 可用于更新或移除指定索引处的元素；
        IndexOf 用来获取指定元素在堆中的索引，获取索引后可继续调用Update 或 Remove方法。
    ```

    更多参考：
    - [intHeap 使用示例](https://github.com/zrcoder/dsGo/edit/master/base/heap/example_intheap_test.go)
    - [优先队列使用示例](https://github.com/zrcoder/dsGo/edit/master/base/heap/example_pq_test.go)
