Ansible 简明教程
================

# 结点清单（Inventory）

在继续开始学习 ansible 之前，你需要一份结点清单文件。该文件的默认位置在 `/etc/ansible/hosts` 目录下。但你可以给 ansible 另外指定一个路径：可以使用环境变量(`ANSIBLE_HOSTS`)，或者在命令行里使用 `-i` 这个开关来指定一个路径。

当前这个目录下已经包含了一个预先写好的清单文件：

    host0.example.org ansible_ssh_host=192.168.33.10 ansible_ssh_user=root
    host1.example.org ansible_ssh_host=192.168.33.11 ansible_ssh_user=root
    host2.example.org ansible_ssh_host=192.168.33.12 ansible_ssh_user=root

`ansible_ssh_host` 是一个特殊的_变量_ (_variable_)，ansible 会在连接到某个主机时使用这个 ip 地址值。
如果你使用了 vagrant-hostmaster 这个 gem，那么这个变量则不是必须有的。
另外就是假如你修改了你的虚拟机的地址，你需要把这几个值修改成相应的地址。

`ansible_ssh_user` 是另外一个特殊的 _变量_ (_variable_), ansible 会使用这个值来作为 ssh 登录时的用户名。这个变量的默认值是系统当前用户名，或者是 ~/.ansible.cfg 里面指定的 `remote_user` 值（如果存在这个配置文件的话。）


# 测试

现在让我们来测试一下刚刚安装好的 ansible 是否正常：

    ansible -m ping all -i step-01/hosts

在这里，ansible 会尝试在全部结点上执行 `ping` 这个模块（关于模块的知识后续会深入讲解）。

命令输出应该类似于：
    
    host0.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

    host1.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

    host2.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

Good! 现在 3 个结点都在正常运作，ansible 也能连接上去。

接下来我们进入到[下一章](https://github.com/leucos/ansible-tuto/tree/master/step-02/README_cn.md)的学习。
