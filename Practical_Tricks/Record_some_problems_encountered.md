>记录设置电脑时遇到的一些小问题。
#### 编码问题——菱形方块乱码

用**UTF-8**打开**GBK**编码导致。在**Chrome**上安装_Set Character Encoding_插件后右键单击页面，改成用GBK即可解决。

---

#### 命令行操作

- 如何查看异常中断/无法中止的进程并强行退出？

```powershell
top
```

然后按下 `P` 排序 CPU 使用率，按下 `M` 排序内存使用率。可以找到想要中止的进程，记住其ID，按`q`退出界面后，使用以下命令结束：

```powershell
kill 1234
kill -9 1234 # -9是强制中止
```

- 如何关闭在X Server的页面上打开的应用？（也使用于其他Linux的进程）

在命令行中输入：`Ctrl+C`，就可以回到命令行继续输入其他命令。

- 如何重命名文件？

  https://blog.csdn.net/ljh574649119/article/details/136707979

---

#### Docker相关

- 创建Dockerfile

  ```
  touch Dockerfile
  ```

- **Dockerfile**的某种格式

  ```dockerfile
  FROM node:14-alpine
  COPY index.js /index.js #COPY source dest
  CMD node /index.js #CMD name_of_executable_app parameter
  ```

- 命令行中的构建镜像命令

  ```
  docker build -t hello-docker .
  ```

  其中`hello-docker`为镜像名称，而`.`表示目录位置。

  而后可以用`docker images`查看已有镜像。

- 运行

  `docker run hello-docker`

  或者上传到docker hub后使用`docker pull ...`来拉取自己的/直接拉取别人的镜像。

- **Docker Desktop**的使用

  简而言之，可以简化命令行的操作。

- **Volume 逻辑卷**

  容器中的数据卷不能持久化，需要利用Volume把某目录映射到宿主机的磁盘上。

- Docker Compose

  使用`docker-compose.yml`来解决各个服务的关联关系。

  执行`docker compose up`来安装各种环境。

---

#### Conda相关操作

- 删除某一环境

  ```
  conda remove --name [name] --all #删除该环境及之中所有的软件
  ```

- 查看已有环境 & 删除特定环境

  ```
  conda env list
   
  conda env remove -p [/path/to/your/env]
  conda env remove -p /home/kuucoss/anaconda3/envs/tfpy36   #example
  ```

----

#### Git相关操作

- `git remote`相关操作

```
git remote add origin https://github.com/ziyuliu258/myCS231n.git #初始化Origin与Skeleton仓库(前提远程要真的存在这个仓库！故先在github上建立repo)
git remote -v #列出当前remote的仓库信息
git remote set-url skeleton https://github.com/ziyuliu258/myCS231n.git #初始化之后修改相关信息
```

- 关于本地git通过ssh链接github时`time out`问题的解决方法：https://blog.csdn.net/the__future/article/details/130038818

  ```bash
  Host github.com
  HostName ssh.github.com
  User git
  Port 22
  PreferredAuthentications publickey
  IdentityFile C:\Users\DELL\.ssh\id_ed25519
  ```

  设置如上，之后解决问题。其实有可能是网络问题，可能根本不用配置这个`config`文件。（还有一种可能是代理开了`IPv6`地址，而似乎ssh不太支持？）
  
  > - 在经历设置DNS，设置`/c/Windows/System32/drivers/etc/hosts`文件，设置`config`文件等一系列操作后，最后在`Cloudflare`的DNS配置下，保持`hosts`文件的默认配置，关闭`vpn`的`IPv6`选项。最后成功连接`ssh`……此后最好不宜瞎折腾。
  > - 又将DNS置换成了`114`系列。仍然成功，同时保证了访问国内网站的解析速度。
  
- 中文乱码解决：https://www.cnblogs.com/sdlz/p/13023342.html

  _（尚未解决）_
  
- 解决`unrelated histories`问题

  在需要执行的git操作（merge或者pull）后面加上`--allow-unrelated-histories`。
  
- 关于改变DNS后无法访问GitHub的问题：https://blog.csdn.net/weixin_53333244/article/details/139971837

- 如果推送失败

  报错信息例：

  ```bash
  error: failed to push some refs to 'github.com:ziyuliu258/ziyuliu258.git'
  ```

  使用`git reset --hard origin/master`退回重新提交。这样会将本地文件重置为最后一次成功提交的样子，所以需要有备份。
  
- 推送显示成功，但在github上不显示

  使用`git config --list`查看当前的邮件，如果和github账号中的设定email不同，则无法显示更新。
  
- 关于**Submodule**的问题

  https://zhuanlan.zhihu.com/p/87053283

----

#### 搭建网站相关（Hexo） 

- 新建标签
  - 前提是保证对应主题文件夹中的`_config.yml`中打开了标签功能。先用`hexo new page tag_name`创建标签页，然后在其中的`index.md`文件中加上`type: 'tags'`一行，标签创建完成。但此时还不能显示。
  - 将自己的一篇文章的`tags`设置成刚创建的值，然后运用相应命令重新生成，就可以在标签页看到对应的标签了。
- 找背景图的好网站：https://commons.wikimedia.org/wiki/Main_Page
- 设置音乐和电影页面：https://juejin.cn/post/7068673573581226021

----

#### 设置网络 

- ##### 关于Windows 11中的DNS设置（来自Epic官网）

1. 右键点击屏幕右下角的 Windows 图标。
2. 选择**系统**。
3. 选择“**网络和 Internet”。**
4. 如果您在使用网线连接，请选择**以太网**。 如果您在使用 Wi-Fi，请点击**Wi-Fi**，之后再点击**硬件属性**。
5. 点击“DNS 服务器分配”旁边的**编辑**。
6. 从下拉菜单中，选择**“手动”**。
7. 将 IPv4 设为“**开**”。
8. 设置想要用的DNS。
10. 选择“**保存**”。

- ##### DoH的设置

  参见此链接：https://blog.haysc.tech/dns-over-https-dnspod-windows-11/

  *（未完全解决）*

- ##### 关于DNS的选择问题

  https://www.cnblogs.com/lunatic-talent/p/12798368.html
  
  https://blog.csdn.net/leeskype/article/details/122338597