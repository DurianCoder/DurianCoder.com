---
title: Maven打包插件-Assembly
date: 2018-08-26 21:36:35
tags:
	- Maven
---

## 0x01、assembly简介

[maven - 使用assembly plugin打包教程](maven - 使用assembly plugin实现自定义打包)

[Maven Assembly官方文档](http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html)

在项目中经常需要将配置文件打包到指定路径，就可以配置使用Assembly插件。

## 0x02、配置使用assembly

1、修改pom.cml

```
<build>
    <plugins>
         <plugin>
             <artifactId>maven-assembly-plugin</artifactId>
              <configuration>
                  <!-- not append assembly id in release file name -->
                  <appendAssemblyId>false</appendAssemblyId>
                   <descriptors>
                      <descriptor>src/main/assemble/package.xml</descriptor>
                   </descriptors>
              </configuration>
               <executions>
                  <execution>
                      <id>make-assembly</id>
                      <phase>package</phase>
                      <goals>
                         <goal>single</goal>
                      </goals>
                  </execution>
              </executions>
        </plugin>
    </plugins>
</build>
```

其中**<artifactId>maven-assembly-plugin</artifactId>**的maven-assembly-plugin是这个插件的标准命名，在maven2.0.*中带的默认版本是 appendAssemblyId属性控制是否在生成的打包文件的文件名中包含assembly id。
**descriptor**属性指定maven-assembly-plugin的配置文件，当然我设置的src/main/assemble/package.xml.容许使用多个，功能强大当然用法也复杂，对于简单情况一个足矣。
**execution**的设置是为了将maven-assembly-plugin继承到标准的maven打包过程中，这样在运行maven-package时就会执行maven-assembly-plugin的操作，从而实现我们需要的自定义打包。 

##### 2、assemble descriptor file

  我的src/main/assemble/package.xml内容如下： 

```
<assembly xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/assembly-1.0.0.xsd">
    <id>package</id>
    <formats>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>true</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>src/main/bin</directory>
            <outputDirectory>/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>src/main/config</directory>
            <outputDirectory>config</outputDirectory>
        </fileSet>
    </fileSets>
    <dependencySets>
        <dependencySet>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
</assembly>
```

  简单解释一下：

1）、<format>标签

​	用来设置打包格式，支持zip,tar,tar,gz,tar,bz

2）、<fileset>标签

​	将<directory>标签配置的路径下的文件打包到<outputDirectory>配置的路径下。

3）、<dependencySets>标签

​	将scope作用域下的依赖包打包到outputDirectory目录下



总结：在项目部署过程中，通过在pom.xml中引入maven-assembly-plugin，然在assemble descriptor file文件按需配置，maven install 可以将配置文件打包到指定位置。