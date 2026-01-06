**智能分页显示**：如果内容不满一屏，直接显示；如果超过一屏，再进入跳转界面。

**在终端执行：**
```bash
git config --global core.pager "less -F -X"
```
- **参数解释：**
    
    - `-F` (Quit if one screen)：如果内容不满一屏，自动退出跳转模式，直接显示在屏幕上。
        
    - `-X` (No init)：防止 `less` 在退出时清空屏幕内容。
        
- **效果：** 以后运行 `git branch -v` 时，如果分支不多，它会直接显示在终端；如果分支非常多，它才会跳转让你翻页。

**验证是否成功：** 你可以运行以下命令查看配置是否已经写入：
```bash
git config --global core.pager
```
如果返回 `less -F -X`，说明已经生效。


**检查 `.gitconfig` 文件（可选）：** 你也可以直接查看 Git 的配置文件内容：
```bash
cat ~/.gitconfig
```

你会发现里面多了一行：

```ini, TOML
[core]
    pager = less -F -X
```

