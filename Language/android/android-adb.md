# Android ADB

Android Debug Bridge。借助这样一个工具，我们可以用电脑来操纵手机。

ADB工具位于SDK的platform-tools目录下，因此在命令行中使用ABD的时候，需要通过cd命令，切换到该目录下，或者将platform-tools的路径添加到系统环境变量中，这样就可以直接使用了。

```shell
adb shell	//进入Shell后，可以使用很多Linux下的Shell命令，如ls | grep "data"

android list targets	//显示系统中全部Android平台

adb install -r应用程序.apk	//安装Apk程序之Install
adb install -r F:\Test.apk

adb push <local> <remote>	//安装Apk程序之Push
adb push D:\Test.apk /system/app/

Adb Install是将Apk安装到data/data目录下，作为普通的用户应用程序。而Adb Push则不是安装命令，它是将一个文件写入手机存储系统。因此，只要拥有相应的权限，就可以把任何Apk放到任何目录下，甚至是放到System目录下，成为一个系统应用程序。Adb Push不仅可以安装Apk，它最大的用处还是向手机写入文件。

adb push <local> <remote>	//向手机写入文件
adb push D:\file.txt /system/temp/
adb pull <remote> <local>	//从手机获取文件
adb push /system/temp/ D:\file.txt
其实这些东西，在Android的Android Device Monitor工具中都可以直接使用

adb shell	//查看Log
logcat | grep "abc"

//删除应用
adb remount 	//（重新挂载系统分区，使系统分区重新可写）。
adb shell
cd system/app rm *.apk

adb shell df	//查看系统盘符

adb shell pm list packages –f	//输出所有已经安装的应用
adb shell input keyevent 3	//模拟按键输入，最后面的数字就是要执行的Keyevent的Code
adb shell input touchscreen <x1> <y1> <x2> <y2>	//模拟滑动输入
查看运行状态

adb shell dumpsys	//可以列出很多运行状态
dumpsys activity activities | grep "tencent"	//列出Activity的运行状态，同时过滤“tencent”关键字

Package管理信息
pm list packages –f	//列出所有的Package。通过PM命令，可以看见Android中关于Package的各种相关信息。

AM管理信息
adb shell am start -n包名/包名＋类名	//启动一个Activity

adb shell screenrecord /sdcard/demo.mp4	//录制屏幕
adb reboot	//重新启动
```

