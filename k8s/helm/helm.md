#helm

## 部署服务

### helm upgrade

部署一个 release。加 `-i` 参数后可以替代 `install` 。

```shell
helm upgrade -f [values.yaml] [release-name] [chart-name]
```

如官网上的一个例子。

```shell
helm upgrade -i -f ./values.yaml -f ./override.yaml --description="init" redis ./redis
# chart 可以写路径，也可以写仓库中的 chart name。
helm upgrade -f ./values.yaml -f ./override.yaml redis redis
```

### helm uninstall

卸载部署的 release。比较简单没什么好说的。

```shell
helm uninstall redis -n redis
```

### helm rollback

rollback 要跟 history 配合使用，不知道 history 就无法使用 rollback

```shell
helm history RELEASE_NAME [flags]
```

```
$ helm history angry-bird
REVISION    UPDATED                     STATUS          CHART             APP VERSION     DESCRIPTION
1           Mon Oct 3 10:15:13 2016     superseded      alpine-0.1.0      1.0             Initial install
2           Mon Oct 3 10:15:13 2016     superseded      alpine-0.1.0      1.0             Upgraded successfully
3           Mon Oct 3 10:15:13 2016     superseded      alpine-0.1.0      1.0             Rolled back to 2
4           Mon Oct 3 10:15:13 2016     deployed        alpine-0.1.0      1.0             Upgraded successfully
```

根据查到的 history 。回滚到合适的历史版本。

```shell
helm rollback <RELEASE> [REVISION] [flags]
```

示例如下。

```shell
helm rollback reids 2 -n reids
```

## 开发 chart

### helm create

```shell
helm create NAME [flags]
```

For example, 'helm create foo' will create a directory structure that looks something like this:

```
foo/
├── .helmignore   # Contains patterns to ignore when packaging Helm charts.
├── Chart.yaml    # Information about your chart
├── values.yaml   # The default values for your templates
├── charts/       # Charts that this chart depends on
└── templates/    # The template files
    └── tests/    # The test files
```

### helm template

渲染 chart 。查看结果，主要是用来在部署前查看渲染结果，当然开发的时候也要多看看。

```shell
helm template [RELEASE NAME] [CHART NAME] [flags]
```

示例如下。

```shell
helm template -f values.yaml redis redis --output-dir='./test'
```

### helm package

把开发完的项目打包。

```shell
helm package [CHART PATH] [flags]
```

示例如下。

```shell
helm package ./tounger
```

会在当前目录下生成 tounger-0.0.1.tgz。版本号依赖 chart.yaml 中定义的版本。

打包完成后一般要把这个文件上传到仓库。可以使用相关插件push到仓库。也可以手动上传。因为这玩意并不经常改。