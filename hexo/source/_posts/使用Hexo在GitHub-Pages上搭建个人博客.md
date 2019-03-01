---
title: 使用Hexo在GitHub Pages上搭建个人博客
categories:
  - Others
tags: hexo
abbrlink: 13700
date: 2018-12-12 17:21:46
---

	由于先前在电脑上搭建了hexo，未妥善保存，重装系统后再次使用就没法复原之前的环境了，所以本篇博文一是使用hexo在Github上搭建个人博客的流程，二是如何使之具备迁移能力。

## 一. 搭建可迁移的Hexo
1. 安装Nodejs和git环境，可通过`node -v`,`npm -v`,`git --version`查看版本。

2. 在Github新建repository，命名直接为`username.github.io`，当项目创建好时，可以在settings中看到已经创建好了Github Pages，路径为`https://username.github.io/`。创建hexo分支，作为项目的管理，master分支用来放文章。

3. 在git bash中执行`git clone git@github.com:zxy110/zxy110.github.io.git`，进入文件夹。

4. 配置Hexo： 创建空文件夹，进入该文件夹，执行`npm install hexo -g`，开始安装hexo，输入`hexo -v`，检查hexo是否安装成功，然后依次执行`hexo init`,`npm install`,`npm install hexo-generator-archive --save`,`hexo g`,
  `npm install hexo-deployer-git --save`。
  (若报错`npm WARN deprecated swig@1.4.2: This package is no longer maintained`，是因为国外镜像访问不到，切换到国内镜像就可以了: `npm config set registry https://registry.npm.taobao.org`)。

5. 本地体验hexo： 
  - 开启本地服务器：`hexo s`
  - 打开网址：localhost:4000/

6. 设置本地git（若已设置，可跳过）：
  - 先设置本地git账户：(git bash中执行)
  ```
  git config --global user.name "zxy110"
  git config --global user.email "zxy_whu@qq.com"
  ```
  - 输入`ssh-keygen -t rsa -C "zxy_whu@qq.com"`,连续三个回车，生成密钥。
  - 添加key到ssh： `ssh-add ~/.ssh/id_rsa`
  - 登录Github，点击头像下的settings/SSH and GPG keys中，添加ssh:新建一个new ssh key，将id_rsa.pub文件里的内容复制上去
  - 在git bash中输入`ssh -T git@github.com`，测试添加ssh是否成功。如果看到Hi后面是你的用户名，就说明成功了

7. 执行下列操作,将我们的项目放到分支hexo下：
  ```
  	git remote set-url origin git@github.com:zxy110/zxy110.github.io.git
  	git add --all 
  	git commit -m "新建分支资源文件"
  	git push --set-upstream origin hexo 
  ```

8. 配置Deployment， 在hexo文件夹下，找到_config.yml文件，修改repo值（在文件末尾）：
  ```
  deploy:
  	type: git
  	repository: git@github.com:zxy110/zxy110.github.io.git
  	branch: master
  ```
  执行`hexo g -d`生成网站并部署到GitHub上,部署成功后访问地址：http://zxy110.github.io ,那么将看到生成的文章。
  （若报错`Not a git repository (or any of the parent directories): .git`,是因为没有git库，执行`git init`即可。）

9. 自此，我们就实现了hexo分支管理项目，master分支管理文章。

10. 注意，如果在本地显示没有问题，deploy后显示不出主题框架，则是_config.yml中url的配置问题：

    ```
    # URL
    ## If your site is put in a subdirectory...
    url: https://zxy110.github.io/zxy/
    root: /zxy
    ```

    
## 二. 在本地修改博客
1. 更新项目分支hexo：
	```
		git remote set-url origin git@github.com:zxy110/zxy110.github.io.git
		git add --all 
		git commit -m "新建分支资源文件"
		git push --set-upstream origin hexo 
	```
2. 更新网站master分支：
	``` hexo g -d ```

## 三. 重装系统/在其他电脑上修改博客
1. 使用`git clone -b hexo git@github.com:zxy110/zxy110.github.io.git`拷贝仓库；
2. 在本地新拷贝的`zxy110.github.io`文件夹下通过Git bash依次执行下列指令：`npm install hexo`、`npm install`、`npm install hexo-deployer-git`（不需要`hexo init`这条指令）。

## 三. Hexo主题：snippet
我使用了Hexo的主题： https://github.com/shenliyang/hexo-theme-snippet ，其中使用了Travis CI（ https://www.travis-ci.org/ ）实现系统的集成，及Valine（ https://leancloud.cn/ ）实现评论功能。
Valine的配置：https://bigwin.ml/2018/11/29/valine-for-next/

> 参考: https://www.cnblogs.com/fengxiongZz/p/7707219.html , https://blog.csdn.net/wxl1555/article/details/79293159 
