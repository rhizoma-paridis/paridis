# 增加输入方案

在输入法菜单中选择`输入法设定`

![](attachments/Pasted%20image%2020230921163640.png)

点击`获取更多输入方案`在弹出的终端中输入想要的方案名（recipe）。

> 具体recipe 可以参考[东风破](https://github.com/rime/plum#packages)。

![](attachments/Pasted%20image%2020230921163658.png)

## 手动安装

但是一般因为网络问题，成功较少。所以可以直接从[东风破](https://github.com/rime/plum#packages)中访问配置地址。

我们可以手动安装输入法方案，这里举一个复杂的例子。安装五笔。

进入五笔方案的仓库，文档提示反查依赖**`pinyin-simp`** ，如果不用反查可以不装，但这里我们选择安装，那就去**`pinyin-simp`** 仓库。

可以看到两个文档：`pinyin_simp.dict.yaml`，`pinyin_simp.schema.yaml` 。从名称可以看出一个是字库，一个是输入法方案。

下载这两个文件，放入`用户文件夹`中。

在右键菜单中选择`重新部署` 。

然后在右键菜单中选择`输入法设定` 。可以在方案选单中看到`袖珍简化字拼音` 。如果要启用这个输入法勾选即可，但是我们只是用这个输入法做为五笔的辅助，所以安装后就可以不用管了。

安装完`pinyin-simp` ，继续安装五笔，在五笔输入方案仓库可以看到四个文件：`wubi_pinyin.schema.yaml`，`wubi_trad.schema.yaml`，`wubi86.schema.yaml`，`wubi86.dict.yaml`。这里仓库一口气给出了3种五笔输入法。`wubi_pinyin`→五笔拼音，`wubi_trad`→五笔简入繁出，`wubi86`→五笔86。因为我个人比较菜，所以就选择`wubi_pinyin`。这里下载`wubi_pinyin.schema.yaml`，`wubi86.dict.yaml`这两个文件，即一个字库，一个输入法方案。同上面安装`pinyin_simp`一样，放入`用户文件夹`中。然后`重新部署`。再去`输入法设定` 中勾选启用即可。

# 配置文件说明

## 文件位置

![](attachments/Pasted%20image%2020230921163827.png)

**用户文件夹**即用户自定义配置的文件夹。

**程序文件夹**即程序默认配置文件夹。

## 配置文件

如下所有修改的配置都是在用户配置中修改。用户配置相当于是对默认配置的修改和新增。

### 全局配置

默认配置：`default.yaml`

用户配置：`default.custom.yaml`

全局设定配置，设置内容主要有 schema_list, menu 等，具体可以查看文件。

我的配置如下：

```YAML
patch:
  # 我只用五笔，所以配置中只留五笔，其实这个可以不用，因为下面 hotkeys 配置禁用后，这个就没用了。
  schema_list:
    - {schema: wubi_pinyin}
  # 我讨厌输入法的全局快捷键。 f4, ctrl+` 这两个都有用。所以直接设置为 null。就相当于没有快捷键了。
  "switcher/hotkeys":
    - 

```

### 发行版设置

默认配置：`weasel.yaml`

用户配置：`weasel.custom.yaml`

设置中主要有 style 等。

我的配置：

```YAML
 patch:
  "style/color_scheme": brisk
  # 打开软件时切换到英文，注意程序名用全小写
  "app_options/windowsterminal.exe":
    ascii_mode: true

```

## 配置规则

所有 custom 中的配置都放在 `patch` 下。`patch` 规则如下：

```YAML
patch:
  "一级设定项/二级设定项/三级设定项": 新值
  "另一个设定项": 新值
  "再一个设定项": 新值
```

官网写的很多，这里只写常用几条。注意上面的都是 key, value 结构的，注意下面两种写法区别。

```YAML
  # 这样写只会修改 hotkeys 这一项配置
  "switcher/hotkeys":
    - 
  # 这样写会把 switcher 整个配置替换掉，所以不要这样写，除非你就是想整体替换。
  switcher:
    hotkeys:
      - F4
   
```

## 参考文档

https://github.com/rime/home/wiki/CustomizationGuide

https://github.com/rime/home/wiki/RimeWithSchemata