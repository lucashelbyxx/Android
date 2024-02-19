### Dalvik与ART

Dalvik包含了一整套的Android运行环境虚拟机，每个App都会分配Dalvik虚拟机来保证互相之间不受干扰，并保持独立。它的特点是在运行时编译。而在Android 5.X版本开始，ART模式已经取代了Dalvik，ART采用的是安装时就进行编译，以后运行时就不用编译了。