# 二叉树

## 树
树的每个元素叫做节点，用来连线相领节点之间的关系叫做父子关系

![此处输入图片的描述][1]

A 是 B 的父节点，B 是 A 的子节点，B、C、D 是兄弟节点，没有父节点的节点为根节点，没有子节点的节点为叶节点

![此处输入图片的描述][2]

节点的高度：节点到叶节点的最大路径（边数）

节点的深度：根节点到这个节点所经历的边的个数

节点的层数：节点的深度 +1

树的高度：根节点的高度

![此处输入图片的描述][3]



## 二叉树
每个节点最多有两个叉，即两个子节点，分别是左子节点和右子节点；并不要求每个节点都有两个子节点，有的子节点只有左节点，有的只有右节点

![此处输入图片的描述][4]

编号 2 的二叉树，叶节点全部在最底层，除了叶节点之外，每个节点都有左右两个子节点，为满二叉树

编号 3 的二叉树，叶节点在最底下两层，最后一层叶节点靠左排列，并且除了最后一层其他层的节点个数都要达到最大，为完全二叉树

### 存储一个树
- 基于指针或引用的二叉链式存储法
- 基于数组的顺序存储法

### 链式存储法
每个节点有三个字段，一个存储数据，另外两个指向左右节点的指针

 ![此处输入图片的描述][5]

### 顺序存储法：

把根节点存储在下标 `i = 1` 的位置，左子节点存储在 `2 * i = 2` 的位置，右子节点存储在 `2 * i + 1 = 3` 的位置
如果节点 X 存储在数组中下标 i 的位置，它的右节点存储在 2 * i 的位置，左节点存储在 2 * i + 1 的位置；反之，下标 i / 2 的位置是它的父节点

![此处输入图片的描述][6]

完全二叉树的顺序存储法，只会浪费一个下标0的存储位置；如果是非完全二叉树，会浪费比较多的存储空间

![此处输入图片的描述][7]

完全二叉树用数组存储是最节省空间的一种方式，它不需要额外的左右子节点的指针，也没有浪费多余的存储空间



## 二叉树的遍历

- 前序遍历：对于树中的任意节点来说，先打印这个节点，然后打印它的左子树，再打印它的右子树

- 中序遍历：先打印它的左子树，然后打印它自己，再打印它的右子树

- 后序遍历：先打印它的右子树，然后打印左子树，再打印它自己

```cpp
// 递推形式遍历
// 前序遍历的递推公式：
preOrder(r) = print r->preOrder(r->left)->preOrder(r->right)

// 中序遍历的递推公式：
inOrder(r) = inOrder(r->left)->print r->inOrder(r->right)

// 后序遍历的递推公式：
postOrder(r) = postOrder(r->left)->postOrder(r->right)->print r

// 使用栈遍历
/*https://blog.csdn.net/Monster_ii/article/details/82115772?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task*/
// 前序
stack<TreeNode *> s;
s.push(root);
while(!s.empty())
{
    TreeNode * node = s.top();
    s.pop();
    cout<<node->val<<endl;
    if(node->right) s.push(node->right);
    if(node->left)  s.push(node->left);
}

// 中序
std::stack<TreeNode *> sta;
sta.push(root);
while (!sta.empty())
{
	TreeNode * temp = sta.top();
	while (temp->left)
	{
		sta.push(temp->left);
		temp = temp->left;
	}
	while (!sta.empty())
	{
		TreeNode * node = sta.top();
		sta.pop();
		std::cout << node->val << std::endl;
		if (node->right)
		{
			sta.push(node->right);
			break;
		}
	}
}

// 逆序
std::stack<TreeNode *> sta;
sta.push(root);
TreeNode * last = nullptr;
while (!sta.empty())
{
	TreeNode * node = sta.top();
	while (node->left)
	{
		sta.push(node->left);
		temp = node->left;
	}
	while (!sta.empty())
	{
		node = sta.top();
        if(!node->right || last == node->right)
        {
            last = node;
            std::cout<<node->val<<std::endl;
            sta.pop();
        }
		else if (node->right)
		{
			sta.push(node->right);
			break;
		}
	}
}
```

遍历的时间复杂度跟节点个数n成正比，所以二叉树遍历时间复杂度是 O(n)


## 二叉查找树

二叉查找树支持快速查找一个数据，还支持快速插入、删除一个数据

二叉查找树：在树的任意一个节点，其左子树中的每个节点的值都小于这个节点的值，右子树节点的值都大于这个节点

![此处输入图片的描述][8]

### 查找操作
先取根节点，如果等于要查找的数据就返回；如果要查找的数据比根节点的值小，就去左子树中递归查找；如果要查找数据比根节点值大，就去右子树中递归查找

![此处输入图片的描述][9]
```cpp
struct Node
{
    int val_ = 0;
    Node * left_ = nullptr;
    Node * right_ = nullptr;

    ~Node()
    {
        if(left_ != nullptr)
        {
            delete left_;
        }
        if(right_ != nullptr)
        {
            delete right_;
        }
    }
};

class BinarySearchTree
{
private:
    Node * root_ = nullptr;

public:
    Node find(int data)
    {
        Node * p = root_;
        while (p != nullptr)
        {
            if (data < p.val_) p = p.left_;
            else if (data > p.data) p = p.right_;
            else return p;
        }
        return nullptr;
    }
}
```

###  插入操作
新插入的数据一般都是在叶子节点上，所以只需要从根节点开始，依次比较要插入的数据和节点的大小关系；如果要插入的数据比节点的数据大，并且节点的右子树为空就将新数据插入到右子节点的位置，如果不为空就继续遍历树

![此处输入图片的描述][10]

```cpp
void insert(int data)
{
    if (root_ == nullptr)
    {
        root_ = new Node(data);
        return;
    }

    Node * p = root_;
    while (p != nullptr)
    {
        if (data > p.val_)
        {
            if (p.right_ == nullptr)
            {
                p.right_ = new Node(data);
                return;
            }
            p = p.right_;
        }
        else
        {
            // data < p.data
            if (p.left_ == nullptr)
            {
                p.left_ = new Node(data);
                return;
            }
            p = p.left_;
        }
    }
}
```

### 删除操作
第一种情况：要删除的节点没有子节点，则只需要直接将父节点中指向要删除的指针置为NULL

第二种情况：要删除的节点只有一个子节点，则更新父节点，让指向要删除节点的指针指向子节点

第三种情况：要删除的节点有两个子节点，需要找到这个节点的右子树中最小的子节点，把它替换到要删除的节点然后删除这个最小节点

![此处输入图片的描述][11]

 ```cpp
void delete(int data)
{
    Node * p = root_; // p 指向要删除的节点，初始化指向根节点
    Node * pp = nullptr; // pp 记录的是 p 的父节点
    while (p != nullptr && p.val_ != data)
    {
        pp = p;
        if (data > p.data) p = p.right_;
        else p = p.left_;
    }
    if (p == nullptr) return; // 没有找到

    // 要删除的节点有两个子节点
    if (p.left_ != nullptr && p.right_ != nullptr)
    {
        // 查找右子树中最小节点
        Node * minP = p.right_;
        Node * minPP = p; // minPP 表示 minP 的父节点
        while (minP.left_ != nullptr)
        {
            minPP = minP;
            minP = minP.left_;
        }
        p.val_ = minP.val_; // 将 minP 的数据替换到 p 中
        p = minP; // 下面就变成了删除 minP 了
        pp = minPP;
    }

    // 删除节点是叶子节点或者仅有一个子节点
    Node * child; // p 的子节点
    if (p.left_ != nullptr) child = p.left_;
    else if (p.right_ != nullptr) child = p.right_;
    else child = nullptr;

    if (pp == nullptr) tree = child; // 删除的是根节点
    else if (pp.left_ == p) pp.left_ = child;
    else pp.right_ = child;
}
```

### 递归BFS

```cpp
void bfs_rec()
{
    function<void(TreeNode *, int)> bfs = [&](TreeNode * node, int level)
    {
        if(node)
        {
            if(level == 0)
            {
                cout<<node->val<<endl;
                return;
            }
            bfs(node->left, level - 1);
            bfs(node->right, level - 1);
        }
    };
}

```

### 其他操作
支持快速查找最大节点和最小节点、前驱节点和后继节点

中序遍历可以输出有序的数据序列，时间复杂度 O(n)

### 支持重复数据的二叉查找树
实际开发中，二叉查找树存储的是一个包含很多字段的对象，利用对象的某个字段作为键值来构件二叉查找树，其他字段叫做卫星数据

一个节点存储相同的键值：第一种方法通过链表和支持动态扩容的数组等数据结构，把值相同的数据存储在同一个节点上；第二种方法每个节点仍然存储一个数据，在插入过程中遇到值相同的节点，把新插入的数据当做大于这个节点值来处理，查找数据的时候，遇到值相同的节点，并不停止查找操作而是继续在右子树中查找直到叶子节点，对于删除操作，先查找到所有要删除的节点，然后再删除

![此处输入图片的描述][12]

![此处输入图片的描述][13]


## 时间复杂度分析
根节点的左右子树极度不平衡，退化成了链表所以查找的时间复杂度成了 O(n)
完全二叉树不管是插入、删除、还是查找，时间复杂度是 O(height)

## 与哈希表的比较
哈希表的插入、删除、查找操作的时间复杂度可以做到常量级的 O(1)，而二叉查找树在比较平衡的情况下插入、删除、查找的时间复杂度是 O(logn)

1. 哈希表的数据是无序存储的，如果要输出有序的数据需要先进行排序；而二叉查找树只需要中序遍历，时间复杂度O(n)

2. 哈希表扩容耗时很多，遇到哈希冲突性能不稳定；平衡二叉查找树的性能稳定，时间复杂度在O(logn)

3. 因为哈希冲突，哈希表的查找时间不一定比logn小

4. 哈希表的构造比二叉查找树要复杂

5. 为了避免哈希冲突，哈希装载因子不能太大，会浪费一定的存储空间

## 线段树

线段树用来存放给定区间内对应信息的一种数据结构，支持区间查询和元素更新操作，且都是 O(logn)

```cpp
struct SegmentTreeNode {
    int start;  // 区间左闭区间
    int end;  // 区间右闭区间
    int val;  // 节点的值
    SegmentTreeNode * left;
    SegmentTreeNode * right;

    SegmentTreeNode(int s, int e, int v, SegmentTreeNode * l = nullptr, SegmentTreeNode * r = nullptr) : start(s), end(e), val(v), left(l), right(r) {}

    ~SegmentTreeNode() {
        if(left) delete left;
        if(right) delete right;
    }
};

class SegmentTree {
public:
    SegmentTreeNode * root;
    
public:
    SegmentTree(vector<int> & nums) {
        int len = nums.size();
        root = nullptr;
        if(!len) return;
        function<SegmentTreeNode *(int, int)> build = [&](int s, int e)->SegmentTreeNode * {
            if(s == e) return new SegmentTreeNode(s, e, nums[s]);
            int mid = s + (e - s) / 2;
            auto left = build(s, mid);
            auto right = build(mid + 1, e);
            return new SegmentTreeNode(s, e, left->val + right->val, left, right);
        };
        root = build(0, len - 1);
    }
    
    ~SegmentTree() {if(root) delete root;}
    
    void Update(int i, int v, SegmentTreeNode * node) {
        if(node->start == node->end && node->start == i) {
            node->val = v;
            return;
        }
        int mid = node->start + (node->end - node->start) / 2;
        if(mid >= i) Update(i, v, node->left);
        else Update(i, v, node->right);
        node->val = node->left->val + node->right->val;
    }
    
    int QuerySum(int i, int j, SegmentTreeNode * node) {
        if(node->start == i && node->end == j) return node->val;
        int mid = node->start + (node->end - node->start) / 2;
        if(mid >= j) return QuerySum(i, j, node->left);
        else if(mid < i) return QuerySum(i, j, node->right);
        else return QuerySum(i, mid, node->left) + QuerySum(mid + 1, j, node->right);
    }
};
```

对于数组 [2, 5, 1, 4, 9, 3]，节点存储区间内最小值

![8B6i5V.png](https://s1.ax1x.com/2020/03/18/8B6i5V.png)

## 二叉树的序列化与反序列化

```cpp
string serialize(TreeNode* root) {
    if(!root) return {};
    string ans;
    queue<TreeNode *> q;
    q.push(root);
    while(!q.empty()) {
        TreeNode * temp = q.front();
        q.pop();
        if(!temp) {
            ans.push_back('#');
            ans.push_back(',');
            continue;
        } 
        ans += to_string(temp->val);
        ans.push_back(',');
        q.push(temp->left);
        q.push(temp->right);
    }
    return ans;
}

TreeNode* deserialize(string data) {
    const int len = data.length();
    if(!len) return nullptr;
    int e = data.find_first_of(",", 0);
    TreeNode * root = new TreeNode(stoi(data.substr(0, e)));
    queue<TreeNode *> q;
    q.push(root);
    while(e < len - 1) {
        TreeNode * node = q.front();
        q.pop();
        int s = e + 1;
        if(data[s] == '#') {
            node->left = nullptr;
            e = s + 1;
        }
        else {
            e = data.find_first_of(",", s);
            int v = stoi(data.substr(s, e - s));
            node->left = new TreeNode(v);
            q.push(node->left);
        }
        s = e + 1;
        if(data[s] == '#') {
            node->right = nullptr;
            e = s + 1;
        }
        else {
            e = data.find_first_of(",", s);
            int v = stoi(data.substr(s, e - s));
            node->right = new TreeNode(v);
            q.push(node->right);
        }
    }
    return root;
}
```

## 模板

```
// 一棵树
func solve(root)
    if not root: return ...  // 没有左右节点
    if f(root): return ...

    l = solve(root.left)
    r = solve(root.right)
    return g(l, r, root)


// 两棵树
func solve(p, q)
    if not p and not q: return ...  // 两棵树都没有左右节点
    if f(p, q): return ...

    c1 = solve(p.child, q.child)
    c2 = solve(p.child, q.child)
    return g(c1, c2, p, q)
```

  [1]: https://static001.geekbang.org/resource/image/b7/29/b7043bf29a253bb36221eaec62b2e129.jpg
  [2]: https://static001.geekbang.org/resource/image/22/ae/220043e683ea33b9912425ef759556ae.jpg
  [3]: https://static001.geekbang.org/resource/image/22/ae/220043e683ea33b9912425ef759556ae.jpg
  [4]: https://static001.geekbang.org/resource/image/09/2b/09c2972d56eb0cf67e727deda0e9412b.jpg
  [5]: https://static001.geekbang.org/resource/image/12/8e/12cd11b2432ed7c4dfc9a2053cb70b8e.jpg
  [6]: https://static001.geekbang.org/resource/image/14/30/14eaa820cb89a17a7303e8847a412330.jpg
  [7]: https://static001.geekbang.org/resource/image/08/23/08bd43991561ceeb76679fbb77071223.jpg
  [8]: https://static001.geekbang.org/resource/image/f3/ae/f3bb11b6d4a18f95aa19e11f22b99bae.jpg
  [9]: https://static001.geekbang.org/resource/image/96/2a/96b3d86ed9b7c4f399e8357ceed0db2a.jpg
  [10]: https://static001.geekbang.org/resource/image/da/c5/daa9fb557726ee6183c5b80222cfc5c5.jpg
  [11]: https://static001.geekbang.org/resource/image/29/2c/299c615bc2e00dc32225f4d9e3490e2c.jpg
  [12]: https://static001.geekbang.org/resource/image/3f/5f/3f59a40e3d927f567022918d89590a5f.jpg
  [13]: https://static001.geekbang.org/resource/image/fb/ff/fb7b320efd59a05469d6d6fcf0c98eff.jpg