## Trie 树（字典树）

### 本质

Trie 树的本质，就是利用字符串之间的公共前缀，将重复的前缀合并在一起

将 how, hi, her, hello, so, see 组织成 Trie 树为：

![](../Picture/DataStruct/trie/01.jpg)

根节点不包含任务信息，每个节点表示一个字符串中的字符，从根节点到红色节点的一条路径表示一个字符串（红色节点不是叶子结点）

构造过程：

![](../Picture/DataStruct/trie/02.jpg)

![](../Picture/DataStruct/trie/03.jpg)

### 实现

1. 借助散列表的思想，通过一个下标与字符的映射数组，来存储子节点的指针

![](../Picture/DataStruct/trie/04.jpg)

```cpp
class TrieNode
{
public:
    char data;
    TrieNode * children[26];
    bool ending;
public:
    TrieNode(char c) : data(c)
    {
        ending = false;
        for(auto & node : children)
        {
            node = nullptr;
        }
    }
};

class Trie
{
private:
    TrieNode root;

public:
    Trie() : root('/') {

    }

    void insert(string text)
    {
        TrieNode * p = root;
        for(const auto c : text) {
            if(!p->children[c - 'a']) p = new TrieNode(c);
            p = p->children[c - 'a'];
        }
        p->ending = true;
    }

    bool find(string text)
    {
        TrieNode * p = root;
        for(const auto c : text) {
            if(!p->children[c - 'a']) return false;
            p = p->children[c - 'a'];
        }
        return p->ending;
    }
};
```

### 复杂度

构建 Trie 树时间复杂度是 O(n)，查询时间复杂度是 O(k)

Trie 树虽然非常高效，但非常消耗内存，为避免重复存储前缀子串，我们维护了一个长度为 26 的节点指针数组，但很多字符都没有使用，而且真实情况下数组不止 26 个

### 缺点

- 字符集不能太大，如果字符集太大那么存储空间就会浪费太多，即使优化也要付出牺牲查询、插入的效率

- 要求字符串的前缀重合比较多，否则空间消耗变大

- 需要从零开始实现一个 Trie 树

- Trie 树种使用了指针，数据块不连续，所以对缓存不友好