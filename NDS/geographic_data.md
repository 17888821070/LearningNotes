# 数据划分

NDS 使用 WGS84 坐标系统，即经纬度唯一的定位到一个精确的点

NDS 使用 EGM96 来表示高度信息

## 坐标编码

### 经纬度编码

对于经度（X 坐标）从 -180° 到 +180°，完全覆盖 32 位的 `int` 数，也就是覆盖了 -2^31 到 +2^31

对于纬度（Y坐标）从 -90° 到 +90°，覆盖了 -2^30 到 +2^30，只是用到了 31 位

在 NDS 中，`int` 型表示的坐标比地图供应商提供的浮点坐标的表示精确度高的多，所有在 NDS 使用整型编码坐标，运算过程中不会有精度缺失的问题

在把 `float` 的坐标转换成 `int` 的坐标时，我们采用 floor（向下取整） 操作，而不是 truncate（在坐标轴上向 0 方向取），或者 round（四舍五入）

纬度范围为 180°，只用到31bit就可以满足纬度的范围，读取的时候把最高位掩盖掉，比如：值 -272788154，如果用 31bit 来表示，其值为 0x6fae5306，用 32bit 来表示就是 0xefae5306，其实就是最高 2bit 的值必须相同

### Morton Codes

一个经纬度坐标是个 2 维值，包括经度（X）和纬度（Y）

morton code 是个表示 1 维的值，所以需要把 2 维的值映射到 1 维的值

Morton code 为一个 63bit 的 `int` 值

```
x = x[31]x[30]...x[0]
y = y[30]y[29]...y[0]

c = x[31]y[30]x[30]y[29]...y[1]x[1]y[0]x[0]
```

由 x 和 y 的各个 bit 位交错组合而成，c 的值范围为0 <= c <= 2^63

如果按照 morton code 来排序，就是是 morton order