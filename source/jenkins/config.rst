基础配置
++++++++++

* 什么是 jenkins?

    持续集成、自动测试、持续部署的超级引擎，支持自定义工具集、多种交付通道。

镜像管理
"""""""""""""""

Jenkins 官方提供了一个`镜像列表 <http://mirrors.jenkins-ci.org/status.html>`_，会列出最快的镜像。

进入 Jenkins 页面，点击 Manager Jenkisn --> Manager Plugins --> Advanced 

将 ``Update site`` 更换为

.. code-block:: none

    https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json

Jenkins 目录
"""""""""""""""

+--------------------------------+----------+
|              目录              |   内容   |
+--------------------------------+----------+
|      ``/var/lib/jenkins``      |  主目录  |
+--------------------------------+----------+
| ``/var/lib/jenkins/workspace`` | 工作空间 |
+--------------------------------+----------+
|    ``/etc/init.d/jenkins``     | 启动文件 |
+--------------------------------+----------+
|   ``/etc/sysconfig/jenkins``   | 配置文件 |
+--------------------------------+----------+
|     ``/var/cache/jenkins``     | 程序文件 |
+--------------------------------+----------+
|      ``/var/log/jenkins``      | 日志文件 |
+--------------------------------+----------+

卡启动问题
""""""""""""""

Jenkins 在第一次启动的时候会向官网回传信息，如果网络在线，但是 Jenkins 不能访问 https://jenkins-ci.io 
这时关闭网络，离线就能正常安装

备份
""""""""""""

.. code-block:: bash

    tar zcvf jenkins.tar.gz /var/lib/jenkins

写一个每天定时备份的脚本，保留 15 天的备份。

.. code-block:: bash

    #!/usr/bin/env bash

    BACKUP_PATH="/opt/jenkins_backup"
    JENKINS_HOME_PATH="/var/lib/jenkins"
    BACKUP_ROTATE="15"
    DATE_DAY=`date +%F`

    function check_backup_dir() {
        test -d ${BACKUP_PATH} || mkdir -p ${BACKUP_PATH}
    }

    function backup_rotate() {
        DELETE_DATE_DAY=`date -d "-${BACKUP_ROTATE} day ago" +%F`
        if [ -f ${BACKUP_PATH}/jenkins_${DELETE_DATE_DAY}.tar.gz ]; then
            rm -rf ${BACKUP_PATH}/jenkins_${DELET_DATE_DAY}.tar.gz
        fi
    }

    function Backup() {
        check_backup_dir
        backup_rotate
        echo "[`date +'%Y-%m-%d %H:%M:%S'`] start backup"
        if [ ! -f ${BACKUP_PATH}/jenkins_${DATE_DAY}.tar.gz ]; then
            start_backup_second=`date +%s`
            cd ${JENKINS_HOME_PATH} && tar -zcPf ${BACKUP_PATH}/jenkins_${DATE_DAY}.tar.gz .
            stop_backup_second=`date +%s`
            let use_time=${stop_backup_second}-${start_backup_second}
            echo "[`date +'%Y-%m-%d %H:%M:%S'`] use time ${use_time}s"
            echo "[`date +'%Y-%m-%d %H:%M:%S'`] file size `du -h ${BACKUP_PATH}/jenkins_${DATE_DAY}.tar.gz`"
        else
            echo "[`date +'%Y-%m-%d %H:%M:%S'`] Backup completed today!"
        fi
        echo "[`date +'%Y-%m-%d %H:%M:%S'`] stop backup"
    }

    function Recovery() {
        find ${BACKUP_PATH} -name jenkins*.tar.gz | cut -d '/' -f 4 | cut -d "_" -f 2 | cut -d '.' -f 1 | nl
        list_length=`find /opt/jenkins_backup/ -name jenkins*.tar.gz | wc -l`
            read -p "Select date recovery >>> " number
        if grep '^[[:digit:]]*$' <<< "$number"; then
            if [ $number -gt $list_length -o $number -le 0 ]; then
                echo "[`date +'%Y-%m-%d %H:%M:%S'`] The selected date does not exist!"
            else
                date=`find ${BACKUP_PATH} -name jenkins*.tar.gz | cut -d '/' -f 4 | cut -d "_" -f 2 | cut -d '.' -f 1 | sed -n ${number}p`
                tar -zvxf ${BACKUP_PATH}/jenkins_${date}.tar.gz -C ${JENKINS_HOME_PATH}
            fi
        else
            echo "Please enter the correct number"
        fi
    }

    case $1 in
        backup)
            Backup
        ;;
        recovery)
            Recovery
        ;;
        *)
            echo "Usage: $0 {backup|recovery}"
                exit 1
        ;;
    esac

设置定时任务

.. code-block:: none

    0 2 * * * bash /opt/jenkins_backup/jenkins_toolbox.sh backup

构建状态
"""""""""""""

Jenkins 会基于一些后处理器任务为构建发布一个稳健指数（从 0 ~ 100），这些任务一般以插件的方式实现。

他们可能包括单元测试（JUnit）、覆盖率（Cobertura）和静态代码分析（FindBugs）。

分数越高，表明构建越稳定。下图中分级符号概述了稳定性的评分范围。任何构建作业的状态（总分100）低于80分就是不稳定的。

.. image:: /images/jenkins/构建状态.png

+------+--------------------------------+
| 颜色 |              状态              |
+======+================================+
| 蓝色 |   完成构建，被认为是稳定构建   |
+------+--------------------------------+
| 黄色 | 完成构建，被认为是不稳定的构建 |
+------+--------------------------------+
| 红色 |            构建失败            |
+------+--------------------------------+
| 灰色 |           禁用了构建           |
+------+--------------------------------+

.. image:: /images/jenkins/构建稳定性.png

图例可以在 https://jenkins.renkeju.com:8080/legend 中查看

系统设置
"""""""""""""

工作目录设置
'''''''''''''''''''

在 Linux 环境中，Jenkins 的默认工作目录在 ``/var/lib/jenkins/``，但是我们有些需要特殊指定工作目录的项目，需要默认的 JENKINS_HOME 分开。

1. 进入一个项目，在【Genaral】里点击“高级”按钮

    .. image:: /images/jenkins/全局配置高级按钮.png

2. 配置指定自定义工作目录空间，但是需要特别注意目录权限

    .. image:: /images/jenkins/使用指定的目录.png

* Maven 项目设置
* 设置系统 JDK ANT Maven
* Jenkins Location
* 邮件通知
* Configure Global Security
