# Tmux 

## 会话与进程

会话的一个重要特点是，窗口与其中启动的进程是连在一起的。打开窗口，会话开始；关闭窗口，会话结束，会话内部的进程也会随之终止，不管有没有运行完

Tmux 就是会话与窗口的解绑工具，将它们彻底分离
- 它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行程序很有用
- 它可以让新窗口"接入"已经存在的会话
- 它允许每个会话有多个连接窗口，因此可以多人实时共享会话
- 它还支持窗口任意的垂直和水平拆分

## 基本用法

### 启动与退出

`tmux` 进入 Tmux 窗口，`exit` 退出

### 前缀键

Tmux 的所有快捷键都要通过前缀键唤起，前缀键 `Ctrl + b`

## 会话管理

### 新建会话

第一个启动 Tmux 窗口的编号是 0，第二个窗口编号是 1，也可以使用 `tmux new -s <session-name>` 为会话起名

### 分离会话

```
Ctrl + b d : tmux detach  # 将会话与窗口分离
```

### 查看会话

```
Ctrl + b s : tmux ls  # 查看所有会话
```

### 接入会话

`tmux attach` 重新接入某个已存在的会话

```
tmux attach -t 0  # 编号
tmux attach -t <session-name>  # 会话名
```

### 杀死会话

```
tmux kill-session -t 0  # 编号
tmux kill-session -t <session-name>  # 会话名
```

 ### 切换会话

```
tmux switch -t 0  # 编号
tmux switch -t <session-name>  # 会话名
```

### 重命名会话

```
Ctrl + b $
tmux rename-session -t 0 <new-name> # 编号
tmux rename-session -t <session-name>   # 会话名
```

## 窗格操作

### 划分窗格

```
# 划分上下两个窗格
ctrl + b " : mux split-window

# 划分左右两个窗格
ctrl + b % : tmux split-window -h
```

### 移动光标

```
Ctrl + b + 方向键 : 移动光标
Ctrl + b + ; : 上一个窗格
Ctrl + b + o : 上一个窗格

# 光标切换到上方窗格
tmux select-pane -U

# 光标切换到下方窗格
tmux select-pane -D

# 光标切换到左边窗格
tmux select-pane -L

# 光标切换到右边窗格
tmux select-pane -R
```

### 交换窗格位置

```
Ctrl + b { : 当前窗格左移。
Ctrl + b } : 当前窗格右移。
Ctrl + b Ctrl + o ：当前窗格上移。
Ctrl + b Alt + o ：当前窗格下移。

# 当前窗格上移
Ctrl + b o: tmux swap-pane -U

# 当前窗格下移
tmux swap-pane -D
```

### 调整窗格大小

```
Ctrl + b Ctrl + 方向键 ：按箭头方向调整窗格大小
```

### 关闭窗格

```
Ctrl + b x
```

## 窗口管理

除了将一个窗口划分成多个窗格，Tmux 也允许新建多个窗口

### 新建窗口

```
Ctrl + b c
tmux new-window

# 新建一个指定名称的窗口
tmux new-window -n <window-name>
```

### 切换窗口

```
Ctrl + b w 显示窗口列表，j 和 k 上下选择
Ctrl + b n 切到下一个窗口
Ctrl + b p 切到上一个窗口
# 切换到指定编号的窗口
tmux select-window -t <window-number>

# 切换到指定名称的窗口
tmux select-window -t <window-name>
```

### 重命名窗口

```
Ctrl + b ,
tmux rename-window <new-name>
```

### 关闭窗口

```
Ctrl + b & 关闭当前窗口
tmux kill-window -t <window-name>
```