# 单调栈


单调栈即满足单调性的栈结构

单调栈分为单调递增栈和单调递减栈，就是栈内元素是升序还是降序排列的，这也涉及到出栈的逻辑

单调递增栈表示入栈元素时递增的

单调递减栈表示入栈元素时递减的


- 可以在 O(N) 的时间复杂度，找出每个数左右两边第一个大于或小于它的解

- 单调递增栈用于查找两边第一个小于当前元素的值，单调递减栈用于查找两边第一个大于当前元素的值

- 一般数组中的单调性问题，题目中隐含第一个或离此元素最近的大于或小于元素的值，这类问题都可以考虑下，用单调栈是否可以求解

```cpp
// 单调递增栈
stack<int> inc_stack;

void putincstack(int v) {
    while(!inc_stack.empty() && inc_stack.top() >= v) {
        inc_stack.pop();
    }
    inc_stack.push(v);
}

// 单调递减栈
stack<int> des_stack;

void putdesstack(int v) {
    while(!des_stack.empty() && des_stack.top() <= v) {
        des_stack.pop();
    }
    des_stack.push(v);
}
```