title: IIS部署Magento2小记
date: 2016-01-21 20:27:58
tags: 
- magento2
- IIS
- Rewrite
- Win必须死
---
# 前言
看这文章的，都是铁了心要这么做的，就不废话了。当然production另当别论。

# Web安装
Magento2的[官方文档](http://devdocs.magento.com/guides/v2.0/install-gde/install/web/install-web.html)看起来比M1组织好多了，虽然我还没记住文档的大纲，还要不停地点导航。

讲道理M2的安装程序做得不错，结果第一次安装，进度到14%就不动了。好在进度条和可以实时滚动的日志一看就知道是AJAX，随手开F12，我去居然是500。

跑去PHP错误日志文件（好东西，不知道的请搜索`php.ini`中的`error_log`和`log_errors`这两项，顺便吐槽一下这命名水平w）一看，居然是执行超时。好吧，赶紧去改了，顺便把输入超时和内存限制也改了。

如果你和我一样，发现不能重新运行安装程序，可以去命令行界面运行`./magento setup:uninstall`（建议用git的bash shell，不然需要用php magento.php的形式）就能干掉残废的安装而且已填写的信息可以保留。

然而还是到17%的时候500，不过这次PHP日志没东西了。还好意识到FCGI还有一个超时设置，同样修改之，如图。仔细看了“活动超时”的意思，果然也要改才行！

{% asset_img iis.png IIS FastCGI设置 %}

这下终于安装通过了。

# 命令行安装

为了避免被FCGI坑，其实完全可以在命令行安装，不过并不是交互式命令行，你需要一次性提供所需的参数= =||

嘛，以后可以在生产环境下这么干。

# webroot应该在哪里

M2的webroot居然可以在`/`也可以在`/pub`，真是32个赞，所以完全可以直接配置web server在`/pub`上，反正安装维护操作都可以用命令行处理。

但是，如果安装时选择了启用重写，并且想当然的把`/.htaccess`下的重写迁移到IIS Rewrite下的话，又杯具了，静态文件都404了。

# 照猫画虎做Rewrite

打开`/pub/static`就觉得已经解决问题了，空空如也except一个`.htaccess`，又无脑迁移……STOP！一不注意就添加到根目录的重写去了。在子目录上的重写规则，IIS和apache仍然是等效的，不然就只有在根目录下的规则里写死了。

不过要注意继承于父目录的重写规则会在我们设置的规则生效前发生，还需要使用上移将它移动到继承规则的上面（从图上看，4个规则都是本地，相当于都覆盖了一遍，蛋疼。。）

{% asset_img rewrite.png IIS static目录下的Rewrite配置 %}

# 说好的Symlink呢

之前在搜索时发现这个[问答](http://magento.stackexchange.com/questions/64802/magento-2-404-error-for-scipts-and-css)，试了下Windows下确实需要切换生成静态文件的方式，即将`app/etc/di.xml`中的

```xml
<item name="view_preprocessed" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Symlink</item>
```

换成：

```xml
<item name="view_preprocessed" xsi:type="object">Magento\Framework\App\View\Asset\MaterializationStrategy\Copy</item>
```

All the green!

想起之前msysgit自带的`ln`也是莫名其妙地不能用，做成`mklink`的包装器不好么……看了这个Symlink类的代码，也是一个委托调用，没有继续去找是用的什么方式。
