# docker基础知识介绍

什么是docker，引用官方的一句话

> Build, Ship and Run Any Application, Anywhere

docker就是一个这样的工具。它可以帮助开发者很方便的去构建，部署，运行自己的程序。它可以让你非常迅速的测试和部署你的项目到生产环境中。

对于docker的具体实现和原理我们不多讲，让我们直接来做一个简单的例子来体验一下docker的魅力。

首先你需要在你自己的机器上安装docker，详细的安装文档请参考 [Docker 官方文档](https://docs.docker.com/installation)(FIXME: hard copy book isn't a browser, URL is meanningless)。

这里以在 Ubuntu 14.04 系统上安装 Docker 为例。

    curl -sSL https://get.docker.com | sh

一段美妙的小脚本就被安装到了你的机器上，他完成了你安装docker需要的所有内容。下面我们就开始使用它吧。

如果我们以一个简单的小应用来演示肯定激发不了你的兴趣，那么我们以安装一个wordpress为例，看看docker是如何快速安装一个wordpress 的。

以前安装wordpress,你可能需要去了解PHP，mysql，然后还有你的服务器的系统，最后才是去安装wordpress。非常的麻烦，但是如果我们换一种方式，使用docker来安装呢。

    docker run -d -p 80:80 --name wordpress index.alauda.cn/alauda/wordpress

运行以上命令，docker就会自动从灵雀云平台拉取wordpress镜像，这个镜像是已经被build好的，包含了PHP，mysql和wordpress，你所做的工作就是等待docker帮你启动起来以后，在浏览器上访问你服务器的IP就可以看到wordpress的安装页面，然后一步步的点击页面安装即可。对于你的mysql密码

    echo $(docker logs wordpress | grep password)
这个命令就可以获得mysql密码，填写到网页中，这样你就得到了一个可以运行的wordpress，然后开始愉快的使用他吧。

是不是感受到了docker的威力。其实这只是docker强大功能的冰山一角。快速部署是docker其中一个特性。你不需要去登录到服务器，将运行环境一个一个的安装好，最后再部署你自己的代码。docker像集装箱一样，帮助你打包好了一切，你只需要开箱使用即可。就像我们刚才的例子，我们还可以非常简单的再次运行刚才的命令，只需要换一下映射的端口，就可以再启动一个wordpress，这是安装原生应用所不敢想象的。

docker由client，daemon，registry组成。下面图就列出了docker的基本结构。
![docker架构图](https://docs.docker.com/article-img/architecture.svg)（FIXME：local image）

更加细节的docker介绍和讲解请参考docker文档。



