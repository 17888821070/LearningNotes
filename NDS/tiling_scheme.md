# 网格划分

每个 feature 都有自己的唯一的坐标点，可以通过地理位置来查找 feature，定位 feature 和分组 feature

tiling scheme 把整个地球表面按网格切分成一个一个格子，这个格子称为 tile，每个 tile 近似为一个正方形的区域

NDS 中，database 是以 tile 为单位来进行存储的，一个 tile 包含了在这个 tile 边界的地理范围的内的所有 feature，这些 feature 将按 datascript 所定义的格式进行编码，存储为 database 的一个 BLOBs

