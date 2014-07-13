Ansible 简明教程
================

使用 git 来部署我们的站点
-------------------------

现在我们已经安装了 apache、部署了虚拟站点以及保证结点服务的正常重启和运行。
接下来我们会使用 git 模块来部署应用。

# Git 模块

换一个角度来说，本节没有什么新东西要学。`git` 模块本身就和其他模块一样。
但我们还是会来熟悉它，方便我们后面学习 `ansible-pull` 这个命令。

虚拟站点已经准备好，但现在需要进行部署一些新的修改。首先要明确的就是我们的站点是
用 PHP 写的，所以在结点主机上安装 `libapache2-mod-php5` 这个包。第二，我们要安装 `git` 命令
来使用 ansible 的 git 模块（用作克隆应用仓库）。

根据之前学到的只是，我们可以这样做：
        
        ...
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Installs php5 module
          apt: pkg=libapache2-mod-php5 state=installed

        - name: Installs git
          apt: pkg=git state=installed
        ...

但是 ansible 里面提供了一种更加可读的写法。在 ansible 里面可以使用循环来遍历一系列的内容，
然后对它们进行一些操作：


    - hosts: web
      tasks:
        - name: Updates apt cache
          apt: update_cache=true

        - name: Installs necessary packages
          apt: pkg={{ item }} state=latest 
          with_items:
            - apache2
            - libapache2-mod-php5
            - git

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

        - name: Rolling back - Removing out virtualhost
          command: a2dissite awesome-app
          when: result|failed

        - name: Rolling back - Ending playbook
          fail: msg="Configuration file is not valid. Please check that before re-running the playbook."
          when: result|failed

        - name: Deploy our awesome application
          git: repo=https://github.com/leucos/ansible-tuto-demosite.git dest=/var/www/awesome-app
          tags: deploy

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl
          notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

运行我们新的 playbook：

    $ ansible-playbook -i step-08/hosts -l host1.example.org step-08/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Updates apt cache] ********************* 
    ok: [host1.example.org]

    TASK: [Installs necessary packages] ********************* 
    changed: [host1.example.org] => (item=apache2,libapache2-mod-php5,git)

    TASK: [Push future default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Restoring old default virtualhost] ********************* 
    skipping: [host1.example.org]

    TASK: [Rolling back - Removing out virtualhost] ********************* 
    skipping: [host1.example.org]

    TASK: [Rolling back - Ending playbook] ********************* 
    skipping: [host1.example.org]

    TASK: [Deploy our awesome application] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default ssl virtualhost] ********************* 
    changed: [host1.example.org]

    NOTIFIED: [restart apache] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=10   changed=8    unreachable=0    failed=0    

接下来打开你的浏览器，浏览 http://192.168.33.11 ，你应该会在页面上看到一只猫咪和服务器的主机名。

在 playbook 中的 `tags:deploy` 这行给一个任务指定了标签。通过标签我们可以指定运行 playbook 的某一部分。
假设现在我们需要为站点部署一个新的版本；而为了加快部署速度，我们可以只运行和部署相关的部分。
这就是标签所可以做到的。当然在这里 "deploy" 只是一个字符串，它没有任何意义以及它可以是任意的内容。
下面让我们看看怎么通过标签来部署站点：

    $ ansible-playbook -i step-08/hosts -l host1.example.org step-08/apache.yml -t deploy 
    X11 forwarding request failed on channel 0

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Deploy our awesome application] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=2    changed=1    unreachable=0    failed=0    

看起来不错，[接下来](https://github.com/leucos/ansible-tuto/tree/master/step-09)让我们部署另外一个 web 服务器实例。
