# 常见位操作

## 英文字符转小写

```cpp
('a' | ' ') = 'a'
('A' | ' ') = 'a'
```

## 英文字符转大写

```cpp
('b' & '_') = 'B'
('B' & '_') = 'B'
```

## 大小写转换

```cpp
('d' ^ ' ') = 'D'
('D' ^ ' ') = 'd'
```

## 消除数字的二进制最后一个 1

```cpp
int t = n & (n - 1)

/*
输入一个无符号整数，返回二进制中 1 的个数
*/
int main(unsigned int n) {
    int res = 0;
    while(n != 0) {
        n = n & (n - 1);
        ++res;
    }
    return res;
}

/*
判断一个数是不是 2 的指数
*/
bool func(int n) {
    if(n <= 0) return false;
    return (n & (n - 1)) == 0;
}
```