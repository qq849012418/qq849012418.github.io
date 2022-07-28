---
layout: post
title: Notion Guardian——将Notion自动备份到Github
categories:  Tools
description: none
keywords: notion, backup, github
---

一年多没更新了啊，Notion最近在技术人士中逐渐流行，但你是否怕这种云笔记某天突然跑路？若不想手动定期备份，这个工具可以帮你。

------

### 参考文献

1. [Automated Notion backups](https://artur-en.medium.com/automated-notion-backups-f6af4edc298d)
2. [Notion Guardian](https://github.com/richartkeil/notion-guardian)
3. [notion-backup](https://github.com/upleveled/notion-backup)

   

### 工具介绍

> Notion Guardian 提供了一种在私有存储库中设置数据安全备份的快速方法——允许您跟踪笔记随时间的变化并了解您的数据是否安全。
>
> 这个 repo 包含一个每天运行的 GitHub 工作流，每次推送到这个 repo。工作流将执行向 Notion 发出导出请求的脚本，等待它完成并将工作区内容下载到临时目录。然后，工作流会将这个目录提交到 repo secrets 中配置的存储库。

简单点就是：每天自动将你的notion导出到一个github私人仓库中，让鸡蛋不在一个篮子里。

### 用法

![image-20220729012414983](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220729012414983.png)

1. 为您的备份创建一个单独的私有存储库（例如“my-notion-backup”）。确保您创建了一个`main`分支——例如，在创建 repo 时单击“Add a README file”。

   ![image-20220728233511277](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728233511277.png)

2. fork存储库（见参考文献2），或者本人的修改版本https://github.com/qq849012418/Keenster-notion-guardian（修改了中文时区并默认为子页面创建文件夹，方便管理）。

3. 点击github右上角个人头像，进入settings。在左侧找到developer settings，选择新页面左侧的personal access tokens，创建勾选了“repo”范围的个人访问令牌（ [docs](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) 可参考) 。复制这个令牌， 回到刚刚fork的repo中，进入settings选项卡并在左侧找到Secrets-》Actions。点击New repository secret，创建一个密钥名称为`REPO_PERSONAL_ACCESS_TOKEN`，粘贴到value中。

   ![image-20220728234207122](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728234207122.png)

4. 同理，将您的 GitHub 用户名存储在`REPO_USERNAME`密钥中。

5. 同理，将新创建的私有仓库的名称存储在`REPO_NAME`密钥中（在本例中为“my-notion-backup”）。

6. 将用于提交更改的电子邮件（通常是您的 GitHub 帐户电子邮件）存储在`REPO_EMAIL`密钥中。

7. 获取您的 Notion space ID 和token。将其存储在`NOTION_SPACE_ID`和`NOTION_TOKEN`密钥中。获取过程需要用到浏览器。以谷歌浏览器为例（其他浏览器请参考文献1），在网页端打开你的notion space，点击space左侧settings&members，找到导出按钮，点击导出，此时会看到一个对话框。

   ![image-20220728235541098](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728235541098.png)

   接着，先使用ctrl+shift+j快捷键，呼出浏览器的debugger，切换到network页面，选择过滤XHR，并清空窗口。

   ![image-20220729011451288](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220729011451288.png)

   点击对话框中的导出按钮，可以看到debugger中多出了一个enqueueTask，在它的Header选项卡中，找到一个很长串的cookie，里面的token_v2后面的字符串就是notion token，而spaceid（格式为xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx）则在最下面的Request Payload项中。

8. 您还需要以相同的方式获取`notion_user_id`（在cookie中）并将其存储在一个`NOTION_USER_ID`密钥中。经过3-8步，你的actions界面应该是下面这样的。

   ![image-20220728234851175](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728234851175.png)

9. 单击fork repo上的“Actions”选项卡，然后通过单击按钮启用操作。

10. 在左侧边栏上单击“Backup Notion Workspace”工作流程。通知将告诉您“计划的操作”已禁用，因此请继续并单击按钮以启用它们。

11. 等到操作第一次运行或将**提交推送到存储库**以触发第一次备份。

    ![image-20220728235126626](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728235126626.png)

12. 检查您的私人存储库以查看您的 Notion 工作区数据是否已自动提交。完毕:cherry_blossom:

![image-20220728235250693](https://keenster-1300019754.cos.ap-shanghai-fsi.myqcloud.com/image-20220728235250693.png)
