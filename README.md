# Maven倉庫配置筆記

+ [發佈](#publish)


## 根據 [gradle-maven-push](https://github.com/chrisbanes/gradle-mvn-push) 配置你的lib專案

詳細流程 [gradle-maven-push](https://github.com/chrisbanes/gradle-mvn-push) 都已有詳細內容，這裡僅做一些重點整理。

#### 1. 在專案根目錄的`gradle.properties`中加入以下資訊
```properties
VERSION_NAME=1.0.0
VERSION_CODE=1
GROUP=org.arxing

POM_DESCRIPTION=This is a lib.
POM_URL=https://github.com/Arxing/xxxx
POM_SCM_URL=https://github.com/Arxing/xxxx.git
POM_SCM_CONNECTION=scm:git@github.com:Arxing/xxxx.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:Arxing/xxxx.git

POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo

POM_DEVELOPER_ID=arxing
POM_DEVELOPER_NAME=Arxing

# 這裡填寫maven配置到本機的路徑[file://]為必要的 不加這個會出錯
RELEASE_REPOSITORY_URL=file://S://maven-repository
SNAPSHOT_REPOSITORY_URL=file://S://maven-repository-snapshots
```
#### 2. 在每個你要發佈的module根目錄中建立`gradle.properties`並填入以下內容
```properties
POM_NAME=arxing-lib
# 此id為唯一值
POM_ARTIFACT_ID=demo-lib
POM_PACKAGING=aar
```

#### 3. 在每個你要發佈的module的`build.gradle`中加入
```gradle
apply from: 'https://raw.github.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle'
```
如果程式碼中含有中文則可能會編不過
只要再加上以下即可
```gradle
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}

tasks.withType(Javadoc) {
    //.java or .kotlin
    excludes = ['**/*.java']
    options.addStringOption('Xdoclint:none', '-quiet')
    options.addStringOption('encoding', 'UTF-8')
    options.addStringOption('charSet', 'UTF-8')
}
```
<h1 id='publish'></h1>

#### 4. 最後準備發佈lib到本地maven 打開終端機並下指令

```bash
$ gradlew clean build uploadArchives
```

#### 5. 執行成功以後會在你配置的路徑下生成文件 接著要將本地maven發佈到網路上

本例的路徑為`S://maven-repository`

#### 6. 本地maven發佈至github

注意: 若您的repository設定為private的 小弟目前尚未找到方法使用 請跳至第7點

+ 直接將你的本地maven發佈至github上 其url格式為

`https://raw.githubusercontent.com/<USER>/<MAVEN-PROJECT-NAME>/<BRANCH>`
+ 在project級的`build.gradle`中加入此來源
```gradle
allprojects {
    repositories {
        maven { url "https://raw.githubusercontent.com/Arxing/XXX/master" }
    }
}
```
+ 至此遠端maven配置已完成 開心的使用你的lib吧!
```gradle
implementation '<groupId>:<artifactId>:<version>'
```

#### 7. 本地maven發佈至aws s3

+ 在s3上建立一個bukkit或隨便找個地方存放您的maven 如建立了一個`sample.maven.repo`的bukkit
+ 將您本地的maven資料夾上傳至s3
+ 自己建立或向管理員取得一組可以存取此bukkit的`access key`及`secret access key`
+ 在project級的`build.gradle`中加入插件依賴
```gradle
classpath 'com.amazonaws:aws-java-sdk-core:1.11.410'
```
+ 在project級的`build.gradle`中加入此來源
```gradle
allprojects {
    repositories {
        maven {
            url "s3://<BUKKIT_NAME>/<MAVEN_ROOT>/"
            credentials(AwsCredentials) {
                accessKey <YOUR_ACCESS_KEY>
                secretKey <YOUR_SECRET_KEY>
            }
        }
    }
}
```

+ 至此遠端maven配置已完成 開心的使用你的lib吧!
```gradle
implementation '<groupId>:<artifactId>:<version>'
```

## Maven倉庫結構

+ **maven根目錄**
    + **release**
        + **groupId**
            + **artifactId**
                + **version**
                    + artifactId-version.aar
                    + artifactId-version.aar.md5
                    + artifactId-version.aar.sha1
                    + artifactId-version.pom
                    + artifactId-version.pom.md5
                    + artifactId-version.pom.sha1
                    + artifactId-version-javadoc.jar
                    + artifactId-version-javadoc.jar.md5
                    + artifactId-version-javadoc.jar.sha1
                    + artifactId-version-sources.jar
                    + artifactId-version-sources.jar.md5
                    + artifactId-version-sources.jar.sha1                    
                + maven-metadata.xml
                + maven-metadata.xml.md5
                + maven-metadata.xml.sha1
    + **snapshot** (結構同release)




