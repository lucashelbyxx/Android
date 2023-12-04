# 常见命令

adb devices 设备连接状态

adb start-server

adb kill-server

adb reboot



adb root 

adb remount 将/system, /vendor (if present) and /oem (if present)置为可写模式，前提是先root，remount后的设备，可以直接安装apk，不受版本号限制



# 日志过滤



# 系统属性

**获取系统属性**

- adb shell getprop [获取Android系统所有的配置信息，包括各种版本号，内存分配大小，手机型号等等]
- adb shell getprop >temp/prop.txt [获取所有配置信息并保存到本地文件]
- adb shell getprop | grep "dalvik.vm.heapgrowthlimit" [获取内存分配的配置]
- adb pull /system/build.prop temp/prop.txt [拷贝编译配置文件，getprop包括build.prop信息]

**修改系统属性**，前提是这些配置是可写的：adb shell setprop [key] [value]

- adb shell setprop dalvik.vm.heapgrowthlimit 512m [将分配内存修改为512MB]

**修改日志等级，**前提是配置文件可写：adb shell setprop log.tag.YOUR_LOG_TAG [LEVEL]
 作用：比如你的TAG只能打印INFO等级的日志，DEBUG不打印无法分析问题，怎么办？修改等级，包括系统类，比如ActivityManager等等，也可以修改，方便分析问题。

- adb shell setprop log.tag.EngineBridge DEBUG [将TAG为EngineBridge的日志打印级别改为DEBUG]



# dumpsys命令

**查看所有系统服务信息**

- adb shell dumpsys

**查看ActvityManagerService的信息**

- adb shell dumpsys activity [查看ActvityManagerService所有信息]
- adb shell dumpsys activity package &package_name [查看当前应用]
- adb shell dumpsys activity activities [查看Activity组件信息]
- adb shell dumpsys activity top  [查看当前界面的UI信息(View Hierarchy)]
- adb shell dumpsys activity services  [查看Service组件信息]
- adb shell dumpsys activity providers  [查看ContentProvider组件信息]
- adb shell dumpsys activity broadcasts  [查看BraodcastReceiver信息]

**查看应用信息**

- adb shell dumpsys meminfo  [查看系统进程内存信息分布情况]
- adb shell dumpsys meminfo &package_name [查看当前应用内存情况]
- adb shell dumpsys package [查看安装包信息]
- adb shell dumpsys package  &package_name [查看当前应用安装包信息]

**输出系统崩溃日志**
 系统应用：

- adb shell dumpsys dropbox system_app_crash --print > crash.txt
- adb shell dumpsys dropbox system_app_anr --print > anr.txt
   三方应用：
- adb shell dumpsys dropbox data_app_crash --print > crash.txt
- adb shell dumpsys dropbox data_app_anr --print > anr.txt



# am命令

**参数包括**

- -n表示组件；-a表示动作；-d表示传入的数据；-t表示传入的类型；--es表示传入键值对

**用法举例**

- adb shell am start -a android.intent.action.CALL -d tel:10086  [ 拨打电话]
- adb shell am start -n com.android.browser/.BrowserActivity [启动浏览器]
- adb shell am start -W com.android.browser/.BrowserActivity [统计启动时间]
- adb shell am start -a android.intent.action.VIEW -d  [http://www.baidu.com](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.baidu.com) [打开网址]
- adb shell am stack list [查看所有activity的堆栈和任务信息]



# pm命令

- adb shell pm list packages [列出所有安装应用的包名]
- adb shell pm clear &package_name [删除指定应用的数据]



# 工作高频命令

**获取进程号的方式**

打印所有进程号：adb shell ps -A
打印指定应用进程号：adb shell ps -A | grep -E "&package_name"



**日志查看**

- 清除所有的缓存日志：adb locat -c 或 adb locat -c all 或 adb logcat -c main events radio system  crash
- 查看日志：adb logcat  -v threadtime | grep  -iE "&TAG1|&TAG2|..."
- 查看日志(window不支持grep，测试同事用方便)：adb logcat -s &TAG1 -s &TAG2
- 查看异常日志：
   adb logcat  -v threadtime | grep  -iE "AndroidRuntime"
   adb logcat  -v threadtime | grep  -iE "FATAL"
   adb logcat  -v threadtime | grep  -iE "Exception"
   adb logcat  -v threadtime | grep  -iE "System.err"
   adb logcat  -v threadtime | grep  -iE "ANR"
   adb logcat  -v threadtime | grep  -iE "crash"
- 保存日志：adb logcat -v threadtime > log.txt  或 adb logcat |tee log.txt
- 搜索日志：grep -rE "&TAG1|&TAG2" temp/log.txt > output.txt
- 查看界面跳转：adb logcat -b events



**查看应用内存信息**

- 查看实时内存：adb shell dumpsys meminfo &package_name



**操作应用**

- 清除应用所有数据：adb shell pm clear &package_name
- 强制停止应用进程：adb shell am force-stop &package_name



**启动Activity**

- 通过intent启动Activity:

1. 先root获取权限: adb root
2. adb shell 'am start



