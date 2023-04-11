# Github同步Gitee镜像仓库自动化脚本

---

## 前言

在软件开发中，使用Git作为代码管理工具是非常普遍的。而GitHub和Gitee则是我们熟知的两个在线Git代码托管平台。如果我们在这两个平台上都有代码仓库，并且希望实现自动同步，应该怎么做呢？这就需要使用GitHub Action中的Hub Mirror Action了。

## 什么是Hub Mirror Action？

![image-20230411153729450](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111537579.png)

### 1.介绍

[Hub Mirror Action](https://github.com/marketplace/actions/hub-mirror-action)是GitHub Action中的一个组件，可以将GitHub仓库内容自动同步到Gitee上，也可以实现从Gitee到GitHub的自动同步。Hub Mirror Action提供了多种同步方式，支持单向同步和双向同步，可以在配置文件中进行灵活设置。

### 2.用法

```yml
steps:
- name: Mirror the Github organization repos to Gitee.
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/kunpengcompute
    dst: gitee/kunpengcompute
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    account_type: org
    # src_account_type: org
    # dst_account_type: org
```

附：详细使用案例请查看官方仓库 [https://github.com/Yikun/hub-mirror-action](https://github.com/Yikun/hub-mirror-action)

## 配置步骤

### 1.生成密钥对

我们先在本地使用git命令行打开终端，输入如下命令：

```c
ssh-keygen -t rsa -f ~/Documents/ssh-key/id_rsa
```

注：请确保文件夹`~/Documents/ssh-key/`存在，当然你也可以选择放置在其他地方

过程中一路回车即可，注意不要设置密码。

![image-20230411154330166](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111543237.png)

![image-20230411155124878](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111551053.png)

### 2.GitHub私钥配置

首先为了存放自动化脚本，我们需要创建一个新的GitHub仓库，并为其配置相关环境。

![image-20230411154716849](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111547966.png)

* 依次点击`Settings->Secrets and variables->Actions`

* 点击`New respository secret`，创建一个名为`GITEE_PRIVATE_KEY`的secret，值为我们之前生成的密钥对中的私钥(id_rsa)

![image-20230411155314101](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111553202.png)

### 3.Gitee公钥配置

我们打开Gitee账号，进入`Settings->安全设置->SSH公钥`

添加一个名为gitee_sync的公钥，值也就是我们前面生成的公钥(id_rsa.pub)

![image-20230411155846562](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111558684.png)

### 4.Gitee生成私人令牌

![image-20230411155958898](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111559999.png)

令牌名称随意，同时复制生成的令牌值。

![image-20230411160127393](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111601466.png)

### 5.Github绑定Gitee令牌

* 依次点击`Settings->Secrets and variables->Actions`

* 点击`New respository secret`，创建一个名为`GITEE_TOKEN`的secret，值为Gitee生成的令牌值

![image-20230411160340721](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111603849.png)

### 6.编写CI脚本

将`ci_bot`仓库（放置及部署自动化脚本的仓库）下载到本地，同时创建这样的文件层次目录：
```c
.ci_bot/
    |——.github
    	|——workflows
    		|——Sync.yml
```

在`Sync.yml`文件中，添加以下代码：

```yml
name: Sync Github Repos To Gitee

on: [ push, delete, create ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:

    - name: Sync Github Repos To Gitee  # 名字随便起
      uses: Yikun/hub-mirror-action@master  # 使用Yikun/hub-mirror-action
      with:
        src: github/kurisaW  # 源端账户名(github)
        dst: gitee/kurisaW  # 目的端账户名(gitee)
        dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}  # SSH密钥对中的私钥
        dst_token:  ${{ secrets.GITEE_TOKEN }}  # Gitee账户的私人令牌
        account_type: user  # 账户类型
        clone_style: "https"  # 使用https方式进行clone，也可以使用ssh
        debug: true  # 启用后会显示所有执行命令
        force_update: true  # 启用后，强制同步，即强制覆盖目的端仓库
        static_list: "kurisaW_docs"  # 静态同步列表，在此填写需要同步的仓库名称，可填写多个
        timeout: '600s'  # git超时设置，超时后会自动重试git操作
```

保存退出后，将本次修改push到远端仓库。

查看Action运行情况：

![image-20230411161143741](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111611887.png)

### 7.多仓库同步推送

如果你想同时同步多个仓库，只需要完成如下修改

```yml
static_list 默认为'', 配置后，仅同步静态列表，不会再动态获取需同步列表（黑白名单机制依旧生效），如“repo1,repo2,repo3”。
```

![image-20230411163307283](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111633375.png)

![image-20230411163135259](https://raw.githubusercontent.com/kurisaW/picbed/main/img2023/202304111631352.png)

## 总结

通过以上步骤，我们已经完成了GitHub同步Gitee镜像仓库自动化脚本配置的操作。Hub Mirror Action作为GitHub Action中的一个组件，可以帮助我们在两个平台之间实现代码自动同步，极大地减轻了我们手动同步代码的工作量。当然如果你有任何问题环境留言区提出，我将竭力为你解答。