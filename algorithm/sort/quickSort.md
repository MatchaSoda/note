# 快速排序 quickSort
### 性能
* 平均时间复杂度：O(nlogn)
* 最佳时间复杂度：O(nlogn)
* 最差时间复杂度：O(n2)
* 空间复杂度：根据实现方式的不同而不同
### 逻辑
使用分治法把一个序列分成两个。  
从数组中选出一个元素（最后一个），称为“基准”（pivot），通过交换使得基准左边的数都小于它，右边的数都大于它。  
递归地对左边的和右边的序列进行快排。  

![](https://pic2.zhimg.com/v2-d4e5d0a778dba725091d8317e6bac939_b.webp)
### 代码
```cpp
void quick_sort_recursive(vector<int>& arr, int start, int end) {
  if (start >= end) return;
  int mid = arr[end];
  int left = start, right = end - 1;
  while (left < right) {
    while (arr[left] < mid && left < right) left++;
    while (arr[right] >= mid && left < right) right--;
    swap(arr[left], arr[right]);
  }
  if (arr[left] < arr[end]) {
    left++;
  }
  swap(arr[left], arr[end]);
  quick_sort_recursive(arr, start, left - 1);
  quick_sort_recursive(arr, left + 1, end);
}
vector<int> sortArray(vector<int>& nums) {
  quick_sort_recursive(nums, 0, nums.size() - 1);
  return nums;
}
```