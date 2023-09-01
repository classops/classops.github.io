---
layout: sonatype-publish
title: MavenCentral库发布记录
date: 2023-09-01 09:53:11
tags:
---
## MavenCentral库发布记录

### 一、注册 Sonatype 账号，新建项目

**注册**
https://​​issues.sonatype.org

**登录后，新建项目：**

相关选项，选择：

- **项目**：Community Support - Open Source Project - Repository Hosting (OSSRH)  
- **类型**：New Project  
- **概要**：填写 Git 项目名即可  
- **Group Id**: Github项目，io.github.xxx 用户名  
- **Project URL**: 填写 github 项目主页  
- **SCM url**: 填 github项目地址 + .git  

![新建](images/sona1.png)
![新建](./images/sona2.png)

**点击提交后，等待 3-5分钟 回复，验证 github 所有权。**

- 在自己项目里，在 github 创建 回复的 仓库
- 创建完成后，回复评论，大概意思就是即可： 仓库已创建

![回复](./images/sona3.png)

**等待5-10分钟后，验证成功**

这时候，就有上传到 sonatype nexus 仓库权限了


### 二、准备 GPG 密钥

GPG 来生成 密钥，安装 GPG 
Windows下使用scoop安装：
```
scoop install gpg

gpg --generate-key

gpg --list-keys
```
这里获取到pub公钥，

![GPG](./images/GPG.png)

创建完成，--list-keys 显示 pub 第二方 一串：  
67E95C8F2931C822********************F7E9  
后面用于 **上传和验证 公钥**

**上传公钥**
```
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 
67E95C8F2931C822********************F7E9

# 验证是否上传功能，后 8 位即可
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys ****F7E9
```

**导出密钥，后面签名用**

```
gpg --export-secret-keys 67E95C8F2931C822********************F7E9 > secret.gpg
```

### 三、Gradle项目配置

本地发布和远程发布，需要 maven-publish 插件，
GPG为了签名，需要 signing 插件。项目模块 build.gradle 配置：
```groovy
plugins {
    id 'maven-publish'
    id 'signing'
}
```

添加后，gradle 就多了 publish 任务：

![publish](./images/gradle-publish-task.png)

#### 配置发布

添加maven-publish后，开始配置 publishing 代码块

```groovy
// 配置发布后 groupId，默认 artifactId 是项目模块名
group = 'org.example'
version = '1.0'

task sourcesJar(type: Jar) {
    classifier = 'sources'
    from sourceSets.main.allSource
}

task javadocJar(type: Jar) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

publishing {
    publications {
        release(MavenPublication) {
            // 配置POM信息
            pom {
                name = project.name
                description = 'xxx'
                url = 'https://github.com/xxx/xxx'
                // 开源协议
                licenses {
                    license {
                        name = 'The Apache License, Version 2.0'
                        url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }

                // 配置开发者信息
                developers {
                    developer {
                        id = 'xxx'
                        name = 'xxx'
                        email = 'xxx@163.com'
                    }
                }

                // scm
                scm {
                    connection = 'https://github.com/xxx/xxx.git'
                    developerConnection = 'https://github.com/xxx/xxx.git'
                    url = 'https://github.com/xxx/xxx'
                }
            }

            // 发布文档 JAR
            artifact javadocJar
            // 发布源码 JAR
            artifact sourcesJar

            from components.java
        }
    }

    repositories {
        maven {
            url "https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/"
            credentials {                
                username project.sonaUsername // sonatype username
                password project.sonaPassword // sonatype password
            }
        }
    }
}

signing {
    sign publishing.publications.release
}
```

#### 配置 sonatype 账号 和 GPG密钥

上面配置，还不能直接使用，在 gradle.properties 配置用到的 变量：

对于密码信息，可以放到 用户目录下的 gradle.properties，
Windows下是：
```
C:\Users\用户名\.gradle\gradle.properties
```

添加 账号信息：
```properties
# sonatype 账号信息
sonaUsername=xxxx
sonaPassword=xxxx
# GPG Signing Info
signing.keyId=XXXXXXXX
signing.password=xxxxxx
# 上面导出的 GPG 密钥路径
signing.secretKeyRingFile=C:\\Users\\username\\.gpg\\secret.gpg
```

### 四、发布到仓库

项目下执行命令进行发布：
```bat
.\gradlew.bat publish
```

发布成功后，进入 snoatype nexus 后台，管理发布：
https://s01.oss.sonatype.org/#stagingRepositories

1. 在 Staging Repositories，勾选发布的仓库，点击Close并确定
2. 稍等几分钟，验证通过，点 Release
3. 10-30分钟后，会在 Maven Central 同步更新
- 可在 https://mvnrepository.com/ 搜索查看库


### 文档

- Android 发布: https://developer.android.com/build/publish-library/upload-library
- https://blog.51cto.com/u_12853553/5896541