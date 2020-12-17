# Tmux是什么

## 会话&进程

命令行的典型使用方式是，打开一个终端窗口，在里面输入命令。用户与计算机的这种临时的交互，称为一次"会话"。而问题在于，如果关闭了终端窗口，那么会话即结束。

要解决这个问题，就需要将终端与会话进行分离——终端只是显示会话，关闭终端并不会结束会话。

## Tmux的作用

使用Tmux即可实现会话与窗口分离

# 基本用法

指令 | 作用
:---: | :---:
tmux new -s <name> | 创建一个名为name的会话
tmux detach | 当前会话在后台继续执行
tmux ls	| 显示所有session的名字和编号
tmux attach -t <name> | 重新连接name会话
tmux kill-session -t <name> | 杀死某个会话
tmux switch -t <name> | 切换会话
tmux split-window | 上下分割窗口
tmux split-window -h | 水平(左右)分割窗口
tmux select-pane -U/D/L/R | 选择上/下/左/右的窗口
tmux swap-pane -U/D | 移动上/下的窗口


# 快捷键
快捷键 | 对应作用
:---: | :---:
ctrl+d | 退出窗口并杀死会话
ctrl+b d | tmux detach
ctrl+b ? | 帮助文档
ctrl+b s | 查看所有会话(如果有多个窗口，可以使用方向键`->`查看子窗口)
ctrl+b $ | 重命名会话
ctrl+b % | 划分左右两个窗口
ctrl+b " | 划分上下两个窗口
ctrl+b x | kill窗口
ctrl+b <方向键> | 切换到方向键所在的窗口
ctrl+b ctrl+<方向键> | 按方向键方向调整窗口大小
ctrl+b ! | 将当前窗格拆分成独立的
ctrl+b z | 全屏/缩小
