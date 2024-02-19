# Android 日志工具的使用
## Android 的日志工具类 Log (android.util.log)
- Log.v()：打印琐碎、意义小的日志信息。对应最低级别 verbose。
- Log.d()：打印调试信息，对应级别 debug。
- Log.i()：打印重要数据，对应级别 info。
- Log.w()：打印警告信息，对应级别 warn。
- Log.e()：打印程序中的错误信息。对应级别 error。
``` java
// 打开MainActivity，在onCreate()方法中添加一行打印日志的语句
class MainActivity : AppCompatActivity() {
    override onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.d("MainActivity", "onCreate execute")
    } 
}
```
Log.d()方法传入了两个参数：第一个参数是 tag，一般传入当前类名，主要用于打印信息进行过滤；第二个参数是 msg，即想要打印的具体内容。
