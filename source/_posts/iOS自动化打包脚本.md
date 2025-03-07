---
title: iOS自动化打包脚本
date: 2025-03-07 13:42:11
tags:
---

# iOS自动化打包脚本

> 闲来无事学习一下shell

#### 打包xcframework

- 由于xcode13的模拟器也包含arm64架构，移除模拟器arm64架构，`build Setting`=>`Excluded Architectures`=>勾选`Any ios simulator sdk` 添加`arm64` 也可在脚本中设置-arch

```
#!/bin/bash
set -e
set -u
set -o pipefail

SCHEME_NAME=sdk
WORKSPACE_NAME=sdk
#打包环境
CONFIGURATION=Release

#cd到脚本绝对路径下
ROOT_PATH=$(cd "$(dirname "${BASH_SOURCE[0]}")"; pwd)
#输出路径
EXPORT_PATH=${ROOT_PATH}/archive


#chech scheme
xcodebuild -list || exit 0

#clean
function arch_clean {
    xcodebuild clean \
    -workspace ${WORKSPACE_NAME}.xcworkspace \
    -scheme ${SCHEME_NAME} \
    -configuration ${CONFIGURATION}
}

#下面iphonesimulator15.5 可根据 xcodebuild -showsdks查看
 # #打包模拟器
function archive_simulator {
     xcodebuild archive \
     -workspace ${WORKSPACE_NAME}.xcworkspace \
     -scheme ${SCHEME_NAME} \
     -destination="iOS Simulator" \
     -sdk iphonesimulator15.5 \ 
     -configuration ${CONFIGURATION} \
     SKIP_INSTALL=NO \
     BUILD_LIBRARIES_FOR_DISTRIBUTION=YES \
     -archivePath ${EXPORT_PATH}/ios_simulator.xcarchive
 }

function archive_iosDevice {
     xcodebuild archive \
     -workspace ${WORKSPACE_NAME}.xcworkspace \
     -scheme ${SCHEME_NAME} \
    -sdk iphoneos \
     -destination="iOS" \
     -configuration ${CONFIGURATION} \
     SKIP_INSTALL=NO \
     BUILD_LIBRARIES_FOR_DISTRIBUTION=YES \
     -archivePath ${EXPORT_PATH}/ios_device.xcarchive
 }

 # #创建xcframework
 function create_xcframework {
     xcodebuild -create-xcframework \
     -framework ${EXPORT_PATH}/ios_device.xcarchive/Products/Library/Frameworks/${SCHEME_NAME}.framework \
     -framework ${EXPORT_PATH}/ios_simulator.xcarchive/Products/Library/Frameworks/${SCHEME_NAME}.framework \
     -output ${EXPORT_PATH}/${SCHEME_NAME}.xcframework
 }


 arch_clean || exit 0
 archive_simulator || exit 0

 if [[ -e ${EXPORT_PATH}/ios_simulator.xcarchive ]] ;then
     echo -e "\033[1135m🍺🍺🍺🍺🍺🍺xcarchive ios_simulator success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌xcarchive ios_simulator fail!❌❌❌❌❌❌ \033[0m"
 fi

archive_iosDevice || exit 0

if [[ -e ${EXPORT_PATH}/ios_device.xcarchive ]] ;then
     echo -e "\033[1135m🍺🍺🍺🍺🍺🍺xcarchiveios_device success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌xcarchive ios_device fail!❌❌❌❌❌❌ \033[0m"
 fi

 cd ${EXPORT_PATH} 
 create_xcframework || exit 0
 if [[ -e ${EXPORT_PATH}/${SCHEME_NAME}.xcframework ]] ;then
    echo -e "\033[1135m🍺🍺🍺🍺🍺🍺create ${SCHEME_NAME}.xcframework success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌create ${SCHEME_NAME}.xcframework fail!❌❌❌❌❌❌ \033[0m"
 fi

 open .
```

#### 打包ipa

把shell脚本和ExportOptions.plist一起放在根目录下，下面例子中ExportOptions.plist为ad-hoc模式。

```
#!/bin/bash
set -e
set -u
set -o pipefail

SCHEME_NAME=SdkApp
WORKSPACE_NAME=sdk
#打包环境
CONFIGURATION=Release
#cd到脚本绝对路径下
ROOT_PATH=$(cd "$(dirname "${BASH_SOURCE[0]}")"; pwd)
#输出路径
EXPORT_PATH=${ROOT_PATH}/archive
EXPORT_IPA_PATH=${EXPORT_PATH}/ipa

#chech scheme
xcodebuild -list || exit 0

#clean
function arch_clean {
    xcodebuild clean \
    -workspace ${WORKSPACE_NAME}.xcworkspace \
    -scheme ${SCHEME_NAME} \
    -configuration ${CONFIGURATION}
}

#编译
function archive {
    xcodebuild archive \
    -workspace ${WORKSPACE_NAME}.xcworkspace \
    -scheme ${SCHEME_NAME} \
    -configuration ${CONFIGURATION} \
    -archivePath ${EXPORT_PATH}/${SCHEME_NAME}.xcarchive
}

#打包导出
function arch_export {
    xcodebuild -exportArchive \
    -archivePath ${EXPORT_PATH}/${SCHEME_NAME}.xcarchive \
    -exportPath ${EXPORT_IPA_PATH} \
    -exportOptionsPlist ${ROOT_PATH}/ExportOptions.plist 
}


 arch_clean || exit 0
 archive || exit 0

 if [[ -e ${EXPORT_PATH}/${SCHEME_NAME}.xcarchive ]] ;then
     echo -e "\033[1;35m🍺🍺🍺🍺🍺xcarchive success！🍺🍺 \033[0m"
 else
     echo -e "\033[1;35m❌❌ fail ❌❌❌❌\033[0m"
 fi

arch_export || exit 0

if [[ -e ${EXPORT_IPA_PATH}/${SCHEME_NAME}.ipa ]] ;then
     echo -e "\033[1;35m🍺🍺🍺🍺🍺xcarchive success！🍺🍺 \033[0m"
 else
     echo -e "\033[1;35m❌❌ fail ❌❌❌❌\033[0m"
 fi

open ${EXPORT_IPA_PATH}
```

ExportOptions.plist 文件<可手动打包一次后，生成>

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>destination</key>
    <string>export</string>
    <key>method</key>
    <string>ad-hoc</string>
    <key>signingStyle</key>
    <string>automatic</string>
    <key>stripSwiftSymbols</key>
    <true/>
    <key>thinning</key>
    <string>&lt;none&gt;</string>
</dict>
</plist>
```
