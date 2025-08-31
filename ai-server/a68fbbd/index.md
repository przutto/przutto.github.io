# 中文输入法配置


首先ubuntu-24.04安装完成后默认可以正常使用iBUS中文输入法，这个比此前的版本体验好，但是使用一段时间后就出现输入问题了，比如输入的字符重复多次，空格没法使待选词上屏，一些界面中没法激活输入法，一些界面使用ESC又没法关闭输入法，简直没法使用了...

<!--more-->

所以考虑更换默认的输入法，首先考虑windows中常用的搜狗输入法，使用官方的教程安好后确实很好用，常用的app中使用非常顺畅，再也没有出现之前的各种奇怪问题了，然而，很快又发现了问题，那就是概率的出现在桌面文件重命名时没法激活输入法，而且概率出现打开文本编辑器中没法激活输入法...这很影响使用的，所以只能继续寻找更好的输入法。

最后经过详细的搜索的调研后，发现大家评价最好的中文输入法就是：fcitx系列，最新的是fcitx5这个版本，更新比较及时，搜狗输入法已经很久没有维护了（貌似最近还是很多年的事了，也难怪出现一些激活失败的问题），所以决定试试这个吧。结果经过一段时间的使用，确实如大家的评价一样，使用感受是最好的一款了，下面就针对fcitx5的安装配置进行详细的说明。

### 安装fcitx5

最好的安装配置指导都应该尽量看官方的文档，所以我们到fcitx5的官网看看：[fcitx5 repo 地址](https://github.com/fcitx/fcitx5)，ubuntu-24.04在官方仓库中适配的是这个版本，虽然不是最新的，但根据这么长时间的ubuntu使用经验来看，这个版本的适配性最好，盲目追求最新版本可能会出现适配问题的，所以我们就是使用apt仓库中的版本。

具体方法是可以直接打开 应用中心， 搜索 fcitx，选择类别 Debian包，从搜索结果中选择 “fcitx5输入法” 和 “中文附加组件” 进行安装，

{{< image src="./images/fcitx5.png" alt="fcitx5输入法" caption="fcitx5输入法" width="60%" >}}

{{< image src="./images/fcitx5中文附加组件.png" alt="fcitx5中文附加组件" caption="fcitx5中文附加组件" width="60%" >}}

安装之后，还需要设置环境变量，在 ~/.profile 中末尾添加如下变量：

```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export SDL_IM_MODULE=fcitx
```

接着，我们来设置自动启动，输入法这个最好是在系统启动后自动启动比较方便：

在 应用中心 里搜索 GNOME Tweaks，然后安装这个Debian包

{{< image src="./images/GNOME Tweaks.png" alt="GNOME Tweaks" caption="GNOME Tweaks" width="60%" >}}

安装之后然后在 系统工具 里找到 优化 打开，就是GNOME Tweaks

{{< image src="./images/开机启动程序.png" alt="开机启动程序" caption="开机启动程序" width="60%" >}}

然后找到 fcitx5，然后点击 添加，这样就将输入法设置为开机启动了。

{{< image src="./images/添加开机启动程序.png" alt="添加开机启动程序" caption="添加开机启动程序" width="60%" >}}

接下来，我们还需要配置下fcitx5让它使用起来更加顺手，我们在 系统工具 中打开 fcitx配置，配置成如下图中的顺序：

{{< image src="./images/fcitx设置.png" alt="fcitx设置" caption="fcitx设置" width="60%" >}}

全局选项：按照截图中来配置，红框的项目需要重点配置，这个极为影响使用体验；完成后基本可以和Windows对齐了。

{{< image src="./images/fcitx快捷键设置.png" alt="fcitx快捷键设置" caption="fcitx快捷键设置" width="60%" >}}

附加组件 也按照截图中进行配置，注意勾选的项目，并且取消了非必要的快捷键。

{{< image src="./images/fcitx附件组件设置.png" alt="fcitx附件组件设置" caption="fcitx附件组件设置" width="60%" >}}

到目前位为止，fcitx5输入法的配置已经完成，使用基本可以将windows中使用习惯丝滑平移过来！

但fcitx5默认的候选框主题有点与系统风格不搭，所以接下来可以继续美化下主题显示：

继续在 应用中心 搜索 扩展管理器，安装后打开，在 浏览 中搜索 Input Method Panel并安装。

{{< image src="./images/input method panel.png" alt="input method panel" caption="input method panel" width="60%" >}}


这个插件安装之后基本是立即生效，现在打开输入法，界面优化了，如果出现显示异常或者没有生效的问题，请注销后重新登陆即可解决。

{{< image src="./images/fcitx5输入法效果.png" alt="fcitx5输入法效果" caption="fcitx5输入法效果" width="60%" >}}

至此，中文输入法的所有配置都已完成，现在终于可以在Ubuntu中畅快的输入中文啦！


---

> 作者: pr_zutto  
> URL: https://przutto.github.io/ai-server/a68fbbd/  

