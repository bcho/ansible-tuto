Ansible 简明教程
================

将主机分组
----------

在结点清单里面的主机记录可以任意地进行分组。举个例子：你可以将你的结点划分成 `debian` 组、`web-servers` 组和 `production` 组。

    [debian]
    host0.example.org
    host1.example.org
    host2.example.org

或者用下面更短的写法：

    [debian]
    host[0-2].example.org

假如你想进行嵌套分组，只要将子分组的名字定义为 `[groupname:children]` ，然后往里面添加记录就可以了。
下面这个例子中，假设我们在结点上安装了不同发行版的 linux 系统，那么可以这样在结点清单里分组：

    [ubuntu]
    host0.example.org

    [debian]
    host[1-2].example.org

    [linux:children]
    ubuntu
    debian

当然了，子分组可以继承使用父级分组的配置(Grouping of course, leverages configuration mutualization.)。


设置变量
--------

你可以在几个不同地方里给结点变量进行赋值：结点清单文件、主机变量文件（host vars files），分组变量文件（group vars files），等等。

我通常会把大部分的变量赋值放到 group/host 变量文件中（接下来教程会深入讲解这部分的内容）。
但是我也经常会直接在结点清单文件中指定一些变量，例如指定 `ansible_ssh_host` 来设置主机的 IP 地址。
通常来说 ansible 会在尝试进行 SSH 连接的时候去解析主机的域名。但在你初始化一台主机的时候，它可能还没有设置好 ip 地址 —— 这时用 `ansible_ssh_host` 就可以解决这个问题。

当使用 `ansible-playbook` （不是我们之前使用的 `ansible`）的时候，我们也可以用过设置 `--extra-vars` （或者 `-e`）这个命令行开关来设置变量。
（关于 `ansible-playbook` 的内容将在下一节讲解。）

`ansible_ssh_port` 你也应该能猜到，就是用来指定 SSH 连接时使用的端口：

    [ubuntu]
    host0.example.org ansible_ssh_host=192.168.0.12 ansible_ssh_port=2222

Ansible 会通过搜寻和结点清单文件在同一级的 `group_vars` 和 `host_vars` 文件夹下的文件来添加其他变量值。

Ansible 会根据名字进行搜索。继续用上面的例子： 有关 `host0.example.org` 的变量会在以下这些文件中获得：

- `group_vars/linux`
- `group_vars/ubuntu`
- `host_vars/host0.example.org`

这些文件不是必须的，但只要它们存在，就会被引用。

现在我们已经了解了模块的基本知识、结点清单还有变量。接下来让我们感受下配合了 playbook 的 ansible 的真正力量吧。


接下来，继续[第四节](https://github.com/leucos/ansible-tuto/tree/master/step-04/README_cn.md)的学习。
