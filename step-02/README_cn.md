Ansible 简明教程
================

和结点进行交互
--------------

现在一切顺利，接下来让我们再深入研究一下上一章出现的命令： `ansible` 。
它也是 ansible 中用作和结点进行交互的三条命令中的第一条。

# 做点其他有意思的

在之前那个命令中， `-m ping` 表示的是“使用模块 _ping_”。ping 是 ansible 众多可用模块中的一个。
这个模块十分简单，执行它不需要提供任何参数。假如你要给一个模块提供参数，可以通过 `-a` 这个命令行开关来传递。
接下来让我们试试其他模块。


## Shell 模块

你可以通过这个模块来在远端主机上执行一条 shell 命令：

    ansible -i step-02/hosts -m shell -a 'uname -a' host0.example.org

上面命令的输出应该就是：

    host0.example.org | success | rc=0 >>
    Linux host0.example.org 3.2.0-23-generic-pae #36-Ubuntu SMP Tue Apr 10 22:19:09 UTC 2012 i686 i686 i386 GNU/Linux

超级简单有木有！

## Copy 模块

模块如其名，copy 就是用来让你从当前的控制结点(controlling machine)上复制文件到其他结点上。
假设现在我们要复制 `/etc/motd` 这个文件到目标结点的 `/tmp` 目录下：

    ansible -i step-02/hosts -m copy -a 'src=/etc/motd dest=/tmp/' host0.example.org

输出应该和下面类似：
    
    host0.example.org | success >> {
        "changed": true, 
        "dest": "/tmp/motd", 
        "group": "root", 
        "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
        "mode": "0644", 
        "owner": "root", 
        "size": 0, 
        "src": "/root/.ansible/tmp/ansible-1362910475.9-246937081757218/motd", 
        "state": "file"
    }

Ansible （准确来说在目标结点上运行的 _copy_ 模块）返回了一系列有用的信息（JSON 格式）。
稍后我们会讲到怎么去使用这些信息。

后续我们也会看到其他有用的模块。 Ansible 的文档里有一份详细的[模块清单](http://docs.ansible.com/list_of_all_modules.html)，
它们基本涵盖你可以对系统做的一切操作。假如你没有找到合适你需求的模块，写一个也非常简单（甚至我们连 Python 代码也不用写，写个 JSON  配置文件就好了）。


# 同一个命令，很多个结点

好啦，玩够之后要开始管理我们规模庞大的结点集群了。接下来让我们试试在多台主机上进行操作。

现在假设我们想知道在结点集群里的 Ubuntu 是什么版本，这个工作对 ansible 来说非常简单：

    ansible -i step-02/hosts -m shell -a 'grep DISTRIB_RELEASE /etc/lsb-release' all

上面的 `all` 指的是在所有结点清单里出现的结点上运行，这是 ansible 内置的简写（shortcut）。
命令执行后会返回：

    host1.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

    host2.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

    host0.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

# 更详尽的结点情况

说到要了解结点的状况，这里还有一个非常好（奇）用（葩）的模块： `setup` 。
它的存在就是为了收集结点的 _信息_（_facts_）。

运行一下：

    ansible -i step-02/hosts -m setup host0.example.org

返回了很多有用的信息：

    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.0.60"
        ], 
        "ansible_all_ipv6_addresses": [], 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "01/01/2007", 
        "ansible_bios_version": "Bochs"
        },
        ---snip---
        "ansible_virtualization_role": "guest", 
        "ansible_virtualization_type": "kvm"
    }, 
    "changed": false, 
    "verbose_override": true

为了简洁起见，上面的输出截掉了一部分。但你还是可以从输出里找到许多有意思的信息。
同时你还可以根据自己的需要，过滤返回内容。

例如你现在想要指定所有结点上的内存使用情况，在 ansible 里只需要运行： `ansible -i step-02/hosts -m setup -a 'filter=ansible_memtotal_mb' all` ，同样是超级简单：

    host2.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

    host1.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

    host0.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

注意到这个输出中的结点顺序上一条命令的输出是不一样的 —— 这是因为 ansible 是并行去操作不同结点的。

BTW，在用 setup 这个模块的时候，可以在 `filter=` 这个表达式中使用 `*`：作用和它在 shell 里面一样（shell glob，通配符，识别所有的值）。

# 选择主机来操作

在上面我们学到了 `all` 代表的是“所有主机”，但 ansible 同时也提供了[许多方法来选择主机](http://ansible.cc/docs/patterns.html#selecting-targets)进行操作：

- `host0.example.org:host1.example.org` 将会在 host0.example.org 和 host1.example.org 上执行操作
- `host*.example.org` 将会在所有主机名开头是 'host'，结尾是 '.example.org' 的结点上执行操作（通配符行为和 shell 中一致）

还有其他一些方法牵涉到了结点组（groups），我们会在[下一章](https://github.com/leucos/ansible-tuto/tree/master/step-03/README_cn.md)进行学习。
