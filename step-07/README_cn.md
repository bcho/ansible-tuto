Ansible 简明教程
================

使用分支条件
------------

现在我们已经完成了安装 apache、部署虚拟主机和重启 apache 服务的任务。但我们还想更进一步，
做到在发生错误的时候将结点恢复到正常的状态。


# 当发生错误的时候重新来过

首先需要说明的就是：ansible 不能保护你的主机 —— 上一节出现的错误不是由 ansible 引起的。
而 ansible 本身也不是一个备份系统，它不可以达到重置所有东西的效果。所以你必须确保你的 playbook 执行内容的安全性。
Ansible 本身是不知道如何去重置因为 `a2ensite awesome-app` 而产生的修改的。

但假如我们想做，其实也是可以做到的。

之前说到，一个任务失败之后，将会停止执行。但只要我们去主动处理这个错误（我们[应该](http://www.aaronsw.com/weblog/geremiah)这么做），就可以继续执行了。这也是我们接下来要 演示的：在发生错误的时候执行重写操作，去掉我们所做的变更。


    - hosts: web
      tasks:
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Push future default virtual host configuration
          copy: src=files/awesome-app dest=/etc/apache2/sites-available/ mode=0640

        - name: Activates our virtualhost
          command: a2ensite awesome-app

        - name: Check that our config is valid
          command: apache2ctl configtest
          register: result
          ignore_errors: True

        - name: Rolling back - Restoring old default virtualhost
          command: a2ensite default
          when: result|failed

        - name: Rolling back - Removing our virtualhost
          command: a2dissite awesome-app
          when: result|failed

        - name: Rolling back - Ending playbook
          fail: msg="Configuration file is not valid. Please check that before re-running the playbook."
          when: result|failed

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl

        notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

关键字 `register` 会记录命令 `apache2ctl configtest` 的运行结果（退出状态，stdout，stderr 等等）。
而 `when: result|failed` 则会去检查注册变量（`result`）是否包含错误的状态。

运行结果：

    $ ansible-playbook -i step-07/hosts -l host1.example.org step-07/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push future default virtual host configuration] ********************* 
    ok: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    failed: [host1.example.org] => {"changed": true, "cmd": ["apache2ctl", "configtest"], "delta": "0:00:00.051874", "end": "2013-03-10 10:50:17.714105", "rc": 1, "start": "2013-03-10 10:50:17.662231"}
    stderr: Syntax error on line 2 of /etc/apache2/sites-enabled/awesome-app:
    Invalid command 'RocumentDoot', perhaps misspelled or defined by a module not included in the server configuration
    stdout: Action 'configtest' failed.
    The Apache error log may have more information.
    ...ignoring

    TASK: [Rolling back - Restoring old default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Removing our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Ending playbook] ********************* 
    failed: [host1.example.org] => {"failed": true}
    msg: Configuration file is not valid. Please check that before re-running the playbook.

    FATAL: all hosts have already failed -- aborting

    PLAY RECAP ********************* 
    host1.example.org              : ok=7    changed=4    unreachable=0    failed=1    

看起来和我们预期的结果一样，接下来让我们去尝试重启一下 apache 服务，看看是不是正常：

    $ ansible -i step-07/hosts -m service -a 'name=apache2 state=restarted' host1.example.org
    host1.example.org | success >> {
        "changed": true, 
        "name": "apache2", 
        "state": "started"
    }

好的，apache 服务恢复正常，没有包含错误。

尽管这看起来很麻烦，但实际上不是的。记住你几乎可以在任何地方使用变量，因此很容易把这个写成针对 apache 的模板，
并用它来在任何结点上部署你的虚拟站点：只写一次，到处使用。我们将在第 9 节学到怎么做到这一点，
但现在先让我们[使用 git ](https://github.com/leucos/ansible-tuto/tree/master/step-08/README_cn.md)来部署我们的站点。
