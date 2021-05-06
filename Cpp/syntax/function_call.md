# 函数调用

## 函数调用过程

栈帧专门用于保存函数调用过程中的各种信息，如参数、返回地址本地变量等

栈顶的地址最低，栈底的地址最高，栈指针 esp 指向栈顶的，基址指针 ebp 指向栈底，在过程调用中不变

程序执行时移动，ESP 减小分配空间，ESP 增大释放空间

![](../../Picture/Cpp/syntax/functioncall/01.png)

栈帧从栈顶到栈底如下构成：

- 参数列表为要调用函数建立的参数

- 局部变量

- 保存的寄存器的上下文

- 旧的帧指针，即前一栈帧的 ebp 指针，指向前一帧的栈底

调用者函数栈帧：

- 参数列表

- 本地局部变量

- 返回地址，当被调用者函数内部执行到 ret 指令后，从栈中弹出返回地址，以便调用者函数回到该地址对应的指令继续执行余下的指令

```cpp
int func(int param1 ,int param2, int param3) {
     int var1 = param1;
     int var2 = param2;
     int var3 = param3;
     printf("var1=%d,var2=%d,var3=%d",var1,var2,var3);
     return var1;
}
 
int main(int argc, char* argv[]) {
     int result = func(1,2,3);
     return 0; 
}
```
函数 main 执行，main 各个参数从右向左逐步压入栈中，最后压入返回地址