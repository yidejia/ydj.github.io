---
title: Jenkins+GitLab+Android+iOS+Sonar搭建持续集成环境
date: 2016-11-02 14:12:55
tags:
- Jenkins
categories:
- 移动端

---
![Jenkins持续集成系统](http://upload-images.jianshu.io/upload_images/2083649-6bdfa1e51a834687.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

持续集成是敏捷开发的重要一环,它具备以下优点:

> 减少并降低软件开发中的风险
> 
> 将重复性工作自动化，让开发人员更专注于代码
> 
> 在任何时间、任何地点生成可部署的软件

随着人员以及项目的增加,以上这些就变得尤为重要.所以我们也在恰当的时机把它引入进来.

而市面上持续集成系统琳琅满目,有Jenkins,Travis,Circleci,Bitrise,Flow.ci 我们该如何选择,那就参考大数据,朝内还是用百度指数,目前只收录Jenkins和Travis.

> Jenkins:蓝色      Travis:绿色

![Jenkins和Travis百度搜索趋势](http://upload-images.jianshu.io/upload_images/2083649-669cc44ae2cb7f3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Jenkins和Travis百度搜索人群画像](http://upload-images.jianshu.io/upload_images/2083649-36cdd1235c8b8416.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Jenkins更为成熟，它是框架式的，大部分功能通过插件的方式来实现，可扩展性非常高.

废话少说,直接来撸一撸!

下载[Android SDK](http://www.androiddevtools.cn/ "Android SDK")

下载[Jenkins 安装](https://jenkins.io/ "Jenkins 安装")

通过HomreBrew安装启动和停止jenkins服务:

> brew install jenkins
> 
> brew tap homebrew/services
> 
> Start Jenkins: sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
> 
> Stop Jenkins: sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
> 
> 开机启动 Run at boot:  /Library/LaunchDaemons 可以在这里修改jenkins相关信息
> 
> Run at login: /Users/Lavare/Library/LaunchAgents

 通过pkg文件安装的可以通过 java命令启动:

> java -jar /Applications/Jenkins/jenkins.war

# Jenkins安装配置各种插件: #
**Git  ---- GitLab ---- Gitlab Authentication plugin**  (Git授权插件)

 **Environment Injector Plugin** (环境变量注入,目前用于获取gitLog,并传递给Fir.im上传信息)
 
**Email Extension**  (邮件扩展插件,打包完成后邮件通知各人员)

  **Fir Plugin**  (Fir.im上传插件,apk/ipa 分发渠道)

 **Bearychat Plugin** (上传到Bearychat插件,同于通知)

**Gradle Plugin**   (Android 构建插件)

 **Xcode integration**  (iOS构建插件)

**Keychains and Provisioning Profiles Management**  (iOS证书配置插件)

**Sonar:**代码质量管理平台,也是通过安装各种插件来扩展代码检测功能

> 1.糟糕的复杂度分布
> 
> 2.重复
> 
> 3.缺乏单元测试
> 
> 4.没有代码标准
> 
> 5.没有足够的或者过多的注释
> 
> 6.潜在的bug
> 
> 7.糟糕的设计

在系统管理界面配置->系统设置

GitLab配置,使用Api Token验证

![配置GitLab的Token](http://upload-images.jianshu.io/upload_images/2083649-6dcfef59a82a3822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Bearychat配置,方便后期构建成功后通知到相应群组

> * Team Subdomain: 在 https://ydj.bearychat.com 中，如yjd便是团队的 subdomain
> 
> * Integration Token: 在 BearyChat 中的 Jenkins 机器人的 hook 地址中， 最后的部分便是 token。
> 
> * Channel: 讨论组名称，如果指定的话，可以将 Jenkins 通知推送到该讨论组
> 
> * Build Server URL: 团队的 Jenkins 服务器所在的地址，用于构建 Jenkins 通知中的链接等信息
> 
> * Test Connection: 在填写上面的相关信息后，可以测试下是否配置成功

![Bearychat配置](http://upload-images.jianshu.io/upload_images/2083649-fabc61f59e3309fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置Sonar代码质量管理服务器

![Sonar服务器配置](http://upload-images.jianshu.io/upload_images/2083649-ffd0ddbe56747fb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在Global Tool Config配置jdk,git

![Jdk以及Git安装](http://upload-images.jianshu.io/upload_images/2083649-c3d6c9a0897b1b5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 项目配置 #
新建项目,使用自有构建模式
![项目类型选择](http://upload-images.jianshu.io/upload_images/2083649-59ba19827f5435b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

指定工作目录,和保持构建的天数,及最大个数,自定义目录

> 注意: 如果使用自定义工作空间,要考虑目录对当前是否有写入的权限,
![项目配置](http://upload-images.jianshu.io/upload_images/2083649-99d62727fde70207.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

源码管理使用http地址,也可以使用ssh要配key较麻烦
![GitLab项目配置](http://upload-images.jianshu.io/upload_images/2083649-8b1a1e0e4bc04cef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

构建触发器,配置gitlab的webhook,有push变化则自动构建打包,指定对某一个分支有效:
![GitLab触发配置](http://upload-images.jianshu.io/upload_images/2083649-4ffba65d72bad09e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![GitLab的Webhook配置](http://upload-images.jianshu.io/upload_images/2083649-b6b873d4b8a8bb68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

周期性触发器:

> Build periodically指周期性构建（Provides a cron-like feature to periodically execute this project.）
> 
> Poll SCM指周期性扫描远程git repository，当有变化时进行构建（Configure Jenkins to poll changes in SCM.）

日期定义

> Cron表达式字符串的格式为“分 小时 日 月 星期 年”，其中“年”是可选的，其余5个字段是必须的。
> 
> 区别（1）没有秒 （2）星期的取值范围是0-6（SUN-SAT）
> 
> 字段取值范围通配符分0-59* / , -时0-59* / , -日1-31* / , - ? L W月1-12 or JAN-DEC* / , -星期0-6 or SUN-SAT* / , - ? L #年1970–2099* / , -
> 
> 例子:
> 
> */5 * * * *  // 每5分钟
> 
> H/5 * * * *    // 每5分钟 推荐
> 
> 5 * * * *    // 每小时的第5分钟
> 
> 0 8 * * *    // 每天8点
> 
> 0 16,18,20,22 * * *    // 每天的16点、18点、20点、22点
> 
> 0 1,18 * * *    // 每天的1点和18点
> 
> 03 09 * * 1-5    // 工作日（周日到周五）的9点3分
> 
> 59 23 * * 1-5 或者 @midnight    // 工作日（周日到周五）的9点3分
> 
> 0 20 * * 1-5

H 20 * * 1-5 周一到周五的每晚8点自动构建
![周期性触发配置](http://upload-images.jianshu.io/upload_images/2083649-7ff86e40cb556c71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 自定义环境变量获取Git Log #

> 构建触发器配置完成后,我们还可以通过Environment Injector Plugin(环境变量注入插件),目前用于获取gitLog,并传递给Fir.im上传信息).

要先在Properties File Path 路径下创建一个对应的文件,否则改插件会报错找不到文件,名字可以自定义,但是要和Script Content的脚本内容中done> 后面的文件及路径一致.
![配置环境变量脚本](http://upload-images.jianshu.io/upload_images/2083649-4c36eb4c4654bbc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我这里用于测试后期,只需要显示昨天Git提交的信息:

GIT_CHANGE_LOG=$(git log --after="yesterday" --pretty=format:"%s")

echo "GIT_CHANGE_LOG=$(git log --after="yesterday" --pretty=format:"%s" | while read line

do 

echo $line\\\\\\\\n | tr -d \\n

done)"  ${WORKSPACE}/gitLogFile.properties

关于Git log的高级用法可以参考以下两篇文章:

[博客园-Git log高级用法](http://www.cnblogs.com/BeginMan/p/3577553.html "博客园-Git log高级用法")

[Github-Git log 高级用法](https://github.com/geeeeeeeeek/git-recipes/wiki/5.3-Git-log%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95 "Github-Git log 高级用法")

在构建中配置刚刚的文件路径,用户读取里面的环境变量打
![在构建配置文件路径](http://upload-images.jianshu.io/upload_images/2083649-ae552e1a05117ec1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后就可以在Fir插件上使用刚刚设置的环境变量${GIT_CHANGE_LOG}
![使用环境变量 ${GIT_CHAGE_LOG}](http://upload-images.jianshu.io/upload_images/2083649-aed8a25d8e40d731.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 构建分为Android和iOS构建,放在后面讲,先讲公共流程

构建期还可以通过Sonar来进行代码质量检查,配置源码路径,支持java,oc,swift,php,javascript

> sonar.projectKey=xxx_Android_Key
> 
> sonar.projectName=xxx_Android_Name
> 
> sonar.projectVersion=$BUILD_NUMBER
> 
> sonar.sourceEncoding=UTF-8
> 
> sonar.sources=/Users/jz_mac_mini/xxx/xxx/app/src/main 静态代码目录(必选)
> 
> sonar.java.libraries=/Users/jz_mac_mini/xxx/xxx/app/libs/ 第三方库目录(必选)
> 
> sonar.java.binaries=/Users/jz_mac_mini/xxx/xxx/app/build/intermediates/classes/production/ 编译后的代码目录(必选)

![项目Sonar配置](http://upload-images.jianshu.io/upload_images/2083649-b8b850104d5072e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以通过Sonar的后台查看代码分析
![Sonar分析结果后台](http://upload-images.jianshu.io/upload_images/2083649-98889ea0a711aa70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

构建后使用Fir.im发布,先安装ruby,

> $ gem sources --remove https://rubygems.org/ $ gem sources -a https://ruby.taobao.org/ $ gem sources -l *** CURRENT SOURCES *** https://ruby.taobao.org # 请确保只有 ruby.taobao.org, 如果有其他的源, 请 remove 掉
> 
> Mac OS X 10.11 以后的版本, 由于10.11引入了 rootless , 无法直接安装 fir-cli, 有以下三种解决办法:
> 
> 1. 使用 Homebrew 及 RVM 安装 Ruby, 再安装 fir-cli(推荐)
> 
> # Install Homebrew:
> 
> $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
> 
> # Install RVM:
> 
> $ \curl -sSL https://get.rvm.io | bash -s stable --ruby
> 
> $ gem install fir-cli

然后再通过插件配置Fir.im的账号,以及提交的fir的信息

> jenkins的build版本为:  $BUILD_NUMBER
> 
> git的提交版本  号 为 :  $GIT_COMMIT

![Fir.im配置](http://upload-images.jianshu.io/upload_images/2083649-a0282c1dc4955625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以通过bearychat插件来配置通知群组,在项目中配置
![Bearychat项目配置](http://upload-images.jianshu.io/upload_images/2083649-5dee6977d906121e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Bearychat通知打包结果](http://upload-images.jianshu.io/upload_images/2083649-f4b439f3a4c49603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以配置邮件通知,使用插件来配置email-ext-plugin,可以定制发送的内容,

先配置smtp服务器

![配置SMTP服务器](http://upload-images.jianshu.io/upload_images/2083649-5bf68c263b129b41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后指定发给哪些邮箱地址,以及定制发送的内容

![邮件发送内容](http://upload-images.jianshu.io/upload_images/2083649-157f4d1ceafdd28a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以使用自带的邮件通知,要先制定系统管理员邮件地址
![配置管理员邮件地址](http://upload-images.jianshu.io/upload_images/2083649-70038b72c58536ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后再配置邮件服务器smtp
![配置自带邮件SMTP服务器](http://upload-images.jianshu.io/upload_images/2083649-602532dab5417094.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# Android端构建配置 #

----------

Android SDK配置
![Android SDK配置](http://upload-images.jianshu.io/upload_images/2083649-907664fe4815d8e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

gradle可以指定本地的路径,也可以使用在线版本自动安装
![Gradle配置](http://upload-images.jianshu.io/upload_images/2083649-9ce14831d2c90d70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

构建,可以通过Tasks来指定任务
![Task任务配置](http://upload-images.jianshu.io/upload_images/2083649-fc0ac2fa2e4c20c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# iOS端构建配置 #

----------

目前我们使用的是Xcode默认配置打包,不用jenkins上配置的证书及PP文件.有几个关键点要注意的.

首先要在Xcode把项目中Projec和Target的Code Signing 设置为iOS Developer,以及把PP设置为Automatic.
![Code Sign和PP文件设置](http://upload-images.jianshu.io/upload_images/2083649-c9b39ae10a9c40ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在Product->Scheme->Manage Schemes 把项目设置为Shared,第三方库的不用设置
![scheme设置](http://upload-images.jianshu.io/upload_images/2083649-84e9d7c219bd9b3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统管理->找到Keychain and Provisioning Profiles Management

> 点击选择文件从本地选取Keychain,路径为:/Users/用户名/Library/Keychains/login.keychain

这里我们目前的做法是只配置login.keychain,然后jenkins的机子上安装Code Signing证书,再在xcode项目上fix issue,就能够为我们自动生成一个PP文件,名为-iOS Team Provisioning Profiles : xxxx, 而且当我们在开发者中心更新设备列表的时候,此PP文件是会自动加入的.
![Keychain and Provisioning Profiles配置](http://upload-images.jianshu.io/upload_images/2083649-6cf715e14895ceac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 针对CocoaPods设置 #

----------

如果项目是使用CocoaPods来管理的,还要针对xcworkspace进行build的话,还要额外进行写设置

如果构建是出现编码错误的可以在~/.bash_profile文件加上一句

> export LC_ALL="en_US.UTF-8"

Xcode构建前,先用shell跑一遍pod install,我这里会出现pod: command not found 的错误,所以要在前面加上一句

> #!/bin/bash -l
> 
> pod install --verbose --no-repo-update

![shell配置](http://upload-images.jianshu.io/upload_images/2083649-8e81bbcba58e0818.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置Workspace,红色的地方可以填上target名称,此处参考了[章华龙的文章](http://www.pluto-y.com/jenkins-xcode-configuration/ "章华龙的文章").
![配置Workspace](http://upload-images.jianshu.io/upload_images/2083649-a3c2636440e185ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置Workspace


# 非CocoaPods设置 #

----------

配置Xcode
![配置Xcode构建](http://upload-images.jianshu.io/upload_images/2083649-9196bc98b0d65a5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里可以填入keychain来解锁
![解锁Keychain](http://upload-images.jianshu.io/upload_images/2083649-94e742c8dba0e7e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个构建环境完成.

后期我们打算优化Sonar的检测分析,以及配置单元测试,UI测试,条件测试覆盖率等,继续往持续交付,持续部署的路上走!

Keep Going!