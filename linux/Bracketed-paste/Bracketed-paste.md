括号粘贴模式

这玩意真的有点恶心。这个模式是为了解决在vim中粘贴避免每行都缩进的问题。但是在终端中并不需要启用啊，不知道为什么终端中也会启用。

导致粘贴到命令行的命令前后会多出一些字符

```Bash
# ^[[200~some test^[[201~
0~some test1~
```

## 禁用

```Bash
printf '\e[?2004l'
```

## 启用

虽然没用，不过也记一下

```Bash
printf "\e[?2004h"
```

## 参考资料

[Bracketed Paste Mode in Terminal - jdhao's digital space](https://jdhao.github.io/2021/02/01/bracketed_paste_mode/)
