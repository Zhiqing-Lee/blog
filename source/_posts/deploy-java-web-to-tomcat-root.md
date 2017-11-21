---
title: 部署Java Web应用到Tomcat的根目录
date: 2017-11-21 10:09:06
categories:
- Java
tags:
- Java
- Tomcat
---

之前写Java Web应用都是用的Spring Boot，直接生成 `jar` 文件进行执行，部署方面的一些细节Spring Boot基本都帮我搞定了。最近再写一个论坛程序时，突然就不想用Spring Boot了。最后采用了XML方式配置了SSM环境。顺便熟悉一下Spring相关配置文件的细节（学了以后基本没用过，已经忘得差不多了）。

因为某些原因，中途开发环境从Ubuntu转到了Windows。转到Windows后又因跟校实习中断了一段时间。回校后运行项目时发现Redis服务打不开了。于是就想着干脆直接上Docker算了，免得下次换开发环境又出各种问题。

因为开发时一直用的IDEA，配置好Tomcat后默认是部署到根目录的，使用Spring Boot时也是在根目录。所以开发时基本都是用的绝对路径或相对路径，没有使用上下文路径的习惯。如果直接复制war包多出来一个上下文怕是要出问题。

于是Google了一下如何将war包部署到Tomcat根目录，对比了多个博客，终于还是弄出来了。

这次先记录一下部署到Tomcat，下次再记录部署到Docker。

## 1.重新指定目录

1. 新建一个目录用于存放应用的目录，并将war包放入其中。这种方法不会破坏Tomcat默认目录。

2. 和上面一样在配置文件 `$CATALINA_HOME/conf/server.xml` 中找到如下结点 (`$CATALINA_HOME`为Tomcat安装目录)

```xml
<Host name="localhost"  appBase="webapps" 
        unpackWARs="true" autoDeploy="false">

    ...
</Host>
```

3. 修改该该结点的 `appBase` 值为上面创建的目录，如该目录位于 `$CATALINA_HOME` 中，可以直接写相对路径。

4. 在 `2` 中的结点里添加一个子结点：

```xml
<Host name="localhost"  appBase="folder" 
        unpackWARs="true" autoDeploy="false">

        <Context path="" docBase="appname" debug="0" />

    ...
</Host>
```

其中 `appname` 为应用名，及放入 `folder` 中的war包名。

启动Tomcat后，访问即可通过指定的地址及端口(`http://localhost:8080`)的根目录访问到刚刚部署的应用。

## 2.替换默认文件

1. 移动 `$CATALINA_HOME/webapps/ROOT` 目录到 `$CATALINA_HOME/webapps/ROOT_BAK`

这一步我在网上搜到的是删除 `ROOT` 目录里的所有内容，但我测试时必须要删除整个目录才生效。应该和Tomcat版本有关。

2. 在Tomcat配置文件 `$CATALINA_HOME/conf/server.xml` 中找到如下代码：

```xml
<Host name="localhost"  appBase="webapps" 
        unpackWARs="true" autoDeploy="false">

    ...
</Host>
```

3. 在 `2` 中的结点里添加一个子结点：

```xml
<Host name="localhost"  appBase="webapps" 
        unpackWARs="true" autoDeploy="false">

        <Context path="" docBase="appname" debug="0" />

    ...
</Host>
```

这样部署会破坏Tomcat的默认主页

## 3.总结

虽然说写的是两种方式，但其原理并没有区别。只是一个使用了Tomcat默认的目录，一个没使用而已。重要的是理解其中几个属性的用法。

1. `appBase` 的值为存放app的目录，`Host` 的子结点 `Context` 中的 `docBase` 的值是相对于它的。该值可以使用绝对目录或相对目录。使用相对目录时是相对于 `$CATALINA_HOME` 目录的。

2. `path` 配置该上下文的路径，如为空则为根目录。

3. `docBase` 的值为具体应用的路径，可以设置为war包或者一个目录。可以使用相对路径或绝对路径。相对路径相对于 `appBase` 的值。