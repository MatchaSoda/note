# 堆排序 heapSort
### 性能
* 平均时间复杂度：O(nlogn)
* 最佳时间复杂度：O(nlogn)
* 最差时间复杂度：O(nlogn)
* 稳定性：不稳定
### 逻辑
创建一个堆。堆是一个完全二叉树。保证每个节点的值大于等于（小于等于）子节点的堆是大顶堆（小顶堆）。  
把堆顶和最后一个节点交换，固定最后一个节点为有序区。恢复堆。  
重复上一步，每次交换调整，就相当于找出当前堆的最大值放到了有序区，直到全部数据有序。  


![](https://www.runoob.com/wp-content/uploads/2019/03/heapSort.gif)

创建堆和恢复堆都使用了同一种方法，我们称它为**调整**。创建堆是从最后一个父节点开始，向前向上对每一个父节点进行调整；恢复堆只对堆顶节点进行调整（因为交换的过程破坏了该节点）。  
**调整**，即比较当前节点和两个子节点的大小，如果有大的子节点的话就和父节点交换。交换后，子堆可能不满足条件，需要重新调整，所以有了代码中 while 不断向下的过程。
### 代码
```cpp
void heapSort(vector<int>& nums, int start, int end) {
  int dad = start;
  int son = dad * 2 + 1;
  while (son <= end) {
    if (son + 1 <= end && nums[son] < nums[son + 1]) {
      son++;
    }
    if (nums[dad] >= nums[son]) {
      return;
    } else {//交换后，以子节点为堆顶的子堆需要调整
      swap(nums[son], nums[dad]);
      dad = son;
      son = dad * 2 + 1;
    }
  }
}
vector<int> sortArray(vector<int>& nums) {
  int size = nums.size();
  //初始化，从最后一个父节点开始，对每个结点都进行调整
  for (int i = size / 2 - 1; i >= 0; i--) {
    heapSort(nums, i, size - 1);
  }
  //将堆顶和（未排序的）最后一个节点交换，调整堆顶
  for (int i = size - 1; i > 0; i--) {
    swap(nums[0], nums[i]);
    heapSort(nums, 0, i - 1);
  }
  return nums;
}
```