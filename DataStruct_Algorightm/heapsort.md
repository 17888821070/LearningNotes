# 堆和堆排序

堆排序是一种原地的、时间复杂度 O(nlogn)、非稳定的排序

## 堆

堆是一种特殊的树

1. 堆是一个完全二叉树；除了最后一层，其他层的节点个数都是满的，最后一层的节点都靠左排列
2. 堆中每一个节点的值都大于等于（或小于等于）其子树中每个节点的值；每个节点的值都大于等于子树中每个节点值的堆为“大顶堆”，每个节点都小于等于子节点的堆为“小顶堆”


### 实现
完全二叉树比较适合用数组来存储，不需要存储左右节点的指针，单纯通过数组的下标就可以找到一个节点的左右子节点和父节点
数组下标为 i 的节点，左子节点下标为 2 * i，右子节点下标为 2 * i + 1，父节点下标为 i / 2

![](../Picture/DataStruct/heap/01.jpg)

### 插入
往堆中插入一个元素后，需要进行调整让其满足堆的特性，叫做堆化

顺着节点所在路径向上或向下，对比然后交换

从下往上的堆化：

![](../Picture/DataStruct/heap/02.jpg)

### 删除堆顶元素

把最后一个节点放到堆顶，然后利用父子节点对比方法，互换两个节点，直到父子节点之间满足大小关系为止

从上往下的堆化：

![](../Picture/DataStruct/heap/03.jpg)

### 复杂度
堆化的时间复杂度 O(logn)，插入和删除堆顶元素的主要逻辑是堆化所以时间复杂度也是 O(logn)

堆排序的建堆过程时间复杂度 O(n)

```cpp
class heap {
private:
	vector<int> h;

public:
	heap() : h(1) {}

	void push(int n) {
		h.push_back(n);
		int pos = h.size() - 1;
		while (pos / 2 > 0) {
			if (h[pos] > h[pos / 2]) swap(h[pos], h[pos / 2]);
			pos /= 2;
		}
	}

	void pop() {
		int pos = h.size() - 1;
		swap(h[pos], h[1]);
		h.pop_back();
		pos = 1;
		while (1) {
			if (2 * pos >= h.size()) {
				break;
			}
			else if (2 * pos + 1 >= h.size()) {
				if (h[pos] < h[pos * 2]) {
					swap(h[pos], h[2 * pos]);
				}
				break;
			}
			else {
				if (h[2 * pos] > h[2 * pos + 1]) {
					if (h[pos] < h[2 * pos]) {
						swap(h[pos], h[2 * pos]);
						pos = 2 * pos;
					}
					else break;
				}
				else {
					if (h[pos] < h[2 * pos + 1]) {
						swap(h[pos], h[2 * pos + 1]);
						pos = 2 * pos + 1;
					}
					else break;
				}
			}
		}
	}
};
```

## 堆排序

堆排序时间复杂度非常稳定，是 O(nlogn)，并且还是原地排序算法

堆排序的过程大致分解成两个大的步骤，建堆和排序

### 建堆
将数组原地建成一个堆，不借助另一个数组，在原数组上操作

- 第一种方法：借助堆插入思路，假设堆中只包含一个数据，就是下标为 1 的数据；然后调用插入操作，将下标从 2 到 n 的数据依次插入到堆中；每个数据插入堆中，都是从下往上堆化

- 第二种方法：从后往前处理数组，每个数据都是从上往下堆化；因为叶子节点往下堆化只能跟自己比较，所以从第一个非叶子节点开始，即下标从 n/2 开始到 1 的数据进行堆化，下标 n/2+1 到 n 的节点都是叶子节点，不需要堆化；

![](../Picture/DataStruct/heap/04.jpg)

![](../Picture/DataStruct/heap/05.jpg)

### 排序
建堆结束之后，数组中的数据已经按照大顶堆的特性组织的；把数组中第一个元素跟最后一个元素交换，最大元素就放到了下标为 n 的位置，类似于删除堆顶元素操作；堆顶元素移除后把下标为 n 的元素放到堆顶，然后再堆化，以此类推完成排序

![](../Picture/DataStruct/heap/06.jpg)

```cpp
void buildsort(vector<int>& v, int n) {
	// 1 从下往上
	//for (int i = 1; i < n; ++i) {
	//	int j = i;
	//	while (j > 0) {
	//		int p = (j - 1) / 2;
	//		if (v[p] < v[j]) {
	//			swap(v[p], v[j]);
	//			j = p;
	//		}
	//		else break;
	//	}
	//}

	// 2 从上往下
	for (int i = n / 2 - 1; i >= 0; --i) {
		int j = i;
		while (j < n / 2) {
			int l = 2 * j + 1, r = 2 * j + 2;
			if (r < n) {
				int tmp;
				if (v[l] < v[r]) tmp = r;
				else tmp = l;
				if (v[j] < v[tmp]) {
					swap(v[j], v[tmp]);
					j = tmp;
				}
				else break;
			}
			else if (l < n) {
				if (v[j] < v[l]) {
					swap(v[j], v[l]);
					j = l;
				}
				else break;
			}
		}
	}
}

void heapsort(vector<int>& v) {
	int size = v.size();
	for (int i = 0; i < size; ++i) {
		buildsort(v, v.size() - i);
		swap(v[size - 1 - i], v[0]);
	}
}
```

## 与快排比较
1. 堆排序数据访问方式没有快排友好：堆排跳着访问数据，快排顺序访问数据，对CPU缓存不友好
2. 对于同样的数据，在排序过程中，堆排数据交换次数多于快排

## 应用

- 优先级队列

优先级队列首先是一个队列，但出队的顺序不是先进先出，而是按照优先级，优先级最高的最先出队

一个堆可以看做一个优先级队列，往优先级队列中插入元素相当于往堆中插入一个元素，从优先级队列中取出有优先级最高的元素相当于取出堆顶元素

- 求Top K

- 求中位数
