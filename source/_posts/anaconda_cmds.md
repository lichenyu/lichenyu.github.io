title: Anaconda操作命令
date: 2019/10/25
categories:
- Program Development
tags:
- Python
---


## cmds

|命令|说明|
|---|---|
|`conda --version`| 查看自身版本|
|`conda update conda`|更新自身|
|||
|`conda env list`|显示所有环境|
|`conda create --name <env_name> <package_names>`|新建环境。指定版本号，在包名后面`=版本号`；创建多个包以空格隔开。|
||`conda create -n python3 python=3.5 numpy pandas`|
|`(conda) activate <env_name>`|切入环境|
|`(conda) deactivate`|退出到root|
|`conda remove --name <env_name> --all`|删除环境|
|||
|`conda list`|显示环境中安装的包|
|`conda search <text>`|查找含有`<text>`字段的包，有哪些版本可供安装|
|`conda install <package_name>`|安装包（当前环境）。可指定版本号|
|`conda update --all`|更新所有包（当前环境）|
|`conda update <package_name>`|更新包（当前环境）。可指定版本号。更新多个指定包，则包名以空格隔开|
|`conda remove <package_name>`|删除包（当前环境）|
