Ansible 简明教程
================

本教程将会手把手地教你如何使用 ansible。要使用本教程，你需要一台（虚拟出来的或者物理上的）机器来作为 ansible 的结点。
同时本教程提供了一个基于 vagrant 的环境，你可以使用这个环境来运行这个教程里面的示例。

Ansible 是一套配置管理软件，你可以用它来在一台机器上操纵和配置另外一台机器。和其他类似的管理软件不同的是，
ansible 以（可能存在的） SSH 为基础进行操作，而其他同类型的软件（如 chef, puppet 等）需要额外配置一个公钥服务 (PKI infrastructure)。

Ansible 同时强调使用推送的模型 (push mode?)，也就是说配置是从主结点（master machine, 在 ansible 的世界里，主结点仅仅是指一台可以 SSH 到其他结点的机器），而其他配置管理软件（CM）通常都需要使用其他方法来实现配置功能（例如让结点从主结点处拉取配置）。


这种推送模型有意思的地方在于你再也不需要维护一个“外部”可见的主结点，然后再去配置你其他机器 —— 现在是所有结点都需要做到可访问（在后续教程里面我们会介绍其他方法可以让“隐藏”的结点拉取它们需要的配置），这也是它们最基本的要求。


# Ansible 的依赖

在安装 ansible 之前，你需要在你的机器（也就是你的运行 ansbile 的机器，你的主结点）上安装以下几个 Python 模块：

- python-yaml
- python-jinja2

在 Debina/Ubuntu 上你可以运行这条命令：
``sudo apt-get install python-yaml python-jinja2 python-paramiko python-crypto``


同时本教程也默认你已经在 ~/.ssh 文件夹下建立了属于你的 ssh 密钥对(keypair)。


# 安装 Ansible

## 从源码开始安装

Ansible ``devel`` 分支上的代码永远都是可以正常使用的，所以我们直接把官方的仓库 checkout 下来就可以了。
（哦，当然你需要先安装 git，在 Debin/Ubuntu 上你可以运行： `sudo apt-get install git`）

    git clone git://github.com/ansible/ansible.git
    cd ./ansible

现在我们可以加载 ansible 开发环境：

    source ./hacking/env-setup


## 由 deb 包安装

如果你可以使用自带的包管理软件来安装，上面的方法就显得有点多余了。但你也可以把 ansible 的源代码打包成 deb 安装包。
ansible 仓库提供了 `make target` 来进行打包。在打这个 deb 包之前，你需要安装其他几个包：

    sudo apt-get install make fakeroot cdbs python-support
    git clone git://github.com/ansible/ansible.git
    cd ./ansible
    make deb
    sudo dpkg -i ../ansible_1.1_all.deb (version may vary)

在余下的教程里，我们会默认你已经在系统里安装好了 ansible。


# 克隆这个教程到本地

    git clone https://github.com/leucos/ansible-tuto.git
    cd ansible-tuto


# 使用教程的配套 Vagrant 环境

我们强烈建议你使用 Vagrant 来完成教程剩下的内容。假如你还没配置好，可以按照 [step-00/README.md](https://github.com/leucos/ansible-tuto/tree/master/step-00/README_cn.md) 里面的教程来进行配置，整个过程对你来说肯定非常简单啦！


假如你不想使用 Vagrant （我们**不**建议你这么做），可以直接跳到 [setp-01/README.md](https://github.com/leucos/ansible-tuto/tree/master/step-01/README_cn.md) 开始学习。


## 目录

假如你想跳着阅读本教程的话，可以参考下面的目录：

- [00. 配置 Vagrant 环境](https://github.com/leucos/ansible-tuto/tree/master/step-00/README_cn.md)
- [01. 基本结点清单](https://github.com/leucos/ansible-tuto/tree/master/step-01/README_cn.md)
- [02. 第一个模块以及结点情况 (facts)](https://github.com/leucos/ansible-tuto/tree/master/step-02/README_cn.md)
- [03. 分组和变量](https://github.com/leucos/ansible-tuto/tree/master/step-03/README_cn.md)
- [04. Playbooks](https://github.com/leucos/ansible-tuto/tree/master/step-04/README_cn.md)
- [05. 使用 Playbooks 来推送文件到结点](https://github.com/leucos/ansible-tuto/tree/master/step-05/README_cn.md)
- [06. Playbooks 和失败处理](https://github.com/leucos/ansible-tuto/tree/master/step-06/README_cn.md)
- [07. Playbook 和条件选择](https://github.com/leucos/ansible-tuto/tree/master/step-07/README_cn.md)
- [08. Git 模块](https://github.com/leucos/ansible-tuto/tree/master/step-08/README_cn.md)
- [09. 拓展到多台主机的情况](https://github.com/leucos/ansible-tuto/tree/master/step-09/README_cn.md)
- [10. 模板](https://github.com/leucos/ansible-tuto/tree/master/step-10/README_cn.md)
- [11. 再说变量](https://github.com/leucos/ansible-tuto/tree/master/step-11/README_cn.md)
- [12. 技能提升：使用角色组 (roles)](https://github.com/leucos/ansible-tuto/tree/master/step-12/README_cn.md)
- [13. 部署防火墙规则(待写)]()
- [14. 角色依赖能给我们带来什么好处？(待写)]()
- [99. 尾声](https://github.com/leucos/ansible-tuto/tree/master/step-99/README_cn.md)


## Contributing

感谢所有为本教程作出贡献的朋友：

* mxxcon 
* Hartmut Goebel
* Chris Schmitz
* Alexis Gallagher
* Justin Garrison
* Pierre-Gilles Levallois
* Benny Wong
* David Golden
* Patrick Pelletier
* Ruud Kamphuis
* Victor Boivie

虽然我从 ansible 刚开始开发的时候就开始使用，但在写作本教程过程中我学到了很多之前没有遇到的新知识，所以这是一个很好的学习 ansible 的机会！不要犹豫来参与写作吧！

当前正在撰写的章节可以在[writing 分支](https://github.com/leucos/ansible-tuto/tree/writing)上看到。

假如你觉得有什么内容是值得一写的，你可以在这个分支上开写，或者在 [IDEAS.md](https://github.com/leucos/ansible-tuto/blob/master/IDEAS.md) 里面留下你的想法。

同时也欢迎大家来一起撰写各个章节的内容，假如你有兴趣，请告诉我！

如果你想增改章节，麻烦在 `test/expectations` 里面填充相应的内容以及运行测试 (`test/run.sh`)。
在 `test/run.sh` 这个文件里你可以找到更多相关信息。

当你添加一个新章节的时候（假设叫做 `step-NN`），请在 `step-99` 下重新引用文件夹：

    cd step-99
    ln -sf ../step-NN/{hosts,roles,site.yml,group_vars,host_vars} .

Thank you!


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/leucos/ansible-tuto/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

