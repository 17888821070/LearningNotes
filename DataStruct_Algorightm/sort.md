# 排序

## 一些概念 

![](../Picture/DataStruct/sort/01.jpg)

- 排序算法的内存消耗：针对排序算法的空间复杂度，引入原地排序概念，即空间复杂度 O(1) 的排序算法

- 排序算法的稳定性：待排序的序列中存在值相等的元素，经过排序之后，相等元素之间原有的先后顺序不变

- 有序度、逆序度：对于包含n个数据的数组，有 n! 种排列方式；
有序度：数组中具有有序关系的元素对的个数(逆序数)；我们把完全有序的数组称为满有序度

```
// 有序元素对：a[i] <= a[j]  i < j
// 完全有序的数组：有序度 n * (n - 1) / 2
// 逆序元素对：a[i] > a[j]  i < j
// 逆序度 = 满有序度 - 有序度
```
排序的过程就是增加有序度，减少逆序度的过程，最后达到满有序度

||稳定排序|
|-|-|
冒泡|√
插入|√
选择|√
归并|√
快排|×

## 冒泡排序（Bubble Sort）

冒泡排序只会操作相邻的两个数据；每次冒泡操作都会对相邻的两个元素进行比较，看是否满足大小关系要求，如果不满足就让它们交换；

一次冒泡会让至少一个元素移动到它应该在的位置，重复 n 次，就完成了 n 个数据的排序工作

当某次冒泡操作没有进行数据交互时，说明已经排序完成了，可以不再继续冒泡操作了

```cpp
void bubblesort(vector<int>& v) {
	int size = v.size();
	for (int i = 1; i < size; ++i) {
		bool flag = false;
		for (int j = 0; j < size - i; ++j) {
			if (v[j] > v[j + 1]) {
				swap(v[j], v[j + 1]);
				flag = true;
			}
		}
		if (!flag) break;
	}
}
```

 - 冒泡排序只涉及相邻数据的交换操作，只需要常量级的临时空间，所以空间复杂度为 O(1)，故是原地排序
 - 冒泡排序在相邻两个元素大小相等时不做交换，所以相同大小的数据在排序前后不会改变顺序，故冒泡排序是稳定的排序算法
 - 时间复杂度：平均时间复杂度 O(n^2)；要排序的数据是反序，则需要进行 n 次冒泡操作，所以最坏复杂度 O(n^2)；要排序的数据已经有序，则只需要进行一次冒泡操作，所以最好复杂度 O(n)
   
 
##  插入排序 (Insertion Sort)

![](../Picture/DataStruct/sort/02.jpg)

我们将数组中的数据分为两个区间，已排序区间和未排序区间

插入排序的核心思想是取未排序区间中的元素，在已排序区间中找到合适的插入位置将其插入，并保证已排序区间数据一直有序；重复这个过程，直到未排序区间中元素为空

初始已排序区间只有一个元素

![](../Picture/DataStruct/sort/03.jpg)

插入排序包含两种操作，元素的比较和元素的移动；对于一个给定的初始序列，移动操作的次数总是固定的，即逆序度
  
```cpp
void insertsort(vector<int>& v) {
	int size = v.size();
	for (int i = 1; i < size; ++i) {
		int tmp = v[i];
		for (int j = i - 1; j >= 0; --j) {
			if (v[j + 1] < v[j]) {
				swap(v[j + 1], v[j]);
			}
			else break;
		}
	}
}
```

- 插入排序是原地排序
- 插入排序是稳定排序
- 复杂度：平均时间复杂度 O(n^2)；要排序的数据是反序，则最坏复杂度 O(n^2)；要排序的数据已经有序，则最好复杂度 O(n)


## 选择排序（Selection Sort）
选择排序也分已排序区间和未排序区间，但选择排序每次会从未排序区间中找到最小的元素，将其放到已排序区间的末尾;每次操作确定一个元素的正确位置

![](../Picture/DataStruct/sort/04.jpg)


```cpp
void selectsort(vector<int>& v) {
	int size = v.size();
	for (int i = 0; i < size; ++i) {
		int pos = i;
		for (int j = i; j < size; ++j) {
			if (v[j] < v[pos]) {
				pos = j;
			}
		}
		swap(v[i], v[pos]);
	}
}
```

- 选择排序是原地排序
- 选择排序是稳定排序
- 复杂度：平均时间复杂度 O(n^2)，最好最坏复杂度都是 O(n^2)


## 归并排序
核心思想就是分治，将一个大问题分解成小的子问题；使用递归来实现

![](../Picture/DataStruct/sort/05.jpg)

```cpp
void merge(vector<int>& v, int start, int end) {
	if (end <= start) return;
	int mid = start + (end - start) / 2;
	merge(v, start, mid);
	merge(v, mid + 1, end);
	vector<int> tmp(end - start + 1);
	int i = start, j = mid + 1, k = 0;
	while (k < end - start + 1) {
		if(i < mid + 1 && j < end + 1) {
			if (v[i] < v[j]) {
				tmp[k++] = v[i++];
			}
			else {
				tmp[k++] = v[j++];
			}
		}
		else if (i < mid + 1) {
			while (i < mid + 1) {
				tmp[k++] = v[i++];
			}
		}
		else if (j < end + 1) {
			while (j < end + 1) {
				tmp[k++] = v[j++];
			}
		}
	}
	for (int p = start; p <= end; ++p) {
		v[p] = tmp[p - start];
	}
}

void mergesort(vector<int>& v) {
	merge(v, 0, v.size() - 1);
}
```

- 归并排序是稳定排序
- 时间复杂度：递归时，把一个问题分解成多个子问题，那时间复杂度等于多个子问题的时间复杂度加上将多个子问题合并的时间复杂度；归并排序最好、最坏和平均时间复杂度都是 O(nlogn)
- 空间复杂度：归并排序的合并函数需要借助额外的存储空间，空间复杂度为 O(n)


## 快速排序
如果要排序数组中下标从 p 到 r 之间的一组数据，我们选择中间任意一个数组作为 pivot（分区点）；遍历 p 到 r 之间的数据，将小于 pivot 的放左边，大于 pivot 的放右边，将 pivot 放中间

![](../Picture/DataStruct/sort/06.jpg)

原地分区过程：

![](../Picture/DataStruct/sort/07.jpg)

![](../Picture/DataStruct/sort/08.jpg)

```cpp
int partition(vector<int>& v, int start, int end) {
	int pos = start;
	for (int i = start; i < end; ++i) {
		if (v[i] < v[end]) {
			swap(v[pos++], v[i]);
		}
	}
	swap(v[pos], v[end]);
	return pos;
}

void quicksort(vector<int>& v, int start, int end) {
	if (end <= start) return;
	int p = partition(v, start, end);
	quicksort(v, start, p - 1);
	quicksort(v, p + 1, end);
}

void quicksort(vector<int>& v) {
	quicksort(v, 0, v.size() - 1);
}
```


- 快排原地不稳定排序
- 空间复杂度 O(1)
- 当 pivot 选择合适，每次将区间分为大小相近的两个小区间，则快排最好时间复杂度是 O(nlogn)；当 pivot 选择不合适，每次分区得到的两个区间都不均等，需要进行 n 次分区操作，则最坏时间复杂度 O(n^2)，快排时间复杂度退化到 O(n^2)的概率非常小，可以通过合理的选择 pivot 来避免这种情况；平均时间复杂度是 O(nlogn)

这种 O(n^2) 时间复杂度出现的主要原因还是因为我们分区点选得不够合理，最理想的分区点是：被分区点分开的两个分区中，数据的数量差不多。改进方法：

1. 从区间的首、尾、中间，分别取出一个数，然后对比大小，取这 3 个数的中间值作为分区点。但是，如果要排序的数组比较大，那三数取中可能就不够了，可能要五数取中或者十数取中

2. 每次从要排序的区间中随机选择一个元素作为分区点。这种方法并不能保证每次分区点都选的比较好，但是从概率的角度来看，也不大可能会出现每次分区点都选得很差的情况，所以平均情况下，这样选的分区点是比较好的
  

## 桶排序
核心思想是将要排序的数据分到几个有序的桶里，每个桶里的数据在单独进行排序；桶内排完序之后再把每个桶里的数据按照顺序依次去除，组成的序列就是有序的了

![](../Picture/DataStruct/sort/09.jpg)

- 在每个桶里进行快速排序

- 桶排序的时间复杂度接近 O(n)

- 桶排序对排序数据的要求非常苛刻：要排序的数据需要很容易地划分成 m 个桶，并且桶与桶之间有着天然的大小顺序，这样每个桶内的数据排序完后桶与桶之间的数据不需要再进行排序；数据在各个桶之间的分布是比较均匀的，在低端情况下会退化为 O(nlogn) 的排序

- 桶排序比较适合用于外部排序中，即数据存储在外部磁盘中，数据量比较大，内存有限，无法将数据全部加载到内存中
  
## 计数排序
- 当要排序 n 个数据，所处的范围并不大的时候，比如最大值为 k，我们就可以把数据划分 k 个桶，每个桶内的数据值都是相同的，省掉了桶内排序的时间
- 因为只涉及扫描遍历操作，所有时间复杂度是 O(n)
- 计数排序只能对非负整数排序；如果要排序的数据是其他类型的，要将其在不改变相对大小的情况下，转化为非负整数

![](../Picture/DataStruct/sort/10.jpg)
  
## 基数排序
  
基数排序对要排序的数据是有要求的，需要能分割出独立的位来比较，而且位之间有递进关系，如果高位的数据大则剩下地位则不需要比较；而且，每位的数据范围不能大，能采用线性排序算法来排序