---
title: Mac版本破解starUML
date: 2019-05-03 23:22:04
tags: [developTools]
categories: developTools
---
## 下载并安装
   - 1 从官网http://staruml.io/下载 StarUML，因为是dmg,可以直接安装
   - 2 安装完成后，需要先安装asar
   ```
      npm install asar -g
   ```
     
## 破解
   - 1 安装成功默认装在/Applications/StarUML.app/Contents/Resources，进入该目录
   - 2 解压文件 asar extract app.asar app
   ```
      asar extract app.asar app
   ```
   - 3 修改js文件app/src/engine/license-manager.js
   ```
      vim app/src/engine/license-manager.js ,可通过find命令查找，输入/checkLicenseValidity
   ```
   - 4 找到checkLicenseValidity ()方法，修改成如下代码：
   ```
      checkLicenseValidity () {
          this.validate().then(() => {
            setStatus(this, true)
          }, () => {
            setStatus(this, true)//false 改为true
            // UnregisteredDialog.showDialog()
          })
        }
   ```
   - 5 重新打包app文件夹asar pack app app.asar
   ```
      asar pack app app.asar
   ```
   - 破解成功，重启StarUML即可开启画图的乐趣

  
