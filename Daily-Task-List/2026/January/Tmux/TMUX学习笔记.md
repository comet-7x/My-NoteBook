$tmux$ 的核心操作围绕「会话（Session）」「窗口（Window）」「窗格（Pane）」三层结构展开

## 一、会话管理（最核心，对应你之前问的场景）
会话是 $tmux$ 的最外层容器，一个会话对应一套独立的终端环境。

|功能|完整命令（带全称参数）|缩写命令（带缩写参数）|使用示例|说明|
|---|---|---|---|---|
|新建会话|`tmux new --session-name 会话名`|`tmux new -s 会话名`|`tmux new -s dev`（创建名为 dev 的会话）|省略 - s 则生成默认名称（数字）|
|列出所有会话|`tmux list-sessions`|`tmux ls`|`tmux ls`（查看所有会话）|最常用的会话查看命令|
|连接 / 附加会话|`tmux attach --target-session 会话名/编号`|`tmux attach -t 会话名/编号`|`tmux attach -t dev`（连接 dev 会话）|恢复后台会话的核心命令|
|杀死 / 关闭会话|`tmux kill-session --target-session 会话名/编号`|`tmux kill-session -t 会话名`|`tmux kill-session -t dev`（关闭 dev 会话）|仅关闭指定会话|
|切换会话|`tmux switch --target-session 会话名/编号`|`tmux switch -t test`|`tmux switch -t test`（切到 test 会话）|在已有 tmux 中切换会话|
|重命名会话|`tmux rename-session --target-session 旧名 新名`|`tmux rename-session -t 旧名 新名`|`tmux rename-session -t dev dev-2`|给会话改名|
|脱离当前会话（不关闭）|无命令行形式，仅快捷键|`Ctrl+b d`（先按 Ctrl+b，再按 d）|-|会话保留在后台，回到原终端|
|杀死所有会话|`tmux kill-server`|无缩写|`tmux kill-server`|强制关闭 tmux 所有进程|

### 二、窗口管理（一个会话内的多个独立窗口）

一个会话可以包含多个窗口（类似浏览器的标签页），窗口编号从 0 开始。

|功能|完整命令（带全称参数）|缩写命令（带缩写参数）|使用示例|说明|
|---|---|---|---|---|
|新建窗口|`tmux new-window --window-name 窗口名`|`tmux neww -n editor`|`tmux neww -n editor`（新建名为 editor 的窗口）|省略 - n 则默认名称为命令名|
|列出当前会话的窗口|`tmux list-windows --target-session 会话名`|`tmux lsw -t dev`|`tmux lsw -t dev`（查看 dev 会话的所有窗口）|省略 - t 则默认当前会话|
|切换窗口|`tmux select-window --target-window 编号/名称`|`tmux selectw -t 1`|`tmux selectw -t editor`（切到 editor 窗口）|快捷键：`Ctrl+b 窗口编号`（如 Ctrl+b 1）|
|重命名窗口|`tmux rename-window --target-window 旧名 新名`|`tmux rename-window -t 0 db`|`tmux rename-window -t 0 db`（把 0 号窗口改名为 db）|-|
|关闭窗口|`tmux kill-window --target-window 编号/名称`|`tmux killw -t 1`|`tmux killw -t editor`（关闭 editor 窗口）|快捷键：`Ctrl+b &`（确认后关闭）|

### 三、窗格管理（一个窗口内的分屏，最常用的细节操作）

一个窗口可以分割成多个窗格（分屏），是 $tmux$ 提升效率的核心功能。

|功能|完整命令（带全称参数）|缩写命令（带缩写参数）|使用示例|说明|
|---|---|---|---|---|
|垂直分窗格（左右）|`tmux split-window --vertical`|`tmux splitw -v`|`tmux splitw -v`（当前窗格垂直拆分）|快捷键：`Ctrl+b %`|
|水平分窗格（上下）|`tmux split-window --horizontal`|`tmux splitw -h`|`tmux splitw -h`（当前窗格水平拆分）|快捷键：`Ctrl+b "`|
|切换窗格|`tmux select-pane --up/down/left/right`|`tmux selectp -U/-D/-L/-R`|`tmux selectp -D`（切换到下方窗格）|快捷键：`Ctrl+b 方向键`|
|关闭当前窗格|`tmux kill-pane --target-pane 窗格编号`|`tmux killp -t 1`|`tmux killp`（关闭当前窗格）|快捷键：`Ctrl+b x`（确认后关闭）|
|调整窗格大小|`tmux resize-pane --up/down/left/right 数字`|`tmux resizep -U 5`|`tmux resizep -R 5`（右侧扩展 5 个字符）|快捷键：`Ctrl+b 按住方向键`（逐行调整）|

### 四、其他常用辅助命令

|功能|完整命令|缩写命令|使用示例|说明|
|---|---|---|---|---|
|查看 tmux 帮助|`tmux help`|`tmux -h`|`tmux help split-window`|查看指定命令的详细说明|
|查看当前会话信息|`tmux display-message -p '#S'`|无|`tmux display-message -p '#S'`|打印当前会话名（#S = 会话名，#W = 窗口名）|

### 关键补充：tmux 的「前缀键」

所有 $tmux$ 快捷键都需要先按**前缀键**（默认是 `Ctrl+b`），松开后再按对应按键，比如：

|快捷键|功能描述|
|---|---|
|`Ctrl + b` + `c`|新建窗口|
|`Ctrl + b` + `w`|列出所有窗口 / 会话（可切换）|
|`Ctrl + b` + `n`|切换到下一个窗口|
|`Ctrl + b` + `p`|切换到上一个窗口|
|`Ctrl + b` + `%`|垂直分割当前面板（左右分屏）|
|`Ctrl + b` + `"`|水平分割当前面板（上下分屏）|
|`Ctrl + b` + 方向键|在分割的面板间切换|
|`Ctrl + b` + `d`|分离当前会话（回到本地终端）|
|`Ctrl + b` + `[`|进入滚动模式（可翻页查看历史输出，按 `q` 退出）|

你可以修改前缀键（比如改成 `Ctrl+a`），但新手先熟悉默认的即可。

### 总结

1. **核心参数**：`-s`（--session-name）用于命名新会话，`-t`（--target-*）是通用参数，指定要操作的「目标」（会话 / 窗口 / 窗格），几乎所有 tmux 命令都依赖这两个参数。
2. **操作优先级**：日常使用中，**快捷键**（如 Ctrl+b %/Ctrl+b d）比命令行更高效，命令行主要用于会话的创建 / 连接 / 删除（如 tmux new -s dev、tmux attach -t dev）。
3. **三层结构记忆**：会话（全局）→ 窗口（会话内标签）→ 窗格（窗口内分屏），按这个层级理解命令，更容易记住。