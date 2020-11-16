# 回溯算法

## 定义

回溯算法很多时候都应用在“搜索”这类问题上。不过这里说的搜索，并不是狭义的指我们前面讲过的图的搜索算法，而是在一组可能的解中，搜索满足期望的解

回溯的处理思想，有点类似枚举搜索，枚举所有的解，找到满足期望的解

为了有规律地枚举所有可能的解，避免遗漏和重复，可以把问题求解的过程分为多个阶段。每个阶段，都会面对一个岔路口，先随意选一条路走，当发现这条路走不通的时候（不符合期望的解），就回退到上一个岔路口，另选一种走法继续走

回溯算法非常适合用递归代码实现

经典问题：八皇后

```cpp
vector<vector<string>> solveNQueens(int n) {
	if(n < 1) return {{}};
	if(n == 1) return {{"Q"}};
	
	vector<vector<string>> ans;
	vector<int> board(n, -1);
	
	function<bool(int)> check = [&](int r) {
		for(int i = 0; i < r; ++i) {
			if(board[i] == board[r]) return false;
			if(board[r] - board[i] == r - i) return false;
			if(board[r] - board[i] == i - r) return false;
		}
		return true;
	};
	
	function<void(int)> helper = [&](int r) {
		if(n == r) {
			vector<string> temp(n, string(n, '.'));
			for(int i = 0; i < n; ++i) {
				temp[i][board[i]] = 'Q';
			}
			ans.push_back(temp);
		}
		else {
			for(int i = 0; i < n; ++i) {
				board[r] = i;
				if(check(r)) helper(r + 1);
			}
		}
	};
	
	helper(0);
	return ans;
}
```

## 经典应用

### 0-1 背包

0-1 背包是非常经典的算法问题，很多场景都可以抽象成这个问题模型，经典解法是动态规划，不过采用回溯算法更简单但没有那么高效的解法

有一个背包，背包总的承载重量是 W kg。现在我们有 n 个物品，每个物品的重量不等，并且不可分割，期望选择几件物品，装载到背包中，在不超过背包所能装载重量的前提下，如何让背包中物品的总重量最大

可以把物品依次排列，整个问题就分解为了 n 个阶段，每个阶段对应一个物品怎么选择，先对第一个物品进行处理，选择装进去或者不装进去，然后再递归地处理剩下的物品