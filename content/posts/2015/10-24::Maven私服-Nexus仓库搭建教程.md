+++
title = "Maven私服-Nexus仓库搭建教程"
date = 2015-10-24 23:27:00
url = "archives/25058"
tags = ["Maven"]
categories = ["工作记录"]
featuredimagepreview = 'https://oss.lumingbao.cn/banner/banner2.jpg'
+++

## Maven私服 - Nexus仓库教程

* ## 一、Nexus介绍

>Nexus 是Maven仓库管理器，如果你使用Maven，你可以从Maven中央仓库 下载所需要的构件（artifact），但这通常不是一个好的做法，你应该在本地架设一个Maven仓库服务器，在代理远程仓库的同时维护本地仓库，以节省带宽和时间，Nexus就可以满足这样的需要。此外，他还提供了强大的仓库管理功能，构件搜索功能，它基于REST，友好的UI是一个extjs的REST客户端，它占用较少的内存，基于简单文件系统而非数据库。这些优点使其日趋成为最流行的Maven仓库管理器。

* ## 二、环境搭建 
    * ### 2.1 下载  
      http://www.sonatype.org/nexus/  
      NEXUS OSS [OSS = Open Source Software，开源软件——免费]  
      NEXUS PROFESSIONAL -FREE TRIAL [专业版本——收费]。  
      所以选择NEXUS OSS  
      ![示例](https://oss.lumingbao.cn/images/20210927/09802645d5554447b46cf8e543f64072.png)  
      找到Download andInstall Nexus OSS。下载ZIP的即可：  
      ![示例](https://oss.lumingbao.cn/images/20210927/9a2caea2a7bd42b9aff8152e0b29d055.png)
    * ### 2.2 配置  
      解压下载好的压缩包，等到如下目录结构：  
      ![示例](https://oss.lumingbao.cn/images/20210927/12a25670dd5e4876aa9ed825a71b8042.png)  
      配置环境变量：
      新建 NEXUS_HOME环境变量值为你解压的nexus根目录，如下图所示：  
      ![示例](https://oss.lumingbao.cn/images/20210927/176608f6d7514d808478c19a6ecd95ad.png)  
      再配置Path环境变量末尾加入 %NEXUS_HOME%\bin\jsw\windows-x86-64  
      最后的 windows-x86-64 部分取决有你的操作系统类型，如下图所示：  
      ![示例](https://oss.lumingbao.cn/images/20210927/7c395c7200b442a6ac2e263825dba0c9.png)
    * ### 2.3 启动  
      执行CMD命令进入命令提示符模式：  
      安装命令：install-nexus.bat  
      启动命令：start-nexus.bat  
      如果所示：  
      ![示例](https://oss.lumingbao.cn/images/20210927/739be24170e647e18ce9edf26503faa6.png)  
      启动成功以后本机访问：http://127.0.0.1:8081/nexus 出现如下界面，说明配置成功！  
      ![示例](https://oss.lumingbao.cn/images/20210927/e7defb4b5e0c487d8ef49be9c6cfc993.png)
* ## 三、使用教程
    * ### 3.1 代理外部Maven仓库  
        * #### 3.1.1 登陆  
          要管理Nexus，你首先需要以管理员身份登陆，点击界面右上角的login，输入默认的登录名和密码：admin/admin123，登陆成功后，你会看到左边的导航栏增加了很多内容：  
          ![示例](https://oss.lumingbao.cn/images/20210927/1eff7eff09b04979ba706f8ecb57a7b4.png)  
          这里，可以管理仓库，配置Nexus系统，管理任务，管理用户，角色，权限，查看系统的RSS源，管理及查看系统日志，等等。你会看到Nexus的功能十分丰富和强大，本文，笔者只介绍一些最基本的管理和操作。  
        * #### 3.1.2 代理Maven中央仓库  
          >点击左边导航栏的Repositories，界面的主面板会显示所有一个所有仓库及仓库组的列表，你会看到它们的Type字段的值有group，hosted，proxy，virtual。这里我们不关心virtual，只介绍下另外三种类型：  
          >* hosted，本地仓库，通常我们会部署自己的构件到这一类型的仓库。
          >* proxy，代理仓库，它们被用来代理远程的公共仓库，如maven中央仓库。
          >* group，仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组。  
          > 
          >由此我们知道，我们需要配置一个Maven中央仓库的proxy，其实Nexus已经内置了Maven Central，但我们需要做一些配置。点击仓库列表中的Maven Central，你会注意到它的Policy是release，这说明它不会代理远程仓库的snapshot构件，这是有原因的，远程仓库的snapshot版本构件不稳定且不受你控制，使用这样的构件含有潜在的风险。然后我们发现主面板下方有三个Tab，分别为Browse，Configuration和Mirrors，我们点击Configuration进行配置，你现在需要关心的是两个配置项：“Remote Storage Location”为远程仓库的地址，对于Maven Central来说是http://repo1.maven.org/maven2/；“Download Remote Indexes”顾名思义是指是否下载远程索引文件，Maven Central的该字段默认为False，这是为了防止大量Nexus无意识的去消耗中央仓库的带宽（中央仓库有大量的构件，其索引文件也很大）。这里我们需要将其设置为True，然后点击Save。在Nexus下载的中央仓库索引文件之后，我们就可以在本地搜索中央仓库的所有构件。下图展示了我们刚才所涉及的配置：  
          ![示例](https://oss.lumingbao.cn/images/20210927/d20375fd5a86498ca58a5f9cadfe6df4.png)  

        * #### 3.1.3 添加一个代理仓库  
          这里我们再举一个例子，我们想要代理Sonatype的公共仓库，其地址为：http://repository.sonatype.org/content/groups/public/。步骤如下，在Repositories面板的上方，点击Add，然后选择Proxy Repository，在下方的配置部分，我们填写如下的信息：Repository ID - sonatype；Repository Name - Sonatype Repository；Remote Storage Location - http://repository.sonatype.org/content/groups/public/。其余的保持默认值，需要注意的是Repository Policy，我们不想代理snapshot构件，原因前面已经描述。然后点击Save。配置页面如下：  
          ![示例](https://oss.lumingbao.cn/images/20210927/283cad1739ca46bdbd6b7c3d02e9598f.png)  

    * ### 3.2 代理外部Maven仓库  
      Nexus预定义了3个本地仓库，分别为Releases，Snapshots，和3rd Party。这三个仓库都有各自明确的目的。Releases用于部署我们自己的release构件，Snapshots用于部署我们自己的snapshot构件，而3rd Party用于部署第三方构件，有些构件如Oracle的JDBC驱动，我们不能从公共仓库下载到，我们就需要将其部署到自己的仓库中。  
      当然你也可以创建自己的本地仓库，步骤和创建代理仓库类似，点击Repository面板上方的Add按钮，然后选择Hosted Repository，然后在下方的配置面板中输入id和name，注意这里我们不再需要填写远程仓库地址，Repository Type则为不可修改的hosted，而关于Repository Policy，你可以根据自己的需要选择Release或者Snapshot，如图：  
      ![示例](https://oss.lumingbao.cn/images/20210927/17a0c95b878b44a8b1c868526da12488.png)  
    * ### 3.3 代理外部Maven仓库  
      Nexus中仓库组的概念是Maven没有的，在Maven看来，不管你是hosted也好，proxy也好，或者group也好，对我都是一样的，我只管根据groupId，artifactId，version等信息向你要构件。为了方便Maven的配置，Nexus能够将多个仓库，hosted或者proxy合并成一个group，这样，Maven只需要依赖于一个group，便能使用所有该group包含的仓库的内容。  
      Nexus预定义了“Public Repositories”和“Public Snapshot Repositories”两个仓库组，前者默认合并所有预定义的Release仓库，后者默认合并所有预定义的Snapshot仓库。我们在本文前面的部分创建了一个名为“Sonatype Repository”的仓库，现在将其合并到“Public Repositories”中。  
      点击仓库列表中的“Public Repositories”，然后选择下方的"Configuration" Tab，在配置面板中，将右边“Avaiable Repositories”中的“Sonatype Repository”拖拽到左边的“Ordered Group Repository”中，如图：  
      ![示例](https://oss.lumingbao.cn/images/20210927/314d4ffdb4ab4ca3927c39deb412e88b.png)  
      创建仓库组和创建proxy及hosted仓库类似，这里不再赘述。需要注意的是format字段需要填写“maven2”，添加你感兴趣的仓库即可。
* ## 四、总结  
  >本文介绍强大的仓库管理器——Nexus，包括如何下载安装Nexus，配置Nexus代理中央仓库，管理Nexus的代理仓库，本地仓库，以及仓库组。并帮助你了解如何通过Nexus搜索构件。最后，如何在Maven中配置Nexus仓库，以及如何部署构件到Nexus仓库中。这些都是Nexus中最基本也是最常用的功能。随着使用的深入，你会发现Nexus还有很多其它的特性，如用户管理，角色权限管理等等。  
  
  >Nexus的OSS版本是完全开源的，如果你有兴趣，你可以学习其源码，甚至自己实现一个REST客户端。  
  
  >马上拥抱Nexus吧，它是免费的！