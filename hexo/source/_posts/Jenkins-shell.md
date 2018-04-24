---
title: Jenkins shell
date: 2018-04-24 15:46:53
tags: [Jenkins, shell]
---
分享项目的shell脚本，没有过多的注释，请自行分析，有不懂的可以联系
`echo 6bq76Iqx6JekIDU0MTgxNTg1MiB+fiAgICAgICAgICAgCgo= |base64 -D`

<!---more--->

```bash
#!/usr/bin/env bash
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

复杂点的

```bash
#!/usr/bin/env bash
cd ./OpenSdk
rm -rf output
name="other"
demoDir="dir"

applicationIdsh=$(echo "$ApplicationId" | awk '{gsub(/ /,"")}1')
appIdsh=$(echo "$AppId" | awk '{gsub(/ /,"")}1')
appSecretsh=$(echo "$AppSecret" | awk '{gsub(/ /,"")}1')

case $SDK_User in
    qilekang)
        cd SDK_qilekang
        name="七乐康"
        demoDir="QilekangDemo"
    ;;
    youdeyi)
   	    cd SDK_youdeyi
   	    name="有德医"
   	    demoDir="FacesignDemo"
    ;;
    fengyu)
        cd SDK_fengyu
        name="枫玉"
        demoDir="FengyuDemo"
    ;;
    yuexiu)
        cd SDK_yuexiu
        name="越秀集团"
        demoDir="YuexiuDemo"
    ;;
    pin_sign)
        demoDir="FacesignDemo"
        sed -i '' 's/IS_CA=true/IS_CA=false/g' ../${demoDir}/gradle.properties
        sed -i '' 's/IS_MUTILSIGN=true/IS_MUTILSIGN=false/g' ../${demoDir}/gradle.properties
        cd SDK_calogin_sign_living
        name="输pin码签署"
        sed -i '' 's/IS_CA=true/IS_CA=false/g' gradle.properties
        sed -i '' 's/IS_MUTILSIGN=true/IS_MUTILSIGN=false/g' gradle.properties
        ;;
    *)
        echo "other"
    ;;
esac
/Users/gdca/Documents/nn/gradle-2.14.1/bin/gradle clean
/Users/gdca/Documents/nn/gradle-2.14.1/bin/gradle assembleRelease

cd ..
cd ..

rm -rf output
cd ./OpenSdk/output/$SDK_User
FILENAME=`find . -type f -name "*.aar"`

#! 获取带后缀的aar文件名
aarName=${FILENAME##*/}

#! 获取不带后缀的aar文件名
aarNameforCompile=${aarName%.*}

#! 删除历史的aar 注意其他aar不要以gdca前缀命名
cd ../../../${demoDir}
find app/libs -name gdcasdk-*.aar -exec rm -rf {} \;

cd ../OpenSdk/output/$SDK_User

cp -f ${FILENAME}  ../../../${demoDir}/app/libs/${aarName}

cd ../../../${demoDir}

#! 替换demo里面app的build.gradle的aar引用
cd app
#! mac 在关键字一行后添加新的一行
sed -i '' '/'"compile(name: 'gdcasdk"'/a\
	tempdfds34\
' build.gradle
#! mac delete关键字一行
sed -i '' '/'"compile(name: 'gdcasdk"'/d' build.gradle

sed -i '' 's/tempdfds34/'"compile(name: '${aarNameforCompile}', ext: 'aar')"'/g' build.gradle

sed -i '' 's/minifyEnabled false/minifyEnabled true/g' build.gradle


#! mac 修改applicationId
sed -i '' '/'"applicationId"'/a\
	tempdfds34\
' build.gradle

sed -i '' '/'"applicationId"'/d' build.gradle

sed -i '' 's/tempdfds34/'"applicationId "'"'${applicationIdsh}'"'""'/g' build.gradle

#! mac 修改DemoApplication

cd src/main/java/com/gdca/sdk/demo/

sed -i '' '/'"SdkManager.setPublicSite"'/a\
	tempdfds34\
' DemoApplication.java

sed -i '' '/'"SdkManager.setPublicSite"'/d' DemoApplication.java

sed -i '' 's/tempdfds34/'"SdkManager.setPublicSite($PublicSite);"'/g' DemoApplication.java

sed -i '' '/'"SdkManager.init"'/a\
	tempdfds34\
' DemoApplication.java

sed -i '' '/'"SdkManager.init"'/d' DemoApplication.java

sed -i '' 's/tempdfds34/'"SdkManager.init(this, "'"AppIDtemp"'", "'"AppSecrettemp"'");"'/g' DemoApplication.java


sed -i '' 's/AppIDtemp/'${appIdsh}'/g' DemoApplication.java
sed -i '' 's/AppSecrettemp/'${appSecretsh}'/g' DemoApplication.java

cd ../../../../../../../..
/Users/gdca/Documents/nn/gradle-3.3/bin/gradle clean
/Users/gdca/Documents/nn/gradle-3.3/bin/gradle assembleDebug

cd ..

FILENAME=`find ./${demoDir}/app/build/outputs/apk -type f -name "*.apk"`
mkdir output
cp -f ${FILENAME}  ./output/app-release-$SDK_User.apk
Url="${JENKINS_URL}job/Android_SDK_Demo/"${BUILD_NUMBER}"/artifact/output/app-release-$SDK_User.apk"
echo ${Url}
java -jar /Users/gdca/Documents/nn/qr.jar url=${Url} image=app.jpg save=./output

rm -rf ./${demoDir}/app/build
rm -rf ./${demoDir}/build
rm -rf ./${demoDir}/.gradle

zip -q -r -o ${demoDir}.zip ./${demoDir}

cp -f ${demoDir}.zip  ./output/${demoDir}.zip

rm -rf ${demoDir}.zip

```