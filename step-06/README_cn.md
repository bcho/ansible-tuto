Ansible 简明教程
================

正常的时候再去重启
------------------

上面两节课之后，我们已经安装了 apache、部署了自家的虚拟主机以及重启了服务器上的 apache 服务。
但假如我们想确保在配置文件没问题的时候才重启服务呢？
接下来你会看到怎么做到这一点。


# 犯错的时候我们就重来

Ansible 有个很赞的特性：它可以在发生的错误的时候停止运行。
我们可以利用这个特性来在配置文件格式不正确的时候终止 playbook 的运行。

下面修改 `awesome-app` 的配置文件来体验一下：

    <VirtualHost *:80>
      RocumentDoot /var/www/awesome-app

      Options -Indexes

      ErrorLog /var/log/apache2/error.log
      TransferLog /var/log/apache2/access.log
    </VirtualHost>

之前提到，假如一个任务执行失败，整个流程都会停下来。所以我们要在重启服务之前确保配置文件格式是正确的。
然后我们还要在添加新的虚拟主机 _之前_ 移除原有的默认配置 —— 这样才不会在一连串的重启中搞坏 apache。

需要注意的就是我们得确保检查配置文件这一步要首先完成。因为我们后续还会去禁用原有的虚拟主机配置：

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

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl

        notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

再次运行 ansible：

    $ ansible-playbook -i step-06/hosts -l host1.example.org step-06/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push future default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    failed: [host1.example.org] => {"changed": true, "cmd": ["apache2ctl", "configtest"], "delta": "0:00:00.045046", "end": "2013-03-08 16:09:32.002063", "rc": 1, "start": "2013-03-08 16:09:31.957017"}
    stderr: Syntax error on line 2 of /etc/apache2/sites-enabled/awesome-app:
    Invalid command 'RocumentDoot', perhaps misspelled or defined by a module not included in the server configuration
    stdout: Action 'configtest' failed.
    The Apache error log may have more information.

    FATAL: all hosts have already failed -- aborting

    PLAY RECAP ********************* 
    host1.example.org              : ok=4    changed=2    unreachable=0    failed=1    

从上面的输出你可以看到 `apache2ctl` 返回了错误代码 1，告诉我们配置文件有错误 ——
这个问题 ansible 也发现了，所以它停止了所有的操作，赞吧？

但实际上这样做还不算十分好。因为不管配置文件是否正确，我们已经添加了新的虚拟主机。
接下来重启 apache 的话就会发生错误。所以我们需要其他方法来捕获这个错误并且把它改回来。

我们会在[第七节](https://github.com/leucos/ansible-tuto/tree/master/step-07/README_cn.md)做到这一点。
