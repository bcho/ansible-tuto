Ansible 简明教程
================

为了确保教程的内容可以在你的机器上运行，本教程提供了一份 Vagrant 配置文件。
通过 Vagrant 我们可以方便地创建出一系列只有基本配置的虚拟环境。


# 安装 Vagrant

要运行 Vagrant，需要：

- 安装了 VirtualBox
- 安装了 Ruby （你的系统应该已经安装好了）
- 安装了 Vagrant 1.1 以上的版本（参考：http://docs.vagrantup.com/v2/installation/index.html）

以上就是配置 Vagrant 所需要的东西。


接下来我们用以下的命令来初始化虚拟机环境。需要注意的是在这里你不需要去手动下载任何镜像（box）。
因为这一步已经写到 `Vagrantfile` 里面了，vagrant 会帮你打点好这一切。


`vagrant up`


运行之后我们就可以去喝杯茶休息下等待初始化完成啦。（假如你使用的是 vagrant-hostmaster，你需要输入你的 root 密码。）


在这个过程中如果遇到什么问题，可以参考 Vagrant 的[入门文档](http://docs.vagrantup.com/v2/getting-started/index.html)来解决。


# 添加你的 SSH 密钥到虚拟机里

为了完成教程剩下的内容，你需要把你的 SSH 密钥添加到虚拟机 root 用户的 `authorized_keys` 文件中。
虽然这不是必须的（Ansible 还可以使用 sudo、密码认证等方法），但这么做是最方便的。

Ansible 本身就适合做这种活 —— 但现在我不会解释接下来这条命令的意思，你相信我 ansible 会帮你搞定就行！

    ansible-playbook -c paramiko -i step-00/hosts step-00/setup.yml --ask-pass --sudo

假如命令执行过程中需要你输入密码，请输入 _vagrant_ 。假如你遇到了 "Connections refused" 这个错误，请检查你机器的防火墙配置。

为了让整个过程更加通畅无阻，建议让 ssh-agent 在后台运行，并且把你的密钥添加到里面去（通过 `ssh-add` 命令）。

接下来让我们进入[第一章](https://github.com/leucos/ansible-tuto/tree/master/step-01/README_cn.md)的学习。
