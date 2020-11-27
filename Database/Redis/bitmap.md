# Bitmap


Bitmap 本身是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型

String 类型是会保存为二进制的字节数组，所以 Redis 就把字节数组的每个 bit 位利用起来，用来表示一个元素的二值状态，可以把 Bitmap 看作是一个 bit 数组