# 排序

## 全字段排序

MySQL 会给每个线程分配一块内存用于排序，称为 sort_buffer，对 sort_buffer 中的数据按照 `order by` 字段做快速排序

按字段排序这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数 sort_buffer_size

sort_buffer_size 就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序

内存放不下时，就需要使用外部排序，外部排序一般使用归并排序算法，将需要排序的数据分成 number_of_tmp_files 份，每一份单独排序后存在这些临时文件中，然后把这 nnumber_of_tmp_files 个有序文件再合并成一个有序的大文件

sort_buffer_size 越小，需要分成的份数越多，number_of_tmp_files 的值就越大

