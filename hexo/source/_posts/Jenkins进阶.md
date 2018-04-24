---
title: Jenkins进阶
date: 2018-04-23 09:12:38
tags: [Jenkins]
---

进阶内容包括参数化构建、脚本构建、生成二维码以及qq通知
## 参数化构建
### git branch
1. 先安装插件Git Parameter
2. 参数构建里选择添加参数Git Parameter
3. 填写对应的字段
   ![](/assets/img/jenkinsgitparam.png)
<!---more--->  
### debug or release
1. 参数构建里选择添加参数String Parameter
2. 填写对应的字段
   ![](/assets/img/jenkinsstringparam.png)
### 是否上传到蒲公英
1. 参数构建里选择添加参数Boolean Parameter
2. 填写对应的字段
   ![](/assets/img/jenkinsbooleanparam.png)
    
## 脚本
根据参数进行改变打包策略,以下面的脚本为例子
```
#!/usr/bin/env bash
COMMIT_COUNT=$(git rev-list --count HEAD)
echo $COMMIT_COUNT
if $IS_OTHER_USE && [ $Environment == "Debug" ]
then
    sed -i '' 's/IS_OTHER_USE=false/IS_OTHER_USE=true/g' gradle.properties
fi

/Users/gdca/Documents/nn/gradle-3.3/bin/gradle clean

if [ $Environment == "Debug" ]
then
    /Users/gdca/Documents/nn/gradle-3.3/bin/gradle assembleDebugChannels
else
    /Users/gdca/Documents/nn/gradle-3.3/bin/gradle assembleReleaseChannels
fi
```

其中IS_OTHER_USE和Environment均为参数构建的值，这个脚本比较简单不再作详细解释。
	

二维码
本地or蒲公英
```
if $pgyer && [ $Environment == "Debug" ]
then
    FILENAME=`find ./app/build/outputs/channels -type f -name "*-debug.apk"`
    curl -F "file=@${FILENAME}" -F "uKey=077fcc1c34cbbdcd2e1bebece1cad873" -F "_api_key=78fee71258387dfe53366056d0081693" https://qiniu-storage.pgyer.com/apiv1/app/upload
else
    if [ $Environment == "Debug" ]
    then
        FILENAME=`find ./app/build/outputs/channels -type f -name "*-debug.apk"`
        Url="${JENKINS_URL}job/CloudSign4Android/"${BUILD_NUMBER}"/artifact/"${FILENAME##*./}
        java -jar /Users/gdca/Documents/nn/qr.jar url=${Url} image=${BUILD_NUMBER}-latestapk.jpg save=./app/build/outputs/channels
    else
        FILENAME=`find ./app/build/outputs/channels -type f -name "*-yingyongbao-*-release.apk"`
        Url="${JENKINS_URL}job/CloudSign4Android/"${BUILD_NUMBER}"/artifact/"${FILENAME##*./}
        java -jar /Users/gdca/Documents/nn/qr.jar url=${Url} image=${BUILD_NUMBER}-latestapk.jpg save=./app/build/outputs/channels
    fi
fi
```

这里脚本看到如果上传到蒲公英则执行
```
curl -F "file=@${FILENAME}" -F "uKey=077fcc1c34cbbdcd2e1bebece1cad873" -F "_api_key=78fee71258387dfe53366056d0081693" https://qiniu-storage.pgyer.com/apiv1/app/upload
```
否则本地就要生成一个本地连接的二维码，其中生成二维码需要用到一个jar的工具
```
java -jar /Users/gdca/Documents/nn/qr.jar url=${Url} image=${BUILD_NUMBER}-latestapk.jpg save=./app/build/outputs/channels
```

显示二维码到Jenkins，首先需要description-setter-plugin这个插件，而且这个插件还需要修改代码，然后可以判断显示pgyer还是本地的二维码图片，修改后的源码在[github](https://github.com/DLTech21/description-setter-plugin)，生成hpi然后安装到Jenkins上。

安装后再回到任务配置添加构建步骤
![](/assets/img/jenkinsbuilddesc.png)

在Regular expression添加
```
.*qrcodeHistory\\/(\S{64})
```

在Description添加
```
<img src="http://static.pgyer.com/app/qrcodeHistory/\1">###<img src='${JENKINS_URL}job/CloudSign4Android/${BUILD_NUMBER}/artifact/app/build/outputs/channels/${BUILD_NUMBER}-latestapk.jpg'>
```

这样就可以实现二维码显示apk的下载地址


## qq通知
这个还是需要安装插件[qq-notify](https://github.com/DLTech21/NotifyQQ)

github上有比较详细的配置方法，如有问题再联系
```
echo 6bq76Iqx6JekIDU0MTgxNTg1MiB+fiAgICAgICAgICAgCgo= |base64 -D
```

## Jenkins升级备份
备份最重要就是保存好jenkins的文件夹，当需要更新jenkins.war的时候，通过修改
```
打开tomcat的bin目录，编辑catalina.sh文件。
在# OS specific support.  $var _must_ be set to either true or false.上面添加：export JENKINS_HOME=""
在引号中填入你的路径。
```