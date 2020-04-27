# Maven

为什么会使用到maven

1. 管理jar
2. 解决冲突
3. (远程仓库(中央)、本地仓库、私服仓库)


jdk 1.6以上

settings.xml 作用

1. 配置本地仓库地址
2. 私服地址
3. 权限

pom.xml的作用
1. 管理jar包信息

常用标签


项目版本
<modelVersion></modelVersion>

分组
<groupId></groupId>

jar包唯一Id
<artifactId></artifactId>

版本
<version></version>

可以引入多个依赖
<dependencies>
    <dependency></dependency>
</dependencies>
