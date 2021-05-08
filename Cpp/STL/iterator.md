# 迭代器

|类别|功能|扫描遍数|举例
|-|-|-|-|
输入迭代器|只读不写，只能递增|单遍|`istream::iterator`
输出迭代器|只写不读，只能递增|单遍|`ostream::iterator`
前向迭代器|可读写，只能递增|多遍|`forward_list::iterator`
双向迭代器|可读写，可递增递减|多遍|`map::iterator`
随机访问迭代器|可读写，支持全部迭代器功能|多遍|`vector::iterator`