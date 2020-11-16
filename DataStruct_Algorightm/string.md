# 字符串匹配算法

在字符串 A 中查找字符串 B，那 A 为主串，长度记为 n，B 为模式串，长度记为 m，n > m

## BF 算法

暴力匹配算法（Brute Force）

BF 算法即在主串中，检查起始位置分别是 0、1、2…n-m 且长度为 m 的 n-m+1 个子串，看有没有跟模式串匹配的
![](https://static001.geekbang.org/resource/image/f3/a2/f36fed972a5bdc75331d59c36eb15aa2.jpg)

BF 的最坏情况时间复杂度是 O(n*m)，但却是实际中运用比较常见的字符串匹配算法，原因：
- 实际开发中，模式串和主串的长度都不会太长，而且每次模式串与主串匹配的时候，当中途遇到不能匹配的字符时就停止了
- 代码实现简单，不容易出错



## RK 算法

对 BF 稍加改进，引入哈希算法，时间复杂度就会降低

RK 算法的思路：通过哈希算法对主串中的 n-m+1 个子串分别求哈希值，然后逐个与模式串的哈希值比较大小。如果某个子串的哈希值与模式串相等，那就说明对应的子串和模式串匹配了（先不考虑哈希冲突的问题）。因为哈希值是一个数字，数字之间比较是否相等是非常快速的，所以模式串和子串比较的效率就提高了

通过设计哈希算法来提高计算子串哈希值的效率，比如假设要处理的字符串只包含 26 个小写字母，那就可以只用 26 进制表示一个字符串。为了保证将哈希值落在整形数据范围内，可以牺牲一下允许哈希冲突

当发生哈希冲突，有可能子串和模式串的哈希值相同但本身并不匹配，所以在哈希值相同时再去对比一下子串和模式串本身即可

RK 算法整体的时间复杂度就是 O(n)，但如果存在大量哈希冲突，将导致时间复杂度退化成 O(n*m)


## BM 算法

BM 算法包含两部分，分别是坏字符规则和好后缀规则

### 坏字符串规则

BM 算法中的匹配顺序是按照模式串下标从大到小的顺序倒着匹配的，即我们从模式串的末尾往前倒着匹配

![](https://static001.geekbang.org/resource/image/54/9e/540809418354024206d9989cb6cdd89e.jpg)

当我们发现某个字符没法匹配的时候。我们把这个没有匹配的字符叫作坏字符（主串中的字符）

![](https://static001.geekbang.org/resource/image/22/da/220daef736418df84367215647bca5da.jpg)

拿坏字符 c 在模式串中查找，发现模式串中并不存在这个字符，也就是说，字符 c 与模式串中的任何字符都不可能匹配

这个时候，我们可以将模式串直接往后滑动三位，将模式串滑动到 c 后面的位置，再从模式串的末尾字符开始比较

![](https://static001.geekbang.org/resource/image/4e/64/4e36c4d48d1b6c3b499fb021f03c7f64.jpg)

模式串中最后一个字符 d，还是无法跟主串中的 a 匹配，坏字符 a 在模式串中是存在的，模式串中下标是 0 的位置也是字符 a，可以将模式串往后滑动两位，让两个 a 上下对齐，然后再从模式串的末尾字符开始，重新匹配

![](https://static001.geekbang.org/resource/image/a8/ca/a8d229aa217a67051fbb31b8aeb2edca.jpg)

当发生不匹配的时候，我们把坏字符对应的模式串中的字符下标记作 si。如果坏字符在模式串中存在，我们把这个坏字符在模式串中的下标记作 xi。如果不存在，我们把 xi 记作 -1。那模式串往后移动的位数就等于 si-xi（下标都是字符在模式串的下标）

如果坏字符在模式串里多处出现，那我们在计算 xi 的时候，选择最靠后的那个，因为这样不会让模式串滑动过多，导致本来可能匹配的情况被滑动略过

![](https://static001.geekbang.org/resource/image/8f/2e/8f520fb9d9cec0f6ea641d4181eb432e.jpg)

### 好后缀规则

当模式串滑动到图中的位置的时候，模式串和主串有 2 个字符是匹配的，倒数第 3 个字符发生了不匹配的情况

![](https://static001.geekbang.org/resource/image/d7/8a/d78990dbcb794d1aa2cf4a3c646ae58a.jpg)

把已经匹配的 bc 叫作好后缀，记作 {u}，并拿它在模式串中查找，如果找到了另一个跟 {u} 相匹配的子串 {u*}，那我们就将模式串滑动到子串 {u*} 与主串中 {u} 对齐的位置

![](https://static001.geekbang.org/resource/image/b9/63/b9785be3e91e34bbc23961f67c234b63.jpg)

如果在模式串中找不到另一个等于 {u} 的子串，我们就直接将模式串，滑动到主串中 {u} 的后面，因为之前的任何一次往后滑动，都没有匹配主串中 {u} 的情况

![](https://static001.geekbang.org/resource/image/de/cd/de97c461b9b9dbc42d35768db59908cd.jpg)

但这种滑动太过上头，所以不仅要看好后缀在模式串中，是否有另一个匹配的子串，还要考察好后缀的后缀子串，是否存在跟模式串的前缀子串匹配的

某个字符串 s 的后缀子串，就是最后一个字符跟 s 对齐的子串，比如 abc 的后缀子串就包括 c, bc

前缀子串，就是起始字符跟 s 对齐的子串，比如 abc 的前缀子串有 a，ab

![](https://static001.geekbang.org/resource/image/6c/f9/6caa0f61387fd2b3109fe03d803192f9.jpg)

从好后缀的后缀子串中，找一个最长的并且能跟模式串的前缀子串匹配的

### 规则选择

分别计算好后缀和坏字符往后滑动的位数，然后取两个数中最大的，作为模式串往后滑动的位数

### 实现


#### 坏字符规则

如果拿坏字符在模式串中顺序遍历查找，效率比较低

引入哈希表，将模式串中的每个字符及其下标存进哈希表，这样可以快速找到坏字符在模式串的位置下标

如果字符串的字符集不是很大，每个字符长度是 1 字节，则可以用大小为 256 的数组来记录每个字符在模式串中出现的位置，数组下标对应字符的 ASCII

![](https://static001.geekbang.org/resource/image/bf/02/bf78f8a0506e069fa318f36c42a95e02.jpg)

```cpp
const int SIZE = 256;

void GenerateHashTable(const string & str, int * bc)
{
    // 初始化
    for(int i = 0; i < SIZE; ++i)
    {
        bc[i] = -1;
    }

    // 建立
    for(int i = 0; i < str.size(); ++i)
    {
        bc[(int)str[i]] = str[i];
    }
}

int bm(string haystack, string needle)
{
    int * bc = new int [SIZE];
    GenerateHashTable(needle, bc);
    
    int i = 0;
    while (i <= n - m)
    {    
        int j; 
        for (j = m - 1; j >= 0; --j)  // 模式串从后往前匹配
        {       
            if (a[i+j] != b[j])
            {
                break;  // 坏字符对应模式串中的下标是j
            }    
        }    
        if (j < 0)
        {
            return i; // 匹配成功，返回主串与模式串第一个匹配的字符的位置    
        }
        // 这里等同于将模式串往后滑动j-bc[(int)a[i+j]]位    
        i = i + (j - bc[(int)a[i+j]]);   // 没有考虑到为负的情况
    }
    return -1;
}
```

#### 好后缀规则

- 在模式串中，查找跟好后缀匹配的另一个子串

因为好后缀也是模式串本身的后缀子串，所以可以在模式串和主串正式匹配之前，通过预处理模式串，预先计算好模式串的每个后缀子串，对应的另一个可匹配子串的位置

通过长度可以确定一个唯一的后缀子串

![](https://static001.geekbang.org/resource/image/77/c8/7742f1d02d0940a1ef3760faf4929ec8.jpg)

引入关键的变量 suffix 数组，suffix 数组的下表 k 表示后缀子串的长度，下标对应的数组值存储的是，在模式串中跟好后缀 {u} 相匹配的子串 {u*} 的起始下标值

![](https://static001.geekbang.org/resource/image/99/c2/99a6cfadf2f9a713401ba8feac2484c2.jpg)


- 在好后缀的后缀子串中，查找最长的、能跟模式串前缀子串匹配的后缀子串

suffix 只能处理在模式串中查找跟好后缀匹配的另一个子串，所以需要布尔类型数组 prefix，来记录模式串的后缀子串是否能匹配模式串中的前缀子串

![](https://static001.geekbang.org/resource/image/27/83/279be7d64e6254dac1a32d2f6d1a2383.jpg)


```cpp
void GenerateGS(const string & str, int * suffix, bool * prefix)
{
    // 初始化
    for(int i = 0; i < str.size(); ++i)
    {
        suffix[i] = -1;
        prefix[i] = false;
    }

    // 建立
    for(int i = 0; i < str.size() - 1; ++i)  // str[0, i]
    {
        int j = i;
        int k = 0;

        while(j >= 0 && str[j] == str[m- 1 - k])
        {
            suffix[k] == j;
            --j;
            ++k;
        }
        if(j == -1)
        {
            prefix[k] = true;
        }
    }
}
```

假设好后缀的长度是 k，`suffix[k] != -1`，则后移 j - suffix[k] + 1 位（j 为坏字符在模式串中的下标）

![](https://static001.geekbang.org/resource/image/1d/72/1d046df5cc40bc57d3f92ff7c51afb72.jpg)

如果 `suffix[k] == -1`，表示模式串中不存在另一个跟好后缀匹配的子串片段，好后缀的后缀子串 b[r, m-1]（其中，r 取值从 j+2 到 m-1）的长度 k=m-r，如果 prefix[k] 等于 true，表示长度为 k 的后缀子串，有可匹配的前缀子串，这样我们可以把模式串后移 r 位

如果两条规则都没有找到可以匹配好后缀及其后缀子串的子串，我们就将整个模式串后移 m 位


## KMP

KMP 核心思想跟 BM 思想非常接近，在模式串与主串匹配的过程中，当遇到不可匹配的字符的时候，我们希望找到一些规律，可以将模式串往后多滑动几位，跳过那些肯定不会匹配的情况

在模式串和主串匹配的过程中，把不能匹配的那个字符仍然叫作坏字符，把已经匹配的那段字符串叫作好前缀

![](https://static001.geekbang.org/resource/image/17/be/17ae3d55cf140285d1f34481e173aebe.jpg)

当遇到坏字符的时候，我们就要把模式串往后滑动，在滑动的过程中，只要模式串和好前缀有上下重合，前面几个字符的比较，就相当于拿好前缀的后缀子串，跟模式串的前缀子串在比较

![](https://static001.geekbang.org/resource/image/f4/69/f4ef2c1e6ce5915e1c6460c2e26c9469.jpg)

只需要拿好前缀本身，在它的后缀子串中，查找最长的那个可以跟好前缀的前缀子串匹配的。假设最长的可匹配的那部分前缀子串是{v}，长度是 k。我们把模式串一次性往后滑动 j-k 位，相当于，每次遇到坏字符的时候，我们就把 j 更新为 k，i 不变，然后继续比较

![](https://static001.geekbang.org/resource/image/da/8f/da99c0349f8fac27e193af8d801dbb8f.jpg)

把好前缀的所有后缀子串中，最长的可匹配前缀子串的那个后缀子串，叫作最长可匹配后缀子串；对应的前缀子串，叫作最长可匹配前缀子串

![](https://static001.geekbang.org/resource/image/9e/ad/9e59c0973ffb965abdd3be5eafb492ad.jpg)

求好前缀的最长可匹配前缀和后缀子串，不涉及主串，只需要通过模式串本身就能求解，先预处理计算好，在模式串和主串匹配的过程中，直接拿过来就用

KMP 算法可以提前构建一个数组，用来存储模式串中每个前缀（这些前缀都有可能是好前缀）的最长可匹配前缀子串的结尾字符下标，把这个数组定义为 next 数组叫失效函数

数组的下标是每个前缀结尾字符下标，数组的值是这个前缀的最长可以匹配前缀子串的结尾字符下标

![](https://static001.geekbang.org/resource/image/16/a8/1661d37cb190cb83d713749ff9feaea8.jpg)


我们按照下标从小到大，依次计算 next 数组的值；当我们要计算 next[i] 的时候，前面的 next[0]，next[1]，……，next[i-1] 应该已经计算出来了，利用已经计算出来的 next 值，可以快速推导出 next[i] 的值

假设 b[0, i] 的最长可匹配后缀子串是 b[r, i]，如果把最后一个字符去掉，那 b[r, i-1] 肯定是 b[0, i-1] 的可匹配后缀子串，但不一定是最长可匹配后缀子串

既然 b[0, i-1] 最长可匹配后缀子串对应的模式串的前缀子串的下一个字符并不等于 b[i]，那么就可以考察 b[0, i-1] 的次长可匹配后缀子串 b[x, i-1] 对应的可匹配前缀子串 b[0, i-1-x] 的下一个字符 b[i-x] 是否等于 b[i]，如果等于，那 b[x, i] 就是 b[0, i] 的最长可匹配后缀子串

次长可匹配后缀子串肯定被包含在最长可匹配后缀子串中，而最长可匹配后缀子串又对应最长可匹配前缀子串 b[0, y]；查找 b[0, i-1] 的次长可匹配后缀子串就是查找 b[0, y] 的最长匹配后缀子串的问题

按照这个思路，我们可以考察完所有的 b[0, i-1] 的可匹配后缀子串 b[y, i-1]，直到找到一个可匹配的后缀子串，它对应的前缀子串的下一个字符等于 b[i]，那这个 b[y, i] 就是 b[0, i] 的最长可匹配后缀子串



```cpp
void GenerteNext(string & needle, int * next)
{
    next[0] = -1;
    int k = -1;  // 记录上一个的 next
    for(int i = 1; i < needle.size(); ++i)
    {
        while (k != -1 && needle[k + 1] != b[i]) 
        {
            k = next[k]; 
        } 
        if (needle[k + 1] == needle[i])
        { 
            ++k;
        } 
        next[i] = k;
    }
}

int KMP(string & haystack, string & needle)
{
    int * next = new int[needle.size()];
    GenerteNext(needle, next);

    int j = 0;  // needle 字符串的下标
    for(int i = 0; i < haystack.size() - needle.size() + 1; ++i)
    {
        // 滑动
        while (j > 0 && haystack[i] != needle[j])
        {
             // 一直找到a[i]和b[j] 
             j = next[j - 1] + 1; 
        }

        if(haystack[i] == needle[j])
        {
            ++j;
        }
        if(j == needle.size())
        {
            return i - j + 1;
        }
    }
    return -1;
}

```

空间复杂度是 O(m)，m 表示模式串的长度

计算 next 数组的时间复杂度是 O(m)，查询部分时间复杂度是 O(n)，n 表示主串长度
