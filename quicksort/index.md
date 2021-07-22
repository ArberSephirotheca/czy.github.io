# Algorithm - QuickSort

**这篇文章讨论了快排的原理**
<!--more-->
## 原理
1. 有一个大小为[0...n]的数组，我们要将它进行排序.
2. 选中数字中的一个元素当成pivot.
    1. 把所有小于pivot的元素放在**左边**.
    2. 把所有大于pivot的元素放在**右边**.
3. 对于pviot**左侧**的数组执行**第二步**的步骤.
4. 对于pviot**右侧**的数组进行**第二步**的步骤.
5. 重复以上步骤.

## 时间复杂度
* Average Case: **O(nlogn)**
* Worst Case: **O(n<sup>2</sup>)** 

## 基准的选择
  * 固定位置
      * 通常选择[首个]/[最后]元素作为基准
  * 三数取中(medium of three)
      * 取待排序数组中间，首部和尾部中**第二大**的元素作为基准
      * 实现:
          ```cpp
          void getmid(vecotr<int> arr, int l, int r){
            int mid = l + (r-l)/2;
            int index = 0;
            if(arr[l] <= arr[mid] && arr[l] >= arr[r])
              index = l;
            else if(arr[r] <= arr[mid] && arr[r] >= arr[l])
              index = r;
            else
              index = mid;
            //put medium value at the front
            swap(arr[l],arr[mid]);
          }
          ```

## 缺点
* 如果初始序列基本为有序，则时间复杂度属于**Worst Case**.
* **Pivot**的选取极大影响了快排的效率

## 实现
```cpp
int partitions(vector<int>&arr, int l, int r){
  int pivot = l + (r-l)/2;
  int i = l;
  int j = r;
  while(true){
    //find leftmost element greater than or equal to pivot
    while(arr[i] < pivot)
      i++;
    //find rightmost element less than or equal to pivot
    while(arr[j] > pivot)
      j--;
    //two pointers met
    if(i >= j)
      return j;
    swap(arr[i], arr[j]);
  }
  arr[l] = pivot;
  return l;
}
void quicksort(vector<int>&arr, int l, int r){
  if(l < r){
    int pivot = partitions(arr, l, h);
    quicksort(arr, l, pivot);
    quicksort(arr, pivot+1, r);
  }
}
```
