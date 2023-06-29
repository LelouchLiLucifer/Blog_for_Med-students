# Welcome\_to\_R

使用R语言的第一步是需要在自己的设备上安装R语言程序，我们可以在以下的网址里找到R语言程序的安装包&#x20;

```
#主页面
https://cloud.r-project.org/
#windows
https://cloud.r-project.org/bin/windows/
#Linux
https://cloud.r-project.org/bin/linux/
#macos
https://cloud.r-project.org/bin/macosx/
```

笔者以最常见的windows系统为例:进入`https://cloud.r-project.org/bin/windows/`后点击`base`然后点击`Download R-4.3.1 for Windows`(注:俺在写这篇文章时R最新的版本号为4.3.1，下载时请以当时R实际的最新版本号为准)即可下载到R语言程序的安装包，之后点击安装即可。\
Linux以及Macos请参考上面给出的网址中的安装教程\
需要说明的是如果是在Linux系统上使用conda运行程序，那么可以使用以下方式进行安装`conda install -c conda-forge r-base`，如果需要指定R的版本号则使用`conda install -c conda-forge r-base=x.x.x`(注:需将x.x.x替换成你所指定的版本号，如4.2 or 4.2.1)
