# install

> omz 默认装在 HOME 目录，XDG 的标准好管理些，所以改到 `.confg` 目录。

```bash
export ZSH="$HOME/.config/.oh-my-zsh"
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

HOME 目录会自动生成一些缓存。

# theme

上面改了 omz 安装路径，所以这里也改下。

```shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.config/.oh-my-zsh/custom}/themes/powerlevel10k"
```

在 `~/.zshrc` 设置 `ZSH_THEME="powerlevel10k/powerlevel10k"`。接下来，重启终端会自动引导你配置 `powerlevel10k` 或执行 `p10k configure` 开始配置。

# 插件

## **[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)**

1. 下载插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.config/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

2. 配置

```shell
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```

## [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

1. 安装

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.config/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

2. 配置

```shell
plugins=( [plugins...] zsh-syntax-highlighting)
```
