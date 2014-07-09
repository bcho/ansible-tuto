Ansible 简明教程
================

继续配置 apache
---------------

上一节中，我们已经安装了 apache，现在让我们来配置 virtualhost。

# 完善 playbook

我们要做的只是在服务器上安装一个虚拟主机，但我们想将默认的配置换成另外一个指定的配置。

所以我们需要移除当前服务器上的虚拟主机配置（在这里叫它做 `默认` 的配置），然后上传我们的虚拟主机，激活它，最后重启 apache 服务。

接下来我们会建立一个叫 `files` 的文件夹，然后在里面添加 host1.example.org 上的虚拟主机配置（就叫它做 `awesome-app` 吧）：

    <VirtualHost *:80>
      DocumentRoot /var/www/awesome-app

      Options -Indexes

      ErrorLog /var/log/apache2/error.log
      TransferLog /var/log/apache2/access.log
    </VirtualHost>

接下来，修改之前的 apache playbook 配置：
    
    - hosts: web
      tasks:
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Push default virtual host configuration
          copy: src=files/awesome-app dest=/etc/apache2/sites-available/ mode=0640 

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl

        - name: Activates our virtualhost
          command: a2ensite awesome-app
          notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

运行新的配置文件：

    $ ansible-playbook -i step-05/hosts -l host1.example.org step-05/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default ssl virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    NOTIFIED: [restart apache] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=7    changed=5    unreachable=0    failed=0    

看起来不错，但仔细想想，我们是不是漏了点什么步骤没做？难道我们不应该在重启 apapche 服务之前检查配置是否正常吗？
这样做我们才不会在写错配置文件的时候把服务搞砸噢。

让我们在[下一节](https://github.com/leucos/ansible-tuto/tree/master/step-06/README_cn.md)学习怎么做到这种效果吧。
