Ansible 简明教程
================

Ansible playboooks
------------------

Playbook 的原理很简单：它就是一系列 ansible 命令（“任务”，就是我们在命令行里用 `ansible` 执行的命令）。
这些任务针对不同的 hosts/groups 通过 playbook 组织在一起。

要完成本节任务所需要的文件已经准备好了，你完全不用去敲一遍。

# 以 Apache 为例（或者说：ansible 的 "Hello World!"）

假设我们有如下的结点清单：

    [web]
    host1.example.org

还有就是这些结点上运行的操作系统都是 debian 系的。

提醒：实际上你可以通过 `ansible_ssh_host` 来设定每个主机域名的真是 IP（在本教程里就是这么做的）。
你也可以修改结点清单文件，使用真实的主机域名。
在本教程提供的清单中，我们也会指定 `ansible_ssh_use=root` 来帮助我们简化处理不同的情况。
但请不要在你的生产服务器上面这么做，不然很容易出问题噢！

现在让我们来写一个在 `web` 组主机上安装 apache 的 playbook。

    - hosts: web
      tasks:
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

我们要做的就是把我们要做的东西用合适的 anbile 模块表达出来。
在这个例子中，我们使用了 [apt](http://ansible.cc/docs/modules.html#apt) 这个模块来安装 debian 系的包。
同时在这个例子中我们也在安装包之前更新主机上的包记录缓存。

另外我们也给这个任务指定了一个名字，尽管这不是必须的。但这样做可以给 playbook 提供更多更有用的信息，所以推荐这么做。

总的来说 playbook 其实很简单的啦！

你可以这样运行我们的 playbook （叫它做 `apache.yml` 吧）：


    ansible-playbook -i step-04/hosts -l host1.example.org step-04/apache.yml

在上面的命令中，`setp-04/hosts` 是结点清单文件， `-l` 这个选项限定了只运行在 `host1.example.org` 这台主机上。
而 `apache.yml` 就是我们的 playbook 文件。

当你运行上述命令的时候，你应该会看到如下的输出：
    
    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=2    changed=1    unreachable=0    failed=0    

提醒：假如你安装了 `cowsay` 的话你会看到一只牛（cow）路过。假如你觉得它很烦人，
可以通过指定 `export ANSIBLE_NOCOWS="0"` 这个环境变量来去掉它。

下面让我们来逐行分析上面的输出：

    PLAY [web] ********************* 

这一行表明 ansible 正在 `web` 这一组主机上运行一个 play。一个 play 指的就是一系列关于一个主机的指令。
假如我们在 playbook 文件中通过 `-host: blah` 来指定了另外一个主机，将会显示如下结果（但是在第一个 play 执行结束后才会出现）：

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

还记得我们之前用过的那个 `setup` 模块吗？在执行每个 play 之前，ansible 会在所有需要修改的结点上执行这个模块来收集相关的信息。
假如不需要获取任何关于结点的信息，你也可以在 playbook 中 host 下面的一行添加 `gather_facts: no` （和 `tasks:` 在同一个缩进级别）来禁止这个模块的执行。

    TASK: [Installs apache web server] ********************* 
    changed: [host1.example.org]

接下来就是真正重要的东西上场啦：我们第一个（也是唯一一个）任务运行完毕 —— 输出里面看到一个 `changed`，
从这里我们可以知道这个操作改动了主机 `host1.example.org` 上的东西。

    PLAY RECAP ********************* 
    host1.example.org              : ok=2    changed=1    unreachable=0    failed=0 

最后，ansible 输出了一个报告来总结工作：两个任务成功运行以及改变了结点的状态
（我们的 apache 任务安装了新的包，而 setup 模块没有修改结点状态。）。

现在让我们来再次运行这个命令并且看看会发生什么：

    $ ansible-playbook -i step-04/hosts -l host1.example.org step-04/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=2    changed=0    unreachable=0    failed=0    

现在我们可以看到变更是 '0'，这是 ansible 的核心功能之一：我们的 playbook 只会在需要操作的时候才会操作 —— 这叫做 _幂等性_ (_idempotency_)。这也意味着你可以运行同一个 playbook 任意多次并保证结点主机保持在同一个状态。
（当然除非你使用 `shell` 模块去做一些神奇的操作 —— 这就不在 ansible 的管理范围之内了。）

# 小结

通过这个 playbook 我们在结点主机上安装了 apache 主机，但这还不算是一个完整的功能 —— 它还可以去添加一个虚拟站点（virtualhost）、
确保 apache 服务已经重启，甚至是通过 git 来部署我们的站点。

接下来继续进入到[第五节](https://github.com/leucos/ansible-tuto/tree/master/step-05/README_cn.md)。
