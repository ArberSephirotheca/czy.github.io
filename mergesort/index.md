# Algorithm - MergeSort

**这篇文章讨论了归并排序的原理**
<!--more-->
## 原理
1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列.

2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置.

3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置.

4. 重复步骤 3 直到某一指针达到序列尾.

5. 将另一序列剩下的所有元素直接复制到合并序列尾.

## 实现
```go
  func mergeSort(arr []int) []int{
    length := len(arr)
    if length < 2{
      return arr
    }
    mid := length/2
    left := arr[0:mid]
    right := arr[mid:]
    return merge(mergeSort(left), mergeSort(right))
  }

  func merge(left []int, right []int) []int{
    var result []int
     for len(left) != 0 && len(right) != 0{
       if left[0] < right[0]{
         result = append(result, left[0])
         left = left[1:]
       }else{
         result = append(result, right[0])
         right = right[1:]
       }

       for len(left) != 0{
         result = append(result, left[0])
         left = left[1:]
       }
       for len(right) != 0{
         result = append(result, right[0])
         right = right[1:]
       }
     }
     return result
  }
```

## 时间复杂度
* Average Case: **O(nlogn)**
* Worst Case: **O(nlogn)** 

## 优点
* 时间复杂度稳定

## 缺点
* 虽然 Average Case 都是**O(nlogn)**，但QuickSort比归并排序稍快
