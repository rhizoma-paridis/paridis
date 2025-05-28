# alias

虽然 omz 有 `custom/aliases.zsh` 可以配置自定义的 alias，但是还是不改它的配置。

在 `.config` 下创建文件 `.aliases`。配置自定义别名。

```shell
alias ls='lsd'
```

在 `~/.zshrc` 中引入文件。

```shell
source ~/.config/.aliases
```