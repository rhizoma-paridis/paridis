## install

使用 `scoop` 直接执行命令安装

```shell
scoop install neovim
```

如果不想使用默认 ui，想使用其他 ui。比如 `neovide`，也可以安装。

```shell
scoop install neovide
```

## SpaceVim

以前使用过 lazyVim，有点不喜欢。还是 `spaceVim` 更合适点。参考官方文档。

win 中安装，下载 install.cmd。执行即可。

### config

`spaceVim` 默认 `fileTree` 在右边，有点不习惯，参考官方文档，在自定义配置文件 `~/.SpaceVim.d/init.toml` 中添加配置项。

```toml
[options]
    filetree_direction = "left"
```

## 常用操作

有一些操作可能因各种原因无法正常使用。所以这里只记录简的操作。

### FileTree

#### 目录树

切换目录树：

`<F3>` / `SPC f t`

#### 标签

切换标签：

如果只有一个 Tab, Buffers 将被罗列在标签栏上，每一个包含：序号、文件类型图标、文件名。如果有不止一个 Tab, 那么所有 Tab 将被罗列在标签栏上。标签栏上每一个 Tab 或者 Buffer 可通过快捷键 `<Leader> number` 进行快速访问，默认的 `<Leader>` 是 `\`。

#### 窗口

常用的窗口管理快捷键有一个统一的前缀，默认的前缀 `[Window]` 是按键 `s`，可以在配置文件中通过修改 SpaceVim 选项 `window_leader` 的值来设为其它按键：

切换窗口：

每一个窗口都有一个编号，该编号显示在状态栏的最前端，可通过 `SPC 编号` 进行快速窗口跳转。

关闭窗口：

常规模式下的按键 `q` 被用来快速关闭窗口。

分屏：

`[Window] v`，水平分屏

`[Window] g`，垂直分屏