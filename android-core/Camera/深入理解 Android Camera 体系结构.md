[深入理解Android相机体系结构](https://blog.csdn.net/u012596975/article/details/107135938)

[相机开发_liujun3512159的博客](https://blog.csdn.net/liujun3512159/category_9529699.html)



# 一、相机简史

摄影是一门光与影的艺术，通过透镜将光线导入并依靠其折射特性，将光线最终导向到感光器件中，而感光器件在收到光线刺激之后进行一定的转换，进而形成影像，而这一系列的硬件设备的组合统一被称之为相机系统。

早期的相机系统十分简陋，同时成像效果也一直是灰白色调为主，但随着技术的不断革新，具有成像效果好，感光能力强的胶卷一经推出，便彻底提升了相机系统的成像效果。之后又随着CCD感光器件的发明，彻底将相机系统推入了数字成像时代，它直接将光信号转换为电子信号，进而转换为数字信号，最终将数据存储到计算机系统中。之后又在相机系统加入了图像处理模块，依托其强大的运算能力，成功地将成像效果再一次提升了一个维度，因此该时期的相机系统发展迅速，同时也基于技术的不断发展，各大厂商也推陈出新，打造了各式各样的相机系统，比较有代表性的便是单反和傻瓜相机，其中主打成像效果的单反相机，具有成像效果优秀，受到了众多摄像发烧友的追捧，而具有操作简单的傻瓜式相机，更加适合普通人群使用，在普通市场上反响也相当不错。之后随着手机的普及，对于使用手机进行拍照的需求越发强烈，同时随着制造工艺的进一步发展，一种低成本、小体积感光器件CMOS便顺势被推了出来，就这样手机端相机系统便正式登上了历史舞台。

相机系统在手机端起步较晚，初期的手机软件系统并没有对其有很好的支持，无论从开始的WINCE系统、还是塞班系统亦或是而今Android系统，在开始阶段，都只是简单实现了基本相机功能，使其仅仅能够满足简单的预览拍照录像需求，但是手机市场竞争异常激烈，各个厂商也看到了手机相机这块蓝海，便都投入了巨额资金进行研发，以Android系统为例，开始的相机系统功能简单，成像效果一般，并且无法满足用户的诸如高动态范围拍照等需求，但是经过了全球开发者的不懈努力，针对性为相机系统设计出了一套优秀的软件框架，并且借助一系列优秀的图像算法，再依托强大的硬件模块，而今Android相机系统在某些领域完全可以媲美专业相机。

而今的手机相机系统，除了成像效果有着显著的提高外，各大厂商在手机相机的功能性上也做足了功夫，最显著的代表便是单摄到多摄的演化，刚开始的手机系统仅仅采用了一个后置摄像头，但是由于人们对于自拍的需求日益增长，双摄系统便应运而生，一前一后两个相机模组，后者用于日常拍摄，前者用于自拍，之后随着时代的进步，互联网进一步的普及，将更多的专业照片带到了人们面前，由于审美能力的提高，人们对于手机相机系统便有了更高的要求，再加上技术水平的进一步提高，各大厂商便顺势提出了多摄相机系统，针对不同的拍摄场景或者拍摄效果在一部手机终端上集成多个相机模组，极大的满足了用户对于随手一拍便是大片的需求。

纵观整个Android相机系统的发展，从之前的小分辨率，一步步发展到而今的一亿像素，从之前的成像效果差强人意，到而今的完美呈现各种场景光影效果，从之前的单一模组到而今的多摄系统，它克服了一个又一个的技术难题，解决了一个又一个用户痛点问题，而其发展的背后都源于一个简单的目的，那就是让每一个人都能享受到科技带来的乐趣。



# 二、Android 相机架构概览

Android系统利用分层思想，将各层的接口定义与实现分离开来，以接口作为各层的脉络连接整体框架，将具体实现的主导权交由各自有具体实现需求的平台厂商或者Android 开发者，这样既做到把控全局，也给予了众多开发者足够大的创作空间。Google 根据职能不同将Camera框架一共划分成了五层，分别是App、Service、Provider、Driver以及Hardware，下面的Camera的整体架构图很清晰地显示出了其五层架构以及相互的关联接口。

![Camera-architecture](assets/Camera-architecture.png)

- **Camera App**

  应用层处于整个框架的顶端，承担着于用户直接进行交互的责任，承接来自用户直接或者间接的比如预览/拍照/录像等一系列具体需求，一旦接收到用户相关UI操作，便会通过Camera Api v2标准接口将需求发送至Camera Framework部分，并且等待Camera Framework回传处理结果，其中包括了图像数据以及整体相机系统状态参数，之后将结果以一定方式反馈给用户，达到记录显示种种美好瞬间的目的。

- **Camera Framework**

  该层主要位于Camera App与Camera Service之间，以jar包的形式运行在App进程中，它封装了Camera Api v2接口的实现细节，暴露接口给App进行调用，进而接收来自App的请求，同时维护着请求在内部流转的业务逻辑，最终通过调用Camera AIDL跨进程接口将请求发送至Camera Service中进行处理，紧接着，等待Camera Service结果的回传，进而将最终结果发送至App。

- **Camera Service**

  该层位于Camera Framework与Camera Provider之间，作为一个独立进程存在于Android系统中，在系统启动初期会运行起来，它封装了Camera AIDL跨进程接口，提供给Framework进行调用，进而接收来自Framework的图像请求，同时内部维护着关于请求在该层的处理逻辑，最终通过调用Camera HIDL跨进程接口将请求再次下发到Camera Provider中，并且等待结果的回传，进而将结果上传至Framework中。

- **Camera Provider**

  该层位于Camera Service与Camera Driver之间，作为一个独立的进程存在于Android系统中，同时在系统启动初期被运行，提供Camera HIDL跨进程接口供Camera Service进行调用，封装了该接口的实现细节，接收来自Service的图像请求，并且内部加载了Camera HAL Module，该Module由OEM/ODM实现，遵循谷歌制定的标准Camera HAL3接口，进而通过该接口控制Camera HAL部分，最后等待Camera HAL的结果回传，紧接着Provider通过Camera HIDL接口将结果发送至Camera Service。

- **CamX-CHI(Camera HAL)**

  该部分是高通对谷歌Camera HAL3接口的实现，以so库的形式被加载至Camera Provider中，之前采用的是QCamera & MM-Camera架构，但是为了更好灵活性和可扩展性，而今高通又提出了CamX-CHI架构，该架构提供HAL3接口给Provider进行调用，接收来自Provider的请求，而内部对HAL3接口进行了实现，并且通过V4L2标准框架控制着相机驱动层，将请求下发至驱动部分，并且等待结果回传，进而上报给Camera Provider。

  CamX-CHI架构由CamX和CHI两个部分组成，CamX负责一些基础服务代码的实现，不经常改动，CHI负责实现一些可扩展性和定制化的需求，方便OEM/ODM添加自己的扩展功能。CamX主要包括实现HAL3入口的hal模块，实现与V4L2驱动交互的csl模块，实现硬件node的hwl和实现软件node的swl。CHI通过抽象出Usecase、Feature、Session、Pipeline、Node的概念，使厂商可以通过实现Node接口来接入自己的算法，并通过XML文件灵活配置Usecase、Pipeline、Node的结构关系。

- **Camera Driver**

  Linux为视频采集设备制定了标准的V4L2接口，并在内核中实现了其基础框架V4L2 Core。用户空间进程可以通过V4L2接口调用相关设备功能，而不用考虑其实现细节。V4L2提出了总设备和子设备的概念，并通过media controller机制向用户空间暴露自己的硬件拓扑结构。视频采集设备驱动厂商按照V4L2 Core的要求开发自己的驱动程序，只需要实现相应的结构体和函数接口并调用注册函数注册自己就行。

  在高通平台上，高通对相机驱动部分进行了实现，利用了V4L2框架的可扩展特性，设计出了一套独特的KMD框架。在该框架内部主要包含了三个部分，CRM、Camera Sync以及一系列子设备，首先，作为框架顶层管理者，CRM创建了一个V4L2主设备用来管理所有的子设备，并且暴露设备节点video0给用户空间，同时内部维护着整个底层驱动业务逻辑。其次，Camera Sync创建了一个V4L2主设备，同时暴露了设备节点video1给用户空间，主要用于向用户空间反馈图像数据处理状态。最后，子设备模块被抽象成v4l2_subdev设备，同样也暴露设备节点v4l2-subdev给用户空间进行更精细化的控制。另外，在整个框架初始化的过程中，通过media controller机制，保持了在用户空间进行枚举底层硬件设备的能力。

- **Camera Hardware**

  相机硬件处在整个相机体系的最底层，是相机系统的物理实现部分，该部分包括镜头、感光器、ISP三个最重要的模块，还有对焦马达、闪光灯、滤光片、光圈等辅助模块。镜头的作用是汇聚光线，利用光的折射性把射入的光线汇聚到感光器上。感光器的作用是负责光电转换，通过内部感光元件将接收到的光信号转换为电子信号进而通过数电转换模块转为数字信号，并最后传给ISP。ISP负责对数字图像进行一些算法处理，如白平衡、降噪、去马赛克等。

  通过上面的介绍，我们可以发现，谷歌通过以上五级分层，形成了整个相机框架体系，其中层与层之间通过行业协会、开源社区或者谷歌制订的标准接口进行连接，上层通过调用标准接口下发请求到下层，下层负责对标准接口进行实现，最终将请求再次封装并调用下一层级的对外接口下发到下层。所以总得来说，谷歌使用标准接口作为骨架搭建整体框架，而其具体实现交由各层自己负责，从整体上来看，职责划分明确，界限分明，这样的设计，一来利用标准接口，保持了整个框架业务正常流转，二来极大地降低了各层耦合度，保持了各层的相互独立，最终让整个框架处于一个稳定同时高效的运行状态。



# 三、应用层

## 1、概览

相机应用处于整个框架的上层，在现实生活中，为了满足各式各样的应用场景，会加入很多业务处理逻辑，但是一旦当我们拨开繁杂的业务逻辑，便会发现其核心部分依然是通过调用谷歌制订的一系列Camera Api接口来完成的，而所有的相机行为都包含在该接口中。

起初，相机系统采用的是Camera Api v1接口，它通过一个Camera 类以及该类中的几个标准方法来实现整个相机系统的预览、拍照以及录像功能，控制逻辑比较简单，同时也比较容易理解，但也正是这种简单，导致了它无法逐帧控制底层硬件，无法通过元数据进行修改进而增强帧的表达能力，再加之应用场景的多样化趋势，该接口在新功能的实现上显得些许力不从心。面对该接口难以进一步扩展相机功能这一局面，谷歌在Andorid 5.0(API Level 21)便重新对Camera进行了设计，摒弃了Camera Api v1的设计逻辑，提出了一个全新的API – camera2，引入了Session以及Request概念，将控制逻辑统一成一个视图，因此在使用上更加复杂，同时也支持了更多特性，比如逐帧控制曝光、感光度以及支持Raw格式的输出等。并且由于对控制逻辑的高度抽象化，使得该接口具有很高的灵活性，可以通过简单的操作实现30fps的全高清连拍的功能，总得来说，该接口极大地提高了对于相机框架的控制能力，同时也进一步大幅度提升了其整体性能。

谷歌提出Camera Api v2接口的同时，将其具体实现放入了Camera Framework中来完成，Framework内部负责解析来自App的请求，并且通过AIDL跨进程接口下发到Camera Service中进行处理，并且等待结果的回传。接下来我们首先以Camera Api v2接口为主简单讲解下其逻辑含义，然后详细梳理下Camera Framework对于它的实现，最后以一个简单App Demo为例，来介绍下如何使用该接口来控制整个相机体系。

![camera-app](assets/camera-app.png)

## 2、Camera Api v2

回顾下Api v1接口的基本逻辑，该接口主要通过一个Camera.java类来定义了所有的控制行为，通过定义诸如open、startPreview、takePicture、AutoFocus等标准的接口来实现打开设备、预览、拍照以及对焦操作的功能，同时通过定义Camera.Parameters来实现了参数的读取与设置，其中包括了帧率、图片格式的控制，另外，通过定义了Camera.CameraInfo来实现了图像元数据的获取。而为了更加细致化地控制相机系统的Camera Api v2接口，相对于Api v1接口而言，复杂了许多，通过不同的接口类以及接口方法定义了复杂的相机系统行为，接下来逐一进行介绍：

**CameraManager**
谷歌将CameraManager定义为一个系统服务，通过Context.getSystemService来获取，主要用于检测以及打开系统相机，其中打开操作通过openCamera方法来完成。除此之外，还定义了getCameraCharacteristics方法来获取当前Camera 设备支持的属性信息，而该属性信息通过CameraCharacteristics来表示，其中包括了图像数据的大小以及帧率等信息。

**CameraDevice**
代表了一个被打开的系统相机，类似于Camera Api v1中的Camera类，用于创建CameraCaptureSession以及对于最后相机资源的释放。

**CameraDevice.StateCallback**
该类定义了一系列的回调方法，其实现交由App来完成，主要用于返回创建Camera设备的结果，一旦创建成功相机框架会通过回调其onOpened方法将CameraDevice实例给到App，如果失败，则调用onError返回错误信息。

**CameraCaptureSession**
该类代表了一个具体的相机会话，建立了与Camera设备的通道，而之后对于Camera 设备的控制都是通过该通道来完成的。当需要进行预览或者拍照时，首先通过该类创建一个Session，并且调用其setRepeatingRequest方法开启预览流程，或者调用capture方法开始一次拍照动作。

**CameraCaptureSession.StateCallback**
该接口类定义了一系列回调方法，其实现交由App完成，主要用于返回创建CameraCaptureSession的结果，成功则通过onConfigured方法返回一个CameraCaptureSession实例，如果失败则通过onConfigureFailed返回错误信息。

**CameraCaptureSession.CaptureCallback**
该接口类定义了一系列回调方法，用于返回来自Camera Framework的数据和事件，其中onCaptureStarted方法在下发图像需求之后立即被调用，告知App此次图像需求已经收到，onCaptureProgressed方法在产生partial meta data的时候回调，onCaptureCompleted方法在图像采集完成，上传meta data数据时被调用。

**CaptureRequest**
该类用于表示一次图像请求，在需要进行预览或者拍照时，都需要创建一个CaptureRequest，并且将针对图片的一系列诸如曝光/对焦设置参数都加入到该Request中，通过CameraCaptureSessin下发到相机系统中。

**TotalCaptureResult**
每当通过CameraDevice完成了一次CaptureRequest之后会生成一个TotalCaptureResult对象，该对象包含了此次抓取动作所产生的所有信息，其中包括关于硬件模块(包括Sensor/lens/flash)的配置信息以及相机设备的状态信息等。

**CaptureResult**
该类代表了某次抓取动作最终生成的图像信息，其中包括了此次关于硬件软件的配置信息以及输出的图像数据，以及显示了当前Camera设备的状态的元数据(meta data)，该类并不保证拥有所有的图像信息。



## 3、Camera Framework

基于接口与实现相分离的基本设计原则，谷歌通过Camera Api 接口的定义，搭建起了App与相机系统的桥梁，而具体实现便是由Camera Framework来负责完成的。在采用Camera Api v1接口的时期，该部分是通过JNI层来进行java到C++的转换，进而到达native层，而在native层会通过实现CameraClient建立与Camera Service的通讯 ，整个过程比较繁琐，使得整体框架略显繁杂，而随着Camera Api v2的提出，在该层便大量使用AIDL机制，直接在Java层建立与Camera Service的通信，进一步简化了整体框架。，接下来我们以几个主要接口为主线，简单梳理下其具体实现。

![camera-framework](assets/camera-framework.png)

CameraManager
实现主要在CameraManager.java中，通过CameraManager查询、获取以及打开一个Camera 设备。在该类中还实现了内部类CameraManagerGlobal，该类继承于ICameraServiceListener.Stub，在打开相机设备的时候，在内部会获取到ICameraService远程代理，并且调用ICameraService的addListener方法将自己注册到Camera Service中，一旦Camera Service状态有所变更便会通过其实现的回调方法通知到Camera Manager服务，另外，该类还通过调用ICameraService.connectDevice()方法获取到Camera Service中的CameraDevice远程代理，并且将该代理传入CameraDeviceImpl中，进而与Camera Service建立了连接。

CameraDeviceImpl
该类定义在CameraDeviceImpl.java文件中，继承并实现了CameraDevice接口，代表了一个相机设备，可以完成CameraCaptureSession的创建以及CaptureRequest创建等工作，内部定义了CameraDeviceCallbacks类(该类继承于ICameraDeviceCallbacks.Stub，对应于Camera Service中的 ICameraDeviceCallbacks接口)，用于接收来自Camera Service中的Camera Device的状态回调，并且内部维护着一个Camera Service 的远程ICameraDevice代理，进而可以下发图像请求到Camera Service中。

CameraCaptureSessionImpl
该类定义在CameraCaptureSessionImpl.java文件中，继承并实现了CameraCaptureSession接口，每一个相机设备在一个时间段中，只能创建并存在一个CameraCaptureSession，其中该类包含了两种Session，一种是普通的，适用于一般情况下的会话操作，另一种是用于Reprocess流程的会话操作，该流程主要用于对于现有的图像数据进行再处理的操作。该类维护着来自实例化时传入的Surface列表，这些Surface正是包含了每一个图像请求的数据缓冲区。

除了以上这几个接口，还有几个接口是需要App部分进行实现的，用于返回App所需要的对象或者数据：

CameraDevice.StateCallback
被App端进行继承并实现，用于在调用CameraManager的openCamera方法时，通过参数的形式传入Framework，在Framework中，一旦CameraDeviceImpl创建成功便通过其中的onOpened方法将其返回给App，如果失败，便会通过其他方法返回给App错误信息。

CameraCaptureSession.StateCallback
被App端进行继承并实现，用于在调用CameraDevice的createCaptureSession方法时作为参数传入Framework中，一旦创建成功，Framework便会通过调用该类的onConfigured接口返回一个CameraCaptureSessionImpl的对象，如果失败，Framework会调用其onConfigureFailed方法将错误信息返回至App。

CameraCaptureSession.CaptureCallback
被App端进行继承并实现，App通过调用CameraCaptureSessionImpl的setReaptingRequest或者capture方法是作为参数传入Framework，一旦Framework接收到来自CameraService的数据时，便会通过调用这个回调类将数据发送至App中。

![call](assets/call.png)

**a) openCamera**
当用户打开相机应用时，会去调用该方法打开一个相机设备，其中该方法最终经过层层调用会调用到Camera Framework中的openCameraDeviceUserAsync方法，在该方法中主要做了三件事：

首先是获取ICameraService代理，调用其getCameraInfo方法获取当前设备的属性。
其次是实例化了一个CameraDeviceImpl对象，并将来自App的CameraDevice.StateCallback接口存入该对象中，再将CameraDeviceImpl中的内部类CameraDeviceCallback作为参数通过ICameraService的connectDevice方法传入Camera Service去打开并获取一个ICameraDeviceUser代理，并将该代理存入CameraDeviceImpl中进行管理。
最后通过App传入的回调将CameraDeviceImpl返回给App使用，至此整个流程便完成了。
**b) createCaptureSession**
在打开相机设备之后便需要去创建一个相机会话，用于传输图像请求，其最终实现是调用该方法来进行实现的，而该方法会去调用到Camera Framework中的createCaptureSessionInternal方法，该方法主要做了两件事：

首先调用configureStreamsChecked方法来配置数据流。
其次实例化了一个CameraCaptureImpl对象，并通过传入CameraCaptureSession.StateCallback回调类将该对象发送至至App中。
而在configureStreamsChecked方法中会去调用ICameraDeviceUser代理的一系列方法进行数据流配置，其中调用cancelRequest方法停掉当前的的预览流程，调用deleteStream方法删除之前的数据流，调用createStream创建新的数据流，最后调用endConfigure来进行数据流的配置工作，针对性的配置便在最后这个endConfigure方法中。

**c) createCaptureRequest**
在创建并获取相机会话之后，便可以开始下发图像请求了，而在此之前，需要通过该方法来创建一个CaptureRequest，一旦调用该方法，最终会调用到Camera Service中ICameraDeviceUser的createDefaultRequest方法来创建一个默认配置的CameraMetadataNative，其次实例化一个CaptureRequest.Builder对象，并将刚才获取的CameraMetadataNative传入其中，之后返回该CaptureRequest.Builder对象，在App中，直接通过调用该Buidler对象的build方法，获取一个CaptureRequest对象。

CaptureRequest对象也创建成功了，接下来需要下发图像请求了，一般常用请求分为两种,一个是预览一个是拍照。

**d) setRepeatingRequest**
App调用该方法开始预览流程，通过层层调用最终会调用到Framework中的submitCaptureRequest方法，该方法主要做了两件事：

首先调用CameraService层CameraDeviceUser的submitRequestList方法，将此次Request下发到CameraService中。
其次将App通过参数传入的CameraCaptureSession.CaptureCallback对象存到CameraDeviceImpI对象中。
接下来看下拍照请求的处理流程：
**e) capture**
该方法最终也会调用到Framework中的submitCaptureRequest方法，接下来边和预览流程大致相同，会去调用Camera Service 中的ICameraDeviceUser的submitRequestList方法传入请求，之后将App实现的回调对象存入CameraDeviceImpl对象中。

**f) onCaptureProgressed**
一旦Request下发到Camera Service之后，当底层生成了Partial Meta Data数据，Camera Service会调用通过调用在打开相机设备时传入的ICameraDeviceCallback代理，通过其onResultReceived方法将数据传回Framework，之后调用App传入的CameraCaptureSession.CaptureCallback中的onCaputreProgressed方法将结果回传至App进行解析以及后处理。

**g) onCaptureCompleted**
一旦Request下发到Camera Service之后，当底层生成了Meta data数据，Camera Service会调用通过调用在打开相机设备时传入的ICameraDeviceCallback代理，通过其onResultReceived方法将数据传回Framework，之后调用App传入的CameraCaptureSession.CaptureCallback中的onCaputreCompleted方法将结果回传至App进行解析以及后处理。

**h) onImageAvailable**
之前已经通过两个回调接口onCaptureProgressed以及onCaptureCompleted方法将meta data上传到了App，一般情况下，图像数据会在他们之后上传，而且这个上传过程并不经过Camera Framework，而是通过BufferQueue来进行的，当Camera Service接收到底层传来的图像数据，便会立即调用processCaptureResult_3_4方法，该方法中会去调用BufferQueue中生产者角色的Surface的queueBuffer方法，将数据入队并通知消费者去消费，而此时的消费者正是App端的ImageReader，并经过一层层回调，最终会通过调用ImageReader的onImageAvailable方法，通知ImageReader去将数据取出，并做后期操作。

从上面的梳理不难发现，整个Camera Framework除了是对Camera Api v2的实现外，还承担着与Camera Service跨进程通信的任务，充当了一个位于App与Service之间的中转站的角色。

四、Camera App Demo
经过上面的梳理总结，我们已经对整个Camera Api v2接口以及实现都有了一个较为深入的认识，但是认识暂时仅仅停留在代码层面，为了更好理解其功能，接下来我们以一个简单的相机应用入手来加深下对接口的使用流程的理解：



# 四、服务层

## 1、简介

Camera Service被设计成一个独立进程，作为一个服务端，处理来自Camera Framework 客户端的跨进程请求，并在内部进行一定的操作，随后作为客户端将请求再一次发送至作为服务端的Camera Provider，整个流程涉及到了两个跨进程操作，前者通过AIDL机制实现，后者通过HIDL机制实现，由于在于Camera Provider通信的过程中，Service是作为客户端存在的，所以此处我们重点关注AIDL以及Camera Service 主程序的实现。



## 2、Camera AIDL 接口

在介绍Camera AIDL之前，不妨来简单了解下何为AIDL，谷歌为什么要实现这么一套机制？

在Android系统中，两个进程通常无法相互访问对方的内存，为了解决该问题，谷歌提出了Messager/广播以及后来的Binder，来解决这个问题，但是如果某个进程需要对另一个进程中进行多线程的并发访问，Messager和广播效果往往不是很好，所以Binder会作为主要实现方式，但是Binder的接口使用起来比较复杂，对开发者特别是初学者并不是很友好，所以为了降低跨进程开发门槛，谷歌开创性地提出了AIDL(自定义语言)机制，主动封装了Binder的实现细节，提供给开发者较为简单的使用接口，极大地提升了广大开发者的开发效率。

按照谷歌的针对AIDL机制的要求，需要服务端创建一系列*.aidl文件，并在其中定义需要提供给客户端的公共接口，并且予以实现，接下来我们来看下几个主要的aidl文件。

![camera-aidl](assets/camera-aidl.png)

ICameraService.aidl定义了ICameraService 接口，实现主要通过CameraService类来实现，主要接口如下：

```
getNumberOfCameras： 获取系统中支持的Camera 个数
connectDevice()：打开一个Camera 设备
addListener(): 添加针对Camera 设备以及闪光灯的监听对象
```

ICameraDeviceCallbacks.aidl文件中定义了ICameraDeviceCallbacks接口，其实现主要由Framework中的CameraDeviceCallbacks类进行实现，主要接口如下：

```
//CameraDeviceCallbacks callbacks
cameraUser = cameraService.connectDevice(callbacks, cameraId,
                    mContext.getOpPackageName(),  mContext.getAttributionTag(), uid,
                    oomScoreOffset, mContext.getApplicationInfo().targetSdkVersion);
 
onResultReceived： 一旦Service收到结果数据，便会调用该接口发送至Framework
onCaptureStarted()： 一旦开始进行图像的采集，便调用该接口将部分信息以及时间戳上传至Framework
onDeviceError(): 一旦发生了错误，通过调用该接口通知Framework
```

ICameraDeviceUser.aidl定义了ICameraDeviceUser接口，由CameraDeviceClient最终实现，主要接口如下：

```
disconnect： 关闭Camera 设备
submitRequestList：发送request
beginConfigure： 开始配置Camera 设备，需要在所有关于数据流的操作之前
endConfigure： 结束关于Camera 设备的配置，该接口需要在所有Request下发之前被调用
createDefaultRequest： 创建一个具有默认配置的Request
```

ICameraServiceListener.aidl定义了ICameraServiceListener接口，由Framework中的CameraManagerGlobal类实现，主要接口如下：

```
onStatusChanged： 用于告知当前Camera 设备的状态的变更
```



## 3、Camera Service 主程序

Camera Service 主程序，是随着系统启动而运行，主要目的是向外暴露AIDL接口给Framework进行调用，同时通过调用Camera Provider的HIDL接口，建立与Provider的通信，并且在内部维护从Framework以及Provider获取到的资源，并且按照一定的框架结构保持整个Service在稳定高效的状态下运行，所以接下来我们主要通过几个关键类、初始化过程以及处理来自App的请求三个部分来详细介绍下。

**1、关键类解析**
首先我们来看下几个关键类，Camera Service中主要包含了以下几个类，用于提供AIDL接口，并负责内部一系列逻辑的控制，并且通过HIDL接口保持与Provider的通信。

![camera-service-class](assets/camera-service-class.png)

首先我们看下CameraService的这个类，它主要实现了AIDL中ICameraService 接口，并且暴露给Camera Framework进行调用，这个类在初始化的时候会去实例化一个CameraProviderManager对象，而在实例化的过程中，该对象会去获取系统中所有的Camera Provider，并且在其内部实例化了对应Provider个数的ProviderInfo对象，并且随着每一个ProviderInfo的实例化，将一个Camera Provider作为参数存入ProviderInfo中，并且最终将所有的ProviderInfo存入一个vec容器中进行统一管理，就这样，CameraProviderManager便达到了管理所有的Camera Provider的目的。

而对于单个ProviderInfo而言，内部会维护一个Camera Provider代理，而在系统运行初期，ProviderInfo会去向Camera Provider获取当前这设备所支持的Camera 设备，拿到Camera Provider中的ICameraDevice代理，并且依次存入提前实例化好的DeviceInfo3对象中，最后会将所有的DeviceInfo3存入一个内部容器，进行统一管理，而DeviceInfo3维护着Camera Provider中的ICameraDevice代理，保持了对Camera Provider的控制。

另外，Camera Service 中还包含了CameraDeviceClient类，该类在打开设备的时候被实例化，一次打开设备的操作对应一个该类对象，它实现了ICameraDeviceUser接口，以AIDL方式暴露接口给Camera Framework进行调用，于此同时，该类在打开设备的过程中，获取了来自Camera Framework对于ICameraDeviceCallback接口的实现代理，通过该代理可以将结果上传至Camera Framework中，其中还包含了一个Camera3Device以及FrameProcessorBase，Camera3Device主要实现了对Camera Provider 的ICameraDeviceCallbacks回调接口的实现，通过该接口接收来自Provider的结果上传，进而传给CameraDeviceClient以及FrameProcessBase，其中，Camera3Device会将事件通过notify方法给到CameraDeviceClient，而meta data以及image data 会给到FrameProcessBase，进而给到CameraDeviceClient，所以FrameProcessBase主要用于metadata以及image data的中转处理。而Camera3Device中RequestThread主要用于处理Request的接收与下发工作。

对于Camera Service而言，主要包括了两个阶段，一个是系统刚启动的时候，会通过运行其主程序将其Camera Service 服务运行起来，等待Camera Framework的下发图像需求，另一个阶段就是当用户打开相机应用的时候，会去获取相机设备，进而开始图像采集过程，接下来我们就主要以这两个阶段分别来详细介绍下内部运行逻辑。

**2、启动初始化**
当系统启动的时候，会首先运行Camera Service的主程序，将整个进程运行起来，这里我们首先来看下Camera Service 是怎样运行起来的。

![camera-service-run](assets/camera-service-run.png)

当系统启动的时候会首先运行main_cameraserver程序，紧接着调用了CameraService的instantiate方法，该方法最终会调用到CameraService的onFirstRef方法，在这个方法里面便开始了整个CameraService的初始化工作。
而在onFirstRef方法内又调用了enumerateProviders方法，该方法中主要做了两个工作：

```
一个是实例化一个CameraProviderManager对象，该对象管理着有关Camera Provider的一些资源。
一个是调用CameraProviderManager的initialize方法对其进行初始化工作。
而在CameraProviderManager初始化的过程中，主要做了三件事：
 
首先通过getService方法获取ICameraProvider代理。
随后实例化了一个ProviderInfo对象，之后调用其initialize方法进行初始化。
最后将ProviderInfo加入到一个内部容器中进行管理。
```

而在调用ProviderInfo的initialize方法进行初始化过程中存在如下几个动作：

```
首先接收了来自CameraProviderManager获取的ICameraProvider代理并将其存入内部成员变量中。
其次由于ProviderInfo实现了ICameraProviderCallback接口，
    所以紧接着调用了ICameraProvider的setCallback将自身注册到Camera Provider中，
	接收来自Provider的事件回调。
再然后，通过调用ICameraProvider代理的getCameraDeviceInterface_V3_X接口，
    获取Provider端的ICameraDevice代理，
	并且将这个代理作为参数加入到DeviceInfo3对象实例化方法中，
    而在实例化DeviceInfo3对象的过程中会通过ICameraDevice代理的
	getCameraCharacteristics方法获取该设备对应的属性配置，并且保存在内部成员变量中。
最后ProviderInfo会将每一个DeviceInfo3存入内部的一个容器中进行统一管理，
    至此整个初始化的工作已经完成。
```

通过以上的系列动作，Camera Service进程便运行起来了，获取了Camera Provider的代理，同时也将自身关于Camera Provider的回调注册到了Provider中，这就建立了与Provider的通讯，另一边，通过服务的形式将AIDL接口也暴露给了Framework，静静等待来自Framework的请求。

**3、处理App请求**
一旦用户打开了相机应用，便会去调用CameraManager的openCamera方法进而走到Framework层处理，Framework通过内部处理，最终将请求下发到Camera Service中，而在Camera Service主要做了获取相机设备属性、打开相机设备，然后App通过返回的相机设备，再次下发创建Session以及下发Request的操作，接下来我们来简单梳理下这一系列请求在Camera Service中是怎么进行处理的。

a) 获取属性

对于获取相机设备属性动作，逻辑比较简单，由于在Camera Service启动初始化的时候已经获取了相应相机设备的属性配置，并存储在DeviceInfo3中，所以该方法就是从对应的DeviceInfo3中取出属性返回即可。

b) 打开相机设备
对于打开相机设备动作，主要由connectDevice来实现，内部实现比较复杂，接下来我们详细梳理下。
当CameraFramework通过调用ICameraService的connectDevice接口的时候，主要做了两件事情：

```
一个是创建CameraDeviceClient。
一个是对CameraDeviceClient进行初始化，并将其给Framework。
```

而其中创建CameraDevcieClient的工作是通过makeClient方法来实现的，在该方法中首先实例化一个CameraDeviceClient，并且将来自Framework针对ICameraDeviceCallbacks的实现类CameraDeviceImpl.CameraDeviceCallbacks存入CameraDeviceClient中，这样一旦有结果产生便可以将结果通过这个回调回传给Framework，其次还实例化了一个Camera3Device对象。

其中的CameraDeviceClient的初始化工作是通过调用其initialize方法来完成的，在该方法中：

```
首先调用父类Camera2ClientBase的initialize方法进行初始化。
其次实例化FrameProcessorBase对象并且将内部的Camera3Device对象传入其中，
    这样就建立了FrameProcessorBase和Camera3Device的联系，
    之后将内部线程运行起来，等待来自Camera3Device的结果。
最后将CameraDeviceClient注册到FrameProcessorBase内部，
    这样就建立了与CameraDeviceClient的联系。
```

而在Camera2ClientBase的intialize方法中会调用Camera3Device的intialize方法对其进行初始化工作，并且通过调用Camera3Device的setNotifyCallback方法将自身注册到Camera3Device内部，这样一旦Camera3Device有结果产生就可以发送到CameraDeviceClient中。

而在Camera3Device的初始化过程中，首先通过调用CameraProviderManager的openSession方法打开并获取一个Provider中的ICameraDeviceSession代理，其次实例化一个HalInterface对象，将之前获取的ICameraDeviceSession代理存入其中，最后将RequestThread线程运行起来，等待Request的下发。

```
mInterface = new HalInterface(session, queue, mUseHalBufManager, mSupportOfflineProcessing);
mRequestThread = new RequestThread(
            this, mStatusTracker, mInterface, sessionParamKeys,
            mUseHalBufManager, mSupportCameraMute);
```

而对于CameraProviderManager的openSession方法，它会通过内部的DeviceInfo保存的ICameraDevice代理，调用其open方法从Camera Provider中打开并获取一个ICameraDeviceSession远程代理，并且由于Camera3Device实现了Provider中ICameraDeviceCallback方法，会通过该open方法传入到Provider中，接收来自Provider的结果回传。

至此，整个connectDevice方法已经运行完毕，此时App已经获取了一个Camera设备，紧接着，由于需要采集图像，所以需要再次调用CameraDevice的createCaptureSession操作，到达Framework，再通过ICameraDeviceUser代理进行了一系列操作，分别包含了cancelRequest/beginConfigure/deleteStream/createStream以及endConfigure方法来进行数据流的配置。

c) 配置数据流
其中cancelRequest逻辑比较简单，对应的方法是CameraDeviceClient的cancelRequest方法，在该方法中会去通知Camera3Device将RequestThread中的Request队列清空，停止Request的继续下发。

beginConfigure方法是空实现，这里不进行阐述。

deleteStream/createStream 分别是用于删除之前的数据流以及为新的操作创建数据流。

紧接着调用位于整个调用流程的末尾–endConfigure方法，该方法对应着CameraDeviceClient的endConfigure方法，其逻辑比较简单，在该方法中会调用Camera3Device的configureStreams的方法，而该方法又会去通过ICameraDeviceSession的configureStreams_3_4的方法最终将需求传递给Provider。

到这里整个数据流已经配置完成，并且App也获取了Framework中的CameraCaptureSession对象，之后便可进行图像需求的下发了，在下发之前需要先创建一个Request，而App通过调用CameraDeviceImpl中的createCaptureRequest来实现，该方法在Framework中实现，内部会再去调用Camera Service中的AIDL接口createDefaultRequest，该接口的实现是CameraDeviceClient，在其内部又会去调用Camera3Device的createDefaultRequest方法，最后通过ICameraDeviceSession代理的constructDefaultRequestSettings方法将需求下发到Provider端去创建一个默认的Request配置，一旦操作完成，Provider会将配置上传至Service，进而给到App中。

d) 处理图像需求
在创建Request成功之后，便可下发图像采集需求了，这里大致分为两个流程，一个是预览，一个拍照，两者差异主要体现在Camera Service中针对Request获取优先级上，一般拍照的Request优先级高于预览，具体表现是当预览Request在不断下发的时候，来了一次拍照需求，在Camera3Device 的RequestThread线程中，会优先下发此次拍照的Request。这里我们主要梳理下下发拍照request的大体流程：

下发拍照Request到Camera Service，其操作主要是由CameraDevcieClient的submitRequestList方法来实现，在该方法中，会调用Camera3Device的setStreamingRequestList方法，将需求发送到Camera3Device中，而Camera3Device将需求又加入到RequestThread中的RequestQueue中，并唤醒RequestThread线程，在该线程被唤醒后，会从RequestQueue中取出Request，通过之前获取的ICameraDeviceSession代理的processCaptureRequest_3_4方法将需求发送至Provider中，由于谷歌对于processCaptureRequest_3_4的限制，使其必须是非阻塞实现，所以一旦发送成功，便立即返回，在App端便等待这结果的回传。

e) 接收图像结果
针对结果的获取是通过异步实现，主要分别两个部分，一个是事件的回传，一个是数据的回传，而数据中又根据流程的差异主要分为Meta Data和Image Data两个部分，接下来我们详细介绍下：

在下发Request之后，首先从Provider端传来的是Shutter Notify，因为之前已经将Camera3Device作为ICameraDeviceCallback的实现传入Provider中，所以此时会调用Camera3Device的notify方法将事件传入Camera Service中，紧接着通过层层调用，将事件通过CameraDeviceClient的notifyShutter方法发送到CameraDeviceClient中，之后又通过打开相机设备时传入的Framework的CameraDeviceCallbacks接口的onCaptureStarted方法将事件最终传入Framework，进而给到App端。

在Shutter事件上报完成之后，当一旦有Meta Data生成，Camera Provider便会通过ICameraDeviceCallback的processCaptureResult_3_4方法将数据给到Camera Service，而该接口的实现对应的是Camera3Device的processCaptureResult_3_4方法，在该方法会通过层层调用，调用sendCaptureResult方法将Result放入一个mResultQueue中，并且通知FrameProcessorBase的线程去取出Result，并且将其发送至CameraDeviceClient中，之后通过内部的CameraDeviceCallbacks远程代理的onResultReceived方法将结果上传至Framework层，进而给到App中进行处理。

随后Image Data前期也会按照类似的流程走到Camera3Device中，但是会通过调用returnOutputBuffers方法将数据给到Camera3OutputStream中，而该Stream中会通过BufferQueue这一生产者消费者模式中的生产者的queue方法通知消费者对该buffer进行消费，而消费者正是App端的诸如ImageReader等拥有Surface的类，最后App便可以将图像数据取出进行后期处理了。

初代Android相机框架中，Camera Service层就已经存在了，主要用于向上与Camera Framework保持低耦合关联，承接其图像请求，内部封装了Camera Hal Module模块，通过HAL接口对其进行控制，所以该层从一开始就是谷歌按照分层思想，将硬件抽象层抽离出来放入Service中进行管理，这样的好处显而易见，将平台厂商实现的硬件抽象层与系统层解耦，独立进行控制。之后随着谷歌将平台厂商的实现放入vendor分区中，彻底将系统与平台厂商在系统分区上保持了隔离，此时，谷歌便顺势将Camera HAL Moudle从Camera Service中解耦出来放到了vendor分区下的独立进程Camera Provider中，所以之后，Camera Service 的职责便是承接来自Camera Framework的请求，之后将请求转发至Camera Provider中，作为一个中转站的角色存在在系统中。



# 五、硬件抽象层

## 1、概览

始于谷歌的Treble开源项目，基于接口与实现的分离的设计原则，谷歌加入了Camera Provider这一抽象层，该层作为一个独立进程存在于整个系统中，并且通过HIDL这一自定义语言成功地将Camera Hal Module从Camera Service中解耦出来，承担起了对Camera HAL的封装工作，纵观整个Android系统，对于Camera Provider而言，对上是通过HIDL接口负责与Camera Service的跨进程通信，对下通过标准的HAL3接口下发针对Camera的实际操作，这俨然是一个中央枢纽般的调配中心的角色，而事实上正是如此，由此看来，对Camera Provider的梳理变得尤为重要，接下来就以我个人理解出发来简单介绍下Camera Provider。

Camera Provider通过提供标准的HIDL接口给Camera Service进行调用，保持与Service的正常通信，其中谷歌将HIDL接口的定义直接暴露给平台厂商进行自定义实现，其中为了极大地减轻并降低开发者的工作量和开发难度，谷歌很好地封装了其跨进程实现细节，同样地，Camera Provider通过标准的HAL3接口，向下控制着具体的Camera HAL Module，而这个接口依然交由平台厂商负责去实现，而进程内部则通过简单的函数调用，将HIDL接口与HAL3接口完美的衔接起来，由此构成了Provider整体架构。

![provider-call](assets/provider-call.png)

由图中可以看出Camera Provider进程由两部分组成，一是运行在系统中的主程序通过提供了标准的HIDL接口保持了与Camera Service的跨进程通讯，二是为了进一步扩展其功能，通过dlopen方式加载了一系列So库，而其中就包括了实现了Camera HAL3接口的So库，而HAL3接口主要定义了主要用于实现图像控制的功能，其实现主要交由平台厂商或者开发者来完成，所以Camera HAL3 So库的实现各式各样，在高通平台上，这里的实现就是我们本文重点需要分析的CamX-CHI框架。

在开始梳理CamX-CHI之前，不防先从上到下，以接口为主线简单梳理下Camera Provider的各个部分:

## 2、Camera HIDL 接口

首先需要明确一个概念，就是HIDL是一种自定义语言，其核心是接口的定义，而谷歌为了使开发者将注意力落在接口的定义上而不是机制的实现上，主动封装了HIDL机制的实现细节，开发者只需要通过*.hal文件定义接口，填充接口内部实际的实现即可，接下来来看下具体定义的几个主要接口：

![camera-hidl-interface](assets/camera-hidl-interface.png)

因为HIDL机制本身是跨进程通讯的，所以Camera Service本身通过HIDL接口获取的对象都会有Bn端和Bp端，分别代表了Binder两端，接下来为了方便理解，我们都省略掉Bn/Bp说法,直接用具体接口类代表，忽略跨进程两端的区别。
ICameraProvider.hal源码如下：



## 3、Camera Provider 主程序

接下来进入到Provider内部去看看，整个进程是如何运转的，以下图为例进行分析:

![camera-provider-in](assets/camera-provider-in.png)

在系统初始化的时候，系统会去运行android.hardware.camera.provider@2.4-service_64程序启动Provider进程，并加入HW Service Manager中接受统一管理，在该过程中实例化了一个LegacyCameraProviderImpl_2_4对象，并在其构造函数中通过hw_get_module标准方法获取HAL的camera_module_t结构体,并将其存入CameraModule对象中，之后通过调用该camera_modult_t结构体的init方法初始化HAL Module，紧接着调用其get_number_of_camera方法获取当前HAL支持的Camera数量，最后通过调用其set_callbacks方法将LegcyCameraProviderImpl_2_4（LegcyCameraProviderImpl_2_4继承了camera_modult_callback_t）作为参数传入CamX-CHI中，接受来自CamX-CHI中的数据以及事件，当这一系列动作完成了之后，Camera Provider进程便一直便存在于系统中，监听着来自Camera Service的调用。

![provider-listen](assets/provider-listen.png)

 接下来以上图为例简单介绍下Provider中几个重要流程：

```
Camera Service通过调用ICameraProvider的getCameraDeviceInterface_v3_x
    接口获取ICameraDevice，在此过程中，Provider会去实例化一个CameraDevice对象，
	并且将之前存有camera_modult_t结构体的CameraModule对象传入CameraDevice中，
    这样就可以在CameraDevice内部通过CameraModule访问到camera_module_t的相关资源，
	然后将CameraDevice内部类TrampolineDeviceInterface_3_2
   （该类继承并实现了ICameraDevice接口）返回给Camera Service。
Camera Service通过之前获取的ICameraDevice，调用其open方法来打开Camera设备，
    接着在Provider中会去调用CameraDevice对象的open方法，
	在该方法内部会去调用camera_module_t结构体的open方法，
    从而获取到HAL部分的camera3_device_t结构体，
    紧接着Provider会实例化一个CameraDeviceSession对象，
	并且将刚才获取到的camera3_device_t结构体以参数的方式传入CameraDeviceSession中，
    在CameraDeviceSession的构造方法中又会调用CameraDeviceSession的initialize方法，
	在该方法内部又会去调用camera3_device_t结构体的ops内的initialize方法
    开始HAL部分的初始化工作，
	最后CameraDeviceSession对象被作为camera3_callback_ops的实现传入HAL，接收来自HAL的数据或者 
    具体事件，当一切动作都完成后，Provider会将 
    CameraDeviceSession::TrampolineSessionInterface_3_2
    （该类继承并实现了ICameraDeviceSession接口）
	对象通过HIDL回调的方法返回给Camera Service中。
Camera Service通过调用ICameraDevcieSession的configureStreams_3_5接口进行数据流的配置，
    在Provider中，最终会通过调用之前获取的camera3_device_t结构体内ops的
    configure_streams方法下发到HAL中进行处理。
Camera Service通过调用ICameraDevcieSession的processCaptureRequest_3_4接口下发request请求到 
    Provider中，在Provider中，最终依然会通过调用获取的camera3_device_t结构体内ops中的 
    process_capture_request方法将此次请求下发到HAL中进行处理。
```

从整个流程不难看出，这几个接口最终对应的是HAL3的接口，并且Provider并没有经过太多复杂的额外的处理。

## 4、Camera HAL3 接口

HAL硬件抽象层(Hardware Abstraction Layer)，是谷歌开发的用于屏蔽底层硬件抽象出来的一个软件层， 每一个平台厂商可以将不开源的代码封装在这一层，仅仅提供二进制文件。
该层定义了自己的一套通用标准接口，平台厂商务必按照以下规则定义自己的Module:

```
每一个硬件模块都通过hw_module_t来描述，具有固定的名字HMI
每一个硬件模块都必须实现hw_module_t里面的open方法，
    用于打开硬件设备，并返回对应的操作接口集合
硬件的操作接口集合使用hw_device_t 来描述，
    并可以通过自定义一个更大的包含hw_device_t的结构体来拓展硬件操作集合
```

其中代表硬件模块的是hw_module_t，对应的设备是通过hw_device_t来描述，这两者的定义如下：
hw_module_t/hw_device_t源码如下：

```
typedef struct hw_module_t {
    uint32_t tag; //HMI
    struct hw_module_methods_t* methods;
} hw_module_t;
  
typedef struct hw_module_methods_t {
    int (*open)(const struct hw_module_t* module, const char* id, struct hw_device_t** device);
} hw_module_methods_t;
  
typedef struct hw_device_t {
    struct hw_module_t* module;
    int (*close)(struct hw_device_t* device);
} hw_device_t;
```

从上面的定义可以看出，主要是通过hw_module_t 代表了模块，通过其open方法用来打开一个设备，而该设备是用hw_device_t来表示，其中除了用来关闭设备的close方法外，并无其它方法，由此可见谷歌定义的HAL接口，并不能满足绝大部分HAL模块的需要，所以谷歌想出了一个比较好的解决方式，那便是将这两个基本结构嵌入到更大的结构体内部，同时在更大的结构内部定义了各自模块特有的方法，用于实现模块的功能，这样，一来对上保持了HAL的统一规范，二来也扩展了模块的功能。

基于上面的方式，谷歌便针对Camera 提出了HAL3接口，其中主要包括了用于代表一系列操作主体的结构体以及具体操作函数，接下来我们分别进行详细介绍：

1.核心结构体解析
HAL3中主要定义了camera_module_t/camera3_device_t/camera3_stream_configuration/camera3_stream以及camera3_stream_buffer几个主要结构体。

其中camera_module_t以及camera3_device_t代码定义如下：

```
typedef struct camera_module {
    hw_module_t common;
    int (*get_number_of_cameras)(void);
    int (*get_camera_info)(int camera_id, struct camera_info *info);
    nt (*set_callbacks)(const camera_module_callbacks_t *callbacks);
    void (*get_vendor_tag_ops)(vendor_tag_ops_t* ops);
    int (*open_legacy)(const struct hw_module_t* module, const char* id, uint32_t halVersion, struct hw_device_t** device);
    int (*set_torch_mode)(const char* camera_id, bool enabled);
    int (*init)();
    int (*get_physical_camera_info)(int physical_camera_id, int (*is_stream_combination_supported)(int camera_id,const camera_stream_combination_t *streams);
    int (*is_stream_combination_supported)(int camera_id, const camera_stream_combination_t *streams);
    void (*notify_device_state_change)(uint64_t deviceState);
} camera_module_t;
  
 
typedef struct camera3_device {
    hw_device_t common;
    camera3_device_ops_t *ops; //拓展接口，Camera HAL3定义的标准接口
    void *priv;
} camera3_device_t;
```

由定义不难发现，camera_module_t包含了hw_module_t，主要用于表示Camera模块，其中定义了诸如get_number_of_cameras以及set_callbacks等扩展方法，而camera3_device_t包含了hw_device_t，主要用来表示Camera设备，其中定义了camera3_device_ops操作方法集合，用来实现正常获取图像数据以及控制Camera的功能。

结构体camera3_stream_configuration代码定义如下：

```
typedef struct camera3_stream_configuration {
    uint32_t num_streams;
    camera3_stream_t **streams;
    uint32_t operation_mode;
    const camera_metadata_t *session_parameters;
} camera3_stream_configuration_t;
```

该结构体主要用来代表配置的数据流列表，内部装有上层需要进行配置的数据流的指针，内部的定义简单介绍下：

```
um_streams: 代表了来自上层的数据流的数量，其中包括了output以及input stream。
streams: 是streams的指针数组，包括了至少一条output stream
     以及至多一条input stream。
operation_mode: 当前数据流的操作模式，该模式在camera3_stream_configuration_mode_t中被定义，
     HAL通过这个参数可以针对streams做不同的设置。
session_parameters: 该参数可以作为缺省参数，直接设置为NULL即可，
     CAMERA_DEVICE_API_VERSION_3_5以上的版本才支持。
```

结构体camera3_stream_t的代码定义如下：

```
typedef struct camera3_stream {
    int stream_type;
    uint32_t width;
    uint32_t height;
    int format;
    uint32_t usage;
    uint32_t max_buffers;
    void *priv;
    android_dataspace_t data_space;
    int rotation;
    const char* physical_camera_id;
}camera3_stream_t;
```

该结构体主要用来代表具体的数据流实体，在整个的配置过程中，需要在上层进行填充，当下发到HAL中后，HAL会针对其中的各项属性进行配置，这里便简单介绍下其内部的各个元素的意义：

```
stream_type: 表示数据流的类型，类型在camera3_stream_type_t中被定义。
width： 表示当前数据流中的buffer的宽度。
height: 表示当前数据流中buffer的高度。
format: 表示当前数据流中buffer的格式，
        该格式是在system/core/include/system/graphics.h中被定义。
usage： 表示当前数据流的gralloc用法，其用法定义在gralloc.h中。
max_buffers： 指定了当前数据流中可能支持的最大数据buffer数量。
data_space: 指定了当前数据流buffer中存储的图像数据的颜色空间。
rotation：指定了当前数据流的输出buffer的旋转角度，
          其角度的定义在camera3_stream_rotation_t中，该参数由Camera Service进行设置，
          必须在HAL中进行设置，该参数对于input stream并没有效果。
physical_camera_id： 指定了当前数据流从属的物理camera Id。
```

结构体camera3_stream_buffer_t定义如下：

```
typedef struct camera3_stream_buffer {
    camera3_stream_t *stream;
    buffer_handle_t *buffer
    int status;
    int acquire_fence;
    int release_fence;
} camera3_stream_buffer_t;
```

该结构体主要用来代表具体的buffer对象，其中重要元素如下：

```
stream: 代表了从属的数据流
buffer：buffer句柄
```

2.核心接口函数解析

HAL3的核心接口都是在camera3_device_ops中被定义，代码定义如下：

```
typedef struct camera3_device_ops {
    int (*initialize)(const struct camera3_device *, const camera3_callback_ops_t *callback_ops);
    int (*configure_streams)(const struct camera3_device *, camera3_stream_configuration_t *stream_list);
    int (*register_stream_buffers)(const struct camera3_device *,const camera3_stream_buffer_set_t *buffer_set);
    const camera_metadata_t* (*construct_default_request_settings)(const struct camera3_device *, int type);
    int (*process_capture_request)(const struct camera3_device *, camera3_capture_request_t *request);
    void (*get_metadata_vendor_tag_ops)(const struct camera3_device*, vendor_tag_query_ops_t* ops);
    void (*dump)(const struct camera3_device *, int fd);
    int (*flush)(const struct camera3_device *);
    void (*signal_stream_flush)(const struct camera3_device*, uint32_t num_streams, const camera3_stream_t* const* streams);
    int (*is_reconfiguration_required)(const struct camera3_device*, const camera_metadata_t* old_session_params, const camera_metadata_t* new_session_params);
} camera3_device_ops_t;
```

从代码中可以看见，该结构体定义了一系列的函数指针，用来指向平台厂商实际的实现方法，接下来就其中几个方法简单介绍下：

a) initialize
该方法必须在camera_module_t中的open方法之后，其它camera3_device_ops中方法之前被调用，主要用来将上层实现的回调方法注册到HAL中，并且根据需要在该方法中加入自定义的一些初始化操作，另外，谷歌针对该方法在性能方面也有严格的限制，该方法需要在5ms内返回，最长不能超过10ms。

b) configure_streams
该方法在完成initialize方法之后，在调用process_capture_request方法之前被调用，主要用于重设当前正在运行的Pipeline以及设置新的输入输出流，其中它会将stream_list中的新的数据流替换之前配置的数据流。在调用该方法之前必须确保没有新的request下发并且当前request的动作已经完成，否则会引起无法预测的错误。一旦HAL调用了该方法，则必须在内部配置好满足当前数据流配置的帧率，确保这个流程的运行的顺畅性。
其中包含了两个参数，分别是camera3_device以及stream_list(camera3_stream_configuration_t ),其中第二个参数是上层传入的数据流配置列表，该列表中必须包含至少一个output stream，同时至多包含一个input stream。
另外，谷歌针对该方法有着严格的性能要求，平台厂商在实现该方法的时候，需要在500ms内返回，最长不能超过1000ms。

c) construct_default_request_settings
该方法主要用于构建一系列默认的Camera Usecase的capture 设置项，通过camera_metadata_t来进行描述，其中返回值是一个camera_metadata_t指针，其指向的内存地址是由HAL来进行维护的，同样地，该方法需要在1ms内返回，最长不能超过5ms。

d) process_capture_request
该方法用于下发单次新的capture request到HAL中， 上层必须保证该方法的调用都是在一个线程中完成，而且该方法是异步的，同时其结果并不是通过返回值给到上层，而是通过HAL调用另一个接口process_capture_result()来将结果返回给上层的，在使用的过程中，通过in-flight机制，保证短时间内下发足够多的request，从而满足帧率要求。

该方法的性能依然受到谷歌的严格要求，规定其需要在一帧图像处理完的时长内返回，最长不超过4帧图像处理完成的时长，比如当前预览帧率是30帧，则该方法的操作耗时最长不能超过120ms，否则便会引起明显的帧抖动，从而影响用户体验。

e) dump
该方法用于打印当前Camera设备的状态，一般是由上层通过dumpsys工具输出debug dump信息或者主动抓取bugreport的时候被调用，该方法必须是非阻塞实现，同时需要保证在1ms内返回，最长不能超过10ms。

f) flush
当上层需要执行新的configure_streams的时候，需要调用该方法去尽可能快地清除掉当前已经在处理中的或者即将处理的任务，为配置数据流提供一个相对稳定的环境，其具体工作如下：

```
所有的还在流转的request会尽可能快的返回
并未开始进行流转的request会直接返回，并携带错误信息
任何可以打断的硬件操作会立即被停止
任何无法进行打断的硬件操作会在当前状态下进行休眠
```

flush会在所有的buffer都得以释放，所有request都成功返回后才真正返回，该方法需要在100ms内返回，最长不能超过1000ms。

上面的一系列方法是上层直接对下控制Camera Hal，而一旦Camera Hal产生了数据或者事件的时候，可以通过camera3_callback_ops中定义的回调方法将数据或者事件返回至上层，该结构体定义如下：

```
typedef struct camera3_callback_ops {
  
    void (*process_capture_result)(const struct camera3_callback_ops *, const camera3_capture_result_t *result);
    void (*notify)(const struct camera3_callback_ops *, const camera3_notify_msg_t *msg);
  
    camera3_buffer_request_status_t (*request_stream_buffers)(
        const struct camera3_callback_ops *,
        uint32_t num_buffer_reqs,
        const camera3_buffer_request_t *buffer_reqs,
        /*out*/uint32_t *num_returned_buf_reqs,
        /*out*/camera3_stream_buffer_ret_t *returned_buf_reqs);
  
    void (*return_stream_buffers)( const struct camera3_callback_ops *, uint32_t num_buffers, const camera3_stream_buffer_t* const* buffers);
} camera3_callback_ops_t;
```

其中常用的回调方法主要有两个：用于返回数据的process_capture_result以及用于返回事件的notify，接下来分别介绍下：

a) process_capture_result
该方法用于返回HAL部分产生的metadata和image buffers，它与request是多对一的关系，同一个request，可能会对应到多个result，比如可以通过调用一次该方法用于返回metadata以及低分辨率的图像数据，再调用一次该方法用于返回jpeg格式的拍照数据，而这两次调用时对应于同一个process_capture_request动作。

同一个Request的Metadata以及Image Buffers的先后顺序无关紧要，但是同一个数据流的不同Request之间的Result必须严格按照Request的下发先后顺序进行依次返回的，如若不然，会导致图像数据显示出现顺序错乱的情况。

该方法是非阻塞的，而且并且必须要在5ms内返回。

b) notify
该方法用于异步返回HAL事件到上层，必须非阻塞实现，而且要在5ms内返回。

谷歌为了将系统框架和平台厂商的自定义部分相分离，在Android上推出了Treble项目，该项目直接将平台厂商的实现部分放入vendor分区中进行管理，进而与system分区保持隔离，这样便可以在相互独立的空间中进行各自的迭代升级，而互不干扰，而在相机框架体系中，便将Camera HAL Module从Camera Service中解耦出来，放入独立进程Camera Provider中进行管理，而为了更好的进行跨进程访问，谷歌针对Provider提出了HIDL机制用于Camera Servic对于Camera Provier的访问，而HIDL接口的实现是在Camera Provider中实现，针对Camera HAL Module的控制又是通过谷歌制定的Camera HAL3接口来完成，所以由此看来，Provider的职责也比较简单，通过HIDL机制保持与Camera Service的通信，通过HAL3接口控制着Camera HAL Module。



# 六、硬件抽象层实现

## 1、概览

回顾高通平台Camera HAL历史，之前高通采用的是QCamera & MM-Camera架构，但是为了更精细化控制底层硬件(Sensor/ISP等关键硬件)，同时方便手机厂商自定义一些功能，现在提出了CamX-CHI架构，由于在CamX-CHI中完全看不到之前老架构的影子，所以它完全是一个全新的架构，它将一些高度统一的功能性接口抽离出来放到CamX中，将可定制化的部分放在CHI中供不同厂商进行修改，实现各自独有的特色功能，这样设计的好处显而易见，那便是即便开发者对于CamX并不是很了解，但是依然可以很方便的加入自定义的功能，从而降低了开发者在高通平台的开发门槛。

接下来我们以最直观的目录结构入手对该架构做一个简单的认识，以下便是CamX-CHI基本目录结构：

![CamX-CHI](assets/CamX-CHI.png)

该部分代码主要位于 vendor/qcom/proprietary/ 目录下：
其中 camx 代表了通用功能性接口的代码实现集合（CamX），chi-cdk代表了可定制化需求的代码实现集合（CHI），从图中可以看出Camx部分对上作为HAL3接口的实现，对下

通过v4l2框架与Kernel保持通讯，中间通过互相dlopen so库并获取对方操作接口的方式保持着与CHI的交互。

camx/中有如下几个主要目录：

```
core/ ： 用于存放camx的核心实现模块，其中还包含了主要用于实现hal3接口的hal/目录，以及负责与CHI进行交互的chi/目录
csl/： 用于存放主要负责camx与camera driver的通讯模块，为camx提供了统一的Camera driver控制接口
hwl/: 用于存放自身具有独立运算能力的硬件node，该部分node受csl管理
swl/: 用于存放自身并不具有独立运算能力，必须依靠CPU才能实现的node
```

chi-cdk/中有如下几个主要目录：

```
chioverride/: 用于存放CHI实现的核心模块，负责与camx进行交互并且实现了CHI的总体框架以及具体的业务处理。
bin/: 用于存放平台相关的配置项
topology/: 用于存放用户自定的Usecase xml配置文件
node/: 用于存放用户自定义功能的node
module/: 用于存放不同sensor的配置文件，该部分在初始化sensor的时候需要用到
tuning/: 用于存放不同场景下的效果参数的配置文件
sensor/: 用于存放不同sensor的私有信息以及寄存器配置参数
actuator/: 用于存放不同对焦模块的配置信息
ois/： 用于存放防抖模块的配置信息
flash/： 存放着闪光灯模块的配置信息
eeprom/: 存放着eeprom外部存储模块的配置信息
fd/: 存放了人脸识别模块的配置信息
```

## 2、基本组件概念

1、Usecase
作为CamX-CHI中最大的抽象概念，其中包含了多条实现特定功能的Pipeline，具体实现是在CHI中通过Usecase类完成的，该类主要负责了其中的业务处理以及资源的管理。

Usecase类，提供了一系列通用接口，作为现有的所有Usecase的基类，其中，AdvancedCameraUsecase又继承于CameraUsecaseBase，相机中绝大部分场景会通过实例化AdvancedCameraUsecase来完成，它包括了几个主要接口：

```
Create(): 该方法是静态方法，用于创建一个AdvancedCameraUsecase实例，
      在其构造方法中会去获取XML中的相应的Usecase配置信息。
ExecuteCaptureRequest(): 该方法用于下发一次Request请求。
ProcessResultCb(): 该方法会在创建Session的过程中，
      作为回调方法注册到其中，一旦Session数据处理完成的时候便会
      调用该方法将结果发送到AdvancedCameraUsecase中。
ProcessDriverPartialCaptureResult(): 该方法会在创建Session的过程中，
      作为回调方法注册到其中，一旦Session中产生了partial meta data的时候，
      便会调用该方法将其发送至AdvancedCameraUsecase中。
ProcessMessageCb(): 该方法会在创建Session的过程中，作为回调方法注册到其中，
      一旦Session产生任何事件，便会调用该方法通知到AdvancedCameraUsecase中。
ExecuteFlush(): 该方法用于刷新AdvancedCameraUsecase。
Destroy(): 该方法用于安全销毁AdvancedCameraUsecase。
```

Usecase的可定制化部分被抽象出来放在了common_usecase.xml文件中，这里简单介绍其中的几个主要的标签含义：
Usecase

```
UsecaseName: 代表了该Usecase的名字，后期根据这个名字找到这个Usecase的定义。
Targets: 用于表示用于输出的数据流的集合，其中包括了数据流的格式，输出Size的范围等。
Pipeline: 用于定义该Usecase可以是使用的所有Pipeline，这里必须至少定义一条Pipeline。
```

2、Feature
代表了一个特定的功能，该功能需要多条Pipeline组合起来实现，受Usecase统一管理，在CHI中通过Feature类进行实现，在XML中没有对应的定义，具体的Feature选取工作是在Usecase中完成的，通过在创建Feature的时候，传入Usecase的实例的方式，来和Usecase进行相互访问各自的资源。

以下是现有的Feature，其中Feature作为基类存在，定义了一系列通用方法。

![feature](assets/feature.png)

几个常用的Feature:

```
FeatureHDR: 用于实现HDR功能，它负责管理内部的一条或者几条pipeline的资源以及它们的流转，
     最终输出具有HDR效果的图像。
FeatureMFNR: 用于实现MFNR功能，内部分为几个大的流程，分别包括
     Prefiltering、Blending、Postfilter以及最终的OfflineNoiseReproces
     (这一个是可选择使能的)，每一个小功能中包含了各自的pipeline。
FeatureASD: 用于AI功能的实现，在预览的时候，接收每一帧数据，
     并且进行分析当前场景的AI识别输出结果，并其通过诸如到metadata方式给到上层，进行后续的处理。
```

3、Session
用于管理pipeline的抽象控制单元，一个Session中至少拥有一个pipeine，并且控制着所有的硬件资源，管控着每一个内部pipeline的request的流转以及数据的输入输出，它没有可定制化的部分，所以在CHI中的XML文件中并没有将Session作为一个独立的单元进行定义。

Session的实现主要通过CamX中的Session类，其主要接口如下：

```
Initialize(): 根据传入的参数SessionCreateData进行Session的初始化工作。
NotifyResult(): 内部的Pipeline通过该接口将结果发送到Session中。
ProcessCaptureRequest(): 该方法用于用户决定发送一个Request到Session中的时候调用。
StreamOn(): 通过传入的Pipeline句柄，开始硬件的数据传输。
StreamOff(): 通过传入的Pipeline句柄，停止硬件的数据传输。
```

4、Pipeline
作为提供单一特定功能的所有资源的集合，维护着所有硬件资源以及数据的流转，每一个Pipeline包括了其中的Node/Link，在CamX中通过Pipeline类进行实现，负责整条Pipeline的软硬件资源的维护以及业务逻辑的处理，接下来我们简单看下该类的几个主要接口：

```
Create(): 该方法是一个静态方法，根据传入的PipelineCreateInputData信息来实例化一个Pipeline对象。
StreamOn(): 通知Pipeline开始硬件的数据传输
StreamOff(): 通知Pipeline停止硬件的数据传输
FinalizePipeline(): 用于完成Pipeline的设置工作
OpenRequest(): open一个CSL用于流转的Request
ProcessRequest(): 开始下发Request
NotifyNodeMetadataDone(): 该方法是Pipeline提供给Node，当Node内部生成了metadata,便会调用该方法来通知metadata已经完成，最后当所有Node都通知Pipeline metadata已经完成，Pipeline 便会调用ProcessMetadataRequestIdDone通知Session。
NotifyNodePartialMetadataDone(): 该方法是Pipeline提供给Node，当Node内部生成了partial metadata,便会调用该方法来通知metadata已经完成，最后当所有Node都通知Pipeline metadata已经完成，Pipeline 便会调用ProcessPartialMetadataRequestIdDone通知Session。
SinkPortFenceSignaled(): 用来通知Session 某个sink port的fence处于被触发的状态。
NonSinkPortFenceSignaled(): 用来通知Session 某个non sink port的fence处于被触发的状态。
Pipeline中的Node以及连接方式都在XML中被定义，其主要包含了以下几个标签定义：
 
PipelineName: 用来定义该条Pipeline的名称
NodeList: 该标签中定义了该条Pipeline的所有的Node
PortLinkages: 该标签定义了Node上不同端口之间的连接关系
```

5、Node
作为单个具有独立处理功能的抽象模块，可以是硬件单元也可以是软件单元，关于Node的具体实现是CamX中的Node类来完成的，其中CamX-CHI中主要分为两个大类，一个是高通自己实现的Node包括硬件Node，一个是CHI中提供给用户进行实现的Node，其主要方法如下：

```
Create(): 该方法是静态方法，用于实例化一个Node对象。
ExecuteProcessRequest(): 该方法用于针对hwl node下发request的操作。
ProcessRequestIdDone(): 一旦该Node当前request已经处理完成，便会通过调用该方法通知Pipeline。
ProcessMetadataDone(): 一旦该Node的当前request的metadata已经生成，便会通过调用该方法通知到Pipeline。
ProcessPartialMetadataDone(): 一旦该Node的当前request的partial metadata已经生成，便会通过调用该方法通知到Pipeline。
CreateImageBufferManager(): 创建ImageBufferManager
其可定制化的部分作为标签在XML中进行定义：
 
NodeName： 用来定义该Node的名称
NodeId: 用来指定该Node的ID，其中IPE NodeId为65538，IFE NodeId为65536，用户自定义的NodeId为255。
NodeInstance: 用于定义该Node的当前实例的名称。
NodeInstanceId: 用于指定该Node实例的Id。
```

7、Link
用于定义不同Port的连接，一个Port可以根据需要建立多条与其它从属于不同Node的Port的连接,它通过标签来进行定义，其中包括了作为输入端口，作为输出端口。
一个Link中包含了一个SrcPort和一个DstPort，分别代表了输入端口和输出端口，然后BufferProperties用于表示两个端口之间的buffer配置。

8、Port
作为Node的输入输出的端口，在XML文件中，标签用来定义一个输入端口，标签用来定义输出端口，每一个Node都可以根据需要使用一个或者多个输入输出端口，使用OutputPort以及InputPort结构体来进行在代码中定义。

```
PortId: 该端口的Id: 该端口的名称
NodeName: 该端口从属的Node名称
NodeId: 该端口从属的Node的Id
NodeInstance: 该端口从属的Node的实例名称
NodeInstanceId: 该端口从属的Node的实例的Id
```

## 3、组件结构关系

通过之前的介绍，我们对于几个基本组件有了一个比较清晰地认识，但是任何一个框架体系并不是仅靠组件胡乱堆砌而成的，相反，它们都必须基于各自的定位，按照各自所独有的行为模式，同时按照约定俗称的一系列规则组合起来，共同完成整个框架某一特定的功能。所以这里不得不产生一个疑问，在该框架中它们到底是如何组织起来的呢？它们之间的关系又是如何的呢？ 接下来我们以下图入手开始进行分析：

![component-structure](assets/component-structure.png)

由上图可以看到，几者是通过包含关系组合起来的，Usecase 包含Feature，而Feature包含了Session，Session又维护了内部的Pipeline的流转，而每一条pipeline中又通过Link将所有Node都连接了起来，接下我们就这几种关系详细讲解下：

首先，一个Usecase代表了某个特定的图像采集场景，比如人像场景，后置拍照场景等等，在初始化的时候通过根据上层传入的一些具体信息来进行创建，这个过程中，一方面实例化了特定的Usecase，这个实例是用来管理整个场景的所有资源，同时也负责了其中的业务处理逻辑，另一方面，获取了定义在XML中的特定Usecase，获取了用于实现某些特定功能的pipeline。

其次，在Usecase中，Feature是一个可选项，如果当前用户选择了HDR模式或者需要在Zoom下进行拍照等特殊功能的话，在Usecase创建过程中，便会根据需要创建一个或者多个Feature，一般一个Feature对应着一个特定的功能，如果场景中并不需要任何特定的功能，则也完全可以不使用也不创建任何Feature。

然后，每一个Usecase或者Feature都可以包含一个或者多个Session，每一个Session都是直接管理并负责了内部的Pipeline的数据流转，其中每一次的Request都是Usecase或者Featuret通过Session下发到内部的Pipeline进行处理，数据处理完成之后也是通过Session的方法将结果给到CHI中，之后是直接给到上层还是将数据封装下再次下发到另一个Session中进行后处理，这都交由CHI来决定。

其中，Session和Pipeline是一对多的关系，通常一个Session只包含了一条Pipeline，用于某个特定图像处理功能的实现，但是也不绝对，比如FeatureMFNR中包含的Session就包括了三条pipeline，又比如后置人像预览，也是用一个Session包含了两条分别用于主副双摄预览的Pipeline，主要是要看当前功能需要的pipeline数量以及它们之间是否存在一定关联。

同时，根据上面关于Pipeline的定义，它内部包含了一定数量的Node，并且实现的功能越复杂，所包含的Node也就越多，同时Node之间的连接也就越错综复杂，比如后置人像预览虚化效果的实现就是将拿到的主副双摄的图像通过RTBOfflinePreview这一条Pipeline将两帧图像合成一帧具有虚化效果的图像，从而完成了虚化功能。

最后Pipeline中的Node的连接方式是通过XML文件中的Link来进行描述的，每一个Link定义了一个输入端和输出端分别对应着不同Node上面的输入输出端口，通过这种方式就将其中的一个Node的输出端与另外一个Node的输入端，一个一个串联起来，等到图像数据从Pipeline的起始端开始输入的时候，便可以按照这种定义好的轨迹在一个一个Node之间进行流转，而在流转的过程中每经过一个Node都会在内部对数据进行处理，这样等到数据从起始端一直流转到最后一个Node的输出端的时候，数据就经过了很多次处理，这些处理效果最后叠加在一起便是该Pipeline所要实现的功能，比如降噪、虚化等等。

## 4、关键流程详解

**1、Camera Provider 启动初始化**
当系统启动的时候，Camera Provider主程序会被运行，在整个程序初始化的过程中会通过获取到的camera_module_t调用其get_number_of_camera接口获取底层支持的camera数量，由于是第一次获取，所以在CamX-CHI中会伴随着很多初始化动作，具体操作见下图：

![camera-provider-init](assets/camera-provider-init.png)

主要流程如下：

```
通过HAL3Module::GetInstance()静态方法实例化了HAL3Module对象，
   在其构造方法里面通过HwEnvironment::GetInstance()静态方法又实例化了HwEnvironment对象，
   在其构造方法中，实例化了SettingsManager对象，然后又在它构造方法中通过
   OverrideSettingsFile对象获取了位于/vendor/etc/camera/camoverridesettings.txt
   文件中的平台相关的配置信息（通过这种Override机制方便平台厂商加入自定义配置），
   该配置文件中，可以加入平台特定的配置项，比如可以通过设置multiCameraEnable的值
   来表示当前平台是否支持多摄，或者通过设置overrideLogLevels设置项来配置
   CamX-CHI部分的Log输出等级等等。
 
同时在HwEnvironment构造方法中会调用其Initialize方法，在该方法中实例化了CSLModeManager对象，
   并通过CSLModeManager提供的接口，获取了所有底层支持的硬件设备信息，
   其中包括了Camera Request Manager、CAPS模块（该驱动模块主要用于CSL获取Camera平台驱动信息，
   以及IPE/BPS模块的电源控制）以及Sensor/IPE/Flash等硬件模块，并且通过调用 
   CSLHwInternalProbeSensorHW方法获取了当前设备安装的Sensor模组信息，
   并且将获取的信息暂存起来，等待后续阶段使用，总得来说在HwEnvironment初始化的过程中,
   通过探测方法获取了所有底层的硬件驱动模块，并将其信息存储下来供后续阶段使用。
```

之后通过调用HwEnvironment对象中的ProbeChiCompoents方法在/vendor/lib64/camera/components路径下找寻各个Node生成的So库，并获取Node提供的标准对外接口，这些Node不但包括CHI部分用户自定义的模块，还包括了CamX部分实现的硬件模块，并最后都将其都存入ExternalComponentInfo对象中，等待后续阶段使用。
另外在初始化阶段还有一个比较重要的操作就是CamX 与CHI是通过互相dlopen对方的So库，获取了对方的入口方法，最后通过彼此的入口方法获取了对方操作方法集合，之后再通过这些操作方法与对方进行通讯，其主要流程见下图：

![CamX-CHI-interact](assets/CamX-CHI-interact.png)

从上图不难看出，在HAL3Module构造方法中会去通过dlopen方法加载com.qti.chi.override.so库，并通过dlsym映射出CHI部分的入口方法chi_hal_override_entry，并调用该方法将HAL3Module对像中的成员变量m_ChiAppCallbacks(CHIAppCallbacks)传入CHI中，其中包含了很多函数指针，这些函数指针分别对应着CHI部分的操作方法集中的方法，一旦进入到CHI中，就会将CHI本地的操作方法集合中的函数地址依次赋值给m_ChiAppCallbacks，这样CamX后续就可以通过这个成员变量调用到CHI中方法，从而保持了与CHI的通讯。

同样地，CHI中的ExtensionModule在初始化的时候，其构造方法中也会通过调用dlopen方法加载camera.qcom.so库，并将其入口方法ChiEntry通过dlsym映射出来，之后调用该方法，将g_chiContextOps（ChiContextOps，该结构体中定义了很多指针函数）作为参数传入CamX中，一旦进入CamX中，便会将本地的操作方法地址依次赋值给g_chiContextOps中的每一个函数指针，这样CHI之后就可以通过g_chiContextOps访问到CamX方法。

**2、打开相机设备/初始化相机设备**
一旦用户打开了相机应用，App中便会去调用CameraManager的openCamera方法，该方法之后会最终调用到Camera Service中的CameraService::connectDevice方法，然后通过ICameraDevice::open()这一个HIDL接口通知Provider，然后在Provider内部又通过调用之前获取的camera_module_t中methods的open方法来获取一个Camera 设备，对应于HAL中的camera3_device_t结构体，紧接着，在Provider中会继续调用获取到的camera3_device_t的initialize方法进行初始化动作。接下来我们便来详细分析下CamX-CHI对于open以及initialize的具体实现流程：

a) open

该方法是camera_module_t的标准方法，主要用来获取camera3_device_t设备结构体的，CamX-CHI对其进行了实现，open方法中完成的工作主要有以下几个：

```
将当前camera id传入CHI中进行remap操作，当然这个remap操作逻辑完全是根据CHI中用户需求来的，
    用户可以根据自己的需要在CHI中加入自定义remap逻辑。
实例化HALDevice对象，其构造函数中调用Initialize方法，
    该方法会填充CamX中自定义的Camera3Device结构体。
将m_HALCallbacks.process_capture_result指向了本地方法ProcessCaptureResult
    以及m_HALCallbacks.notify_result指向了本地方法Notify
   （之后会在配置数据流的过程中，将m_HALCallbacks注册到CHI中， 一旦当CHI数据处理完成之后，
     便会通过这两个回调方法将数据或者事件回传给CamX）。
最后将HALDevice 中的Camera3Device成员变量作为返回值给到Provider中的CameraCaptureSession中。
```

Camera3Device 其实重定义了camera3_device_t，其中HwDevice对应于camera3_device_t中的hw_device_t，Camera3DeviceOps对应于camera3_device_ops_t，而在HALDevice的初始化过程中，会将CamX实现的HAL3接口的结构体g_camera3DeviceOps赋值给Camera3DeviceOps中。

b) initialize

该方法在调用open后紧接着被调用，主要用于将上层的回调接口传入HAL中，一旦有数据或者事件产生，CamX便会通过这些回调接口将数据或者事件上传至调用者，其内部的实现较为简单。

initialize方法中有两个参数，分别是之前通过open方法获取的camera3_device_t结构体和实现了camera3_callback_ops_t的CameraDevice，很显然camera3_device_t结构体并不是重点，所以该方法的主要工作是将camera3_callback_ops_t与CamX关联上，一旦数据准备完成便通过这里camera3_callback_ops_t中回调方法将数据回传到Camera Provider中的CameraDevice中，基本流程可以总结为以下几点：

```
实例化了一个Camera3CbOpsRedirect对象并将其加入了g_HAL3Entry.m_cbOpsList队列中，
    这样方便之后需要的时候能够顺利拿到该对象。
将本地的process_capture_result以及notify方法地址分别赋值给
    Camera3CbOpsRedirect.cbOps中的process_capture_result以及notify函数指针。
将上层传入的回调方法结构体指针pCamera3CbOpsAPI赋值给Camera3CbOpsRedirect.pCbOpsAPI，
    并将Camera3CbOpsRedirect.cbOps赋值给pCamera3CbOpsAPI，
    通过JumpTableHal3的initialize方法将pCamera3CbOpsAPI传给HALDevice中的
    m_pCamera3CbOps成员变量，这样HALDevice中的m_pCamera3CbOps就指向了
    CamX中本地方法process_capture_result以及notify。
```

经过这样的一番操作之后，一旦CHI有数据传入便会首先进入到本地方法ProcessCaptureResult，然后在该方法中获取到HALDevice的成员变量m_pCamera3CbOps，进而调用m_pCamera3CbOps中的process_capture_result方法，即camxhal3entry.cpp中定义的process_capture_result方法，然后这个方法中会去调用JumpTableHAL3.process_capture_result方法，该方法最终会去调用Camera3CbOpsRedirect.pCbOpsAPI中的process_capture_result方法，这样就调到从Provider传入的回调方法，将数据顺利给到了CameraCaptureSession中。

3、配置相机设备数据流
在打开相机应用过程中，App在获取并打开相机设备之后，会调用CameraDevice.createCaptureSession来获取CameraDeviceSession，并且通过Camera api v2标准接口，通知Camera Service，调用其CameraDeviceClient.endConfigure方法，在该方法内部又会去通过HIDL接口ICameraDeviceSession::configureStreams_3_4通知Provider开始处理此次配置需求，在Provider内部，会去通过在调用open流程中获取的camera3_device_t结构体的configure_streams方法来将数据流的配置传入CamX-CHI中，之后由CamX-CHI完成对数据流的配置工作，接下来我们来详细分析下CamX-CHI对于该标准HAL3接口 configure_streams的具体实现：

配置数据流是整个CamX-CHI流程比较重要的一环，其中主要包括两个阶段：

```
选择UsecaseId
根据选择的UsecaseId创建Usecase
```

接下来我们就这两个阶段分别进行详细介绍:

① 选择UsecaseId

不同的UsecaseId分别对应的不同的应用场景，该阶段是通过调用UsecaseSelector::GetMatchingUsecase()方法来实现的，该函数中通过传入的operation_mode、num_streams配置数据流数量以及当前使用的Sensor个数来选择相应的UsecaseId，比如当numPhysicalCameras值大于1同时配置的数据流数量num_streams大于1时选择的就是UsecaseId::MultiCamera，表示当前采用的是双摄场景。

② 创建Usecase
根据之前选择的UsecaseId，通过UsecaseFactory来创建相应的Usecase，
其中Class Usecase是所有Usecase的基类，其中定义并实现了一些通用接口，CameraUsecaseBase继承于Usecase，并扩展了部分功能。AdvancedCameraUsecase又继承于CameraUsecaseBase，作为主要负责大部分场景的Usecase实现类，另外对于多摄场景，现提供了继承于AdvancedCameraUsecase的UsecaseMultiCamera来负责实现。

除了双摄场景，其它大部分场景使用的都是AdvancedCameraUsecase类来管理各项资源的，接下来我们重点梳理下AdvancedCameraUsecase::Create()方法。
在AdvancedCameraUsecase::Create方法中做了很多初始化操作，其中包括了以下几个阶段：

```
获取XML文件中Usecase配置信息
创建Feature
保存数据流，重建Usecase的配置信息
调用父类CameraUsecaseBase的initialize方法，进行一些常规初始化工作
```

接下来我们就这几个阶段逐一进行分析：

1.获取XML文件中Usecase配置信息
这一部分主要通过调用CameraUsecaseBase::GetXMLUsecaseByName方法进行实现。
该方法的主要操作是从PerNumTargetUsecases数组中找到匹配到给定的usecaseName的Usecase，并作为返回值返回给调用者，其中这里我们以"UsecaseZSL“为例进行分析，PerNumTargetUsecases的定义是在g_pipeline.h中，该文件是在编译过程中通过usecaseconverter.pl脚本将定义在个平台目录下的common_usecase.xml中的内容转换生成g_pipeline.h。

2.创建Feature
如果当前场景选取了Feature，则调用FeatureSetup来完成创建工作。
该方法主要是通过诸如operation_mode、camera数量以及UsecaseId等信息来决定需要选择哪些Feature,具体逻辑比较清晰，一旦决定需要使用哪一个Feature之后，便调用相应的Feature的Create()方法进行初始化操作。

3.保存数据流，重建Usecase的配置信息
从Camera Service 传入的数据流，需要将其存储下来，供后续使用，同时高通针对Usecase也加入了Override机制，根据需要可以选择性地扩展Usecase，这两个步骤的实现主要是通过SelectUsecaseConfig方法来实现。

其中主要是调用以下两个方法来实现的：

```
ConfigureStream： 该方法将从上层配置的数据流指针存入AdvancedCameraUsecase中，
    其中包括了用于预览的m_pPreviewStream以及用于拍照的m_pSnapshotStream。
BuildUsecase： 这个方法用来重新在原有的Usecase上面加入了Feature中所需要的pipeline，
    并创建了一个新的Usecase，并将其存入AdvancedCameraUsecase中的m_pChiUsecase成员变量中，
    紧接着通过SetPipelineToSessionMapping方法将pipeline与Session进行关联。
```

4.调用父类CameraUsecaseBase的initialize方法，进行一些常规初始化工作

该方法中的操作主要有以下三个：

```
设置Session回调
创建Pipeline
创建Session
```

设置Session回调

该方法有两个参数，第二个是缺省的，第一个是ChiCallBacks，该参数是作为创建的每一条Session的回调方法，当Session中的pipeline全部跑完之后，会回调该方法将数据投递到CHI中。

创建Pipeline
根据之前获取的pipeline信息开始创建每一条pipeline，通过调用CreatePipeline()方法实现。

创建Session
创建Session，通过CreateSession()方法实现，此时会将AdvancedCameraUsecase端的回调函数注册到Session中，一旦Session中数据处理完成，便会调用回调将数据回传给AdvancedCameraUsecase。

综上，整个configure_stream过程，基本可以概括为以下几点：

```
根据operation_mode、camera 个数以及stream的配置信息选取了对应的UsecaseId
根据所选取的UsecaseId，使用UsecaseFactory简单工厂类创建了用于管理整个场景
    下所有资源的AdvancedCameraUsecase对象。
创建AdvancedCameraUsecase对象是通过调用其Create()方法完成，该方法中获取了common_usecase.xml
    定义的关于Usecase的配置信息，之后又根据需要创建了Feature并选取了Feature所需的pipeline，
    并通过Override机制将Feature中所需要的Pipeline加入重建后的Usecase中。
最后通过调用CameraUsecaseBaese的initialize方法依次创建了各个pipeline以及Session，
    并且将AdvancedCameraUsecase的成员方法注册到Session，用于Session将数据返回给Usecase中
```

4.处理拍照请求
当用户打开相机应用进行预览或者点击一次拍照操作的时候，便触发了一次拍照请求，该动作首先通过CameraDeviceSession的capture或者setRepeatingRequest方法将请求通过Camera api v2接口下发到Camera Service中，然后在Camera Service内部将此次请求发送到CameraDevice::RequestThread线程中进行处理，一旦进入到该线程之后，便会最终通过HIDL接口ICameraCaptureSession:processCaptureRequest_3_4将请求发送至Provider中，之后当Provider收到请求之后，会调用camera3_device_t结构体的process_capture_request开始了HAL针对此次Request的处理，而该处理是由CamX-CHI来负责实现，现在我们就来看下CamX-CHI是如何实现该方法的：

首先CamX中会将此次request转发到HALDevice中，再通过HALDevice对象调用之前初始化的时候获取的CHI部分的回调接口m_ChiAppCallbacks.chi_override_process_request方法（chi_override_process_request方法的定义位于chxextensioninterface.cpp中）将request发送到CHI部分。

在chi_override_process_request方法中会去获取ExtensionModule对象，并将request发送到ExtensionModule对象中，该对象中存储了之前创建的Usecase对象，然后经过层层调用，最终会调用AdvancedCameraUsecase的ExecuteCaptureRequest方法，该方法负责处理此次Request，具体流程如下：

在AdvancedCameraUsecase的ExecuteCaptureRequest中会有两个主要的分支来分别处理：

```
如果当前并没有任何Feature需要实现，此时便会走默认流程，根据上面的流程图所示，
   这里会调用CameraUsecaseBase::ExecuteCaptureRequest方法，在该方法中，
   首先会将request取出，重新封装成CHICAPTUREREQUEST，
   然后调用CheckAndActivatePipeline方法唤醒pipeline,
   这一操作到最后会调到Session的StreamOn方法，在唤醒了pipeline之后，继续往下执行，
   再将封装后的Request发送到CamX中，最终调用到相应的Session::ProcessCaptureRequest方法，
   此时Request就进入到了Session内部进行流转了。
如果当前场景需要实现某个Feature，则直接调用Feature的ExecuteProcessRequest方法
   将此次request送入Feature中处理，最后依然会调用到Session::StreamOn
   以及Session::ProcessCaptureRequest方法来分别完成唤醒pipeline
   以及下发request的到Session的操作。
```

该流程最终都会调用到两个比较关键的方法Session::StreamOn以及Session::ProcessCaptureRequest，接下来针对这两个方法重点介绍下：

Session::StreamOn

从方法名称基本可以知道该方法主要用于开始硬件的数据输出，具体点儿就是进行配置Sensor寄存器，让其开始出图，并且将当前的Session的状态告知每一Node，让它们在自己内部也做好处理数据的准备，所以之后的相关Request的流转都是以该方法为前提进行的，所以该方法重要性可见一斑，其操作流程见下图：

Session的StreamOn方法中主要做了如下两个工作：

```
调用FinalizeDeferPipeline()方法，如果当前pipeline并未初始化，
    则会调用pipeline的FinalizePipeline方法，这里方法里面会去针对每一个从属于
    当前pipeline的Node依次做FinalizeInitialization、CreateBufferManager、
    NotifyPipelineCreated以及PrepareNodeStreamOn操作，FinalizeInitialization
    用于完成Node的初始化动作，NotifyPipelineCreated用于通知Node当前Pipeline的状态，
    此时Node内部可以根据自身的需要作相应的操作，PrepareNodeStreamOn方法的主要是完成
    Sensor以及IFE等Node的控制硬件模块出图前的配置，
    其中包括了曝光的参数的设置，CreateBufferManagers方法涉及到CamX-CHI中的一个
    非常重要的Buffer管理机制，用于Node的ImageBufferManager的创建，
    而该类用于管理Node中的output port的buffer申请/流转/释放等操作。
调用Pipeline的StreamOn方法，里面会进一步通知CSL部分开启数据流，
    并且调用每一个Node的OnNodeStreamOn方法，该方法会去调用ImageBufferManager的Activate(),
    该方法里面会去真正分配用于装载图像数据的buffer，
    之后会去调用CHI部分实现的用户自定义的Nod的pOnStreamOn方法，
    用户可以在该方法中做一些自定义的操作。
```

Session::ProcessCaptureRequest
针对每一次的Request的流转，都是以该方法为入口开始的，具体流程见下图：

![Session-ProcessCaptureRequest](assets/Session-ProcessCaptureRequest.png)

上述流程可以总结为以下几个步骤：

```
通过调用Session的ProcessCaptureRequest方法进入到Session，然后调用Pipeline中的
    ProcessRequest方法通知Pipeline开始处理此次Request。
在Pipeline中，会先去调用内部的每一个Node的SetupRequest方法分别设置该Node的Output Port
    以及Input Port，之后通过调用DRQ(DeferredRequestQueue)的AddDeferredNode方法将所有的
    Node加入到DRQ中，其中DRQ中有两个队列分别是用于保存没有依赖项的Node的m_readyNodes
    以及保存处于等待依赖关系满足的Node的m_deferredNodes，当调用DRQ的DispatchReadyNodes方法后，
    会开始从m_readyNodes队列中取出Node调用其ProcessRequest开始进入Node内部处理本次request，
    在处理过程中会更新meta data数据，并更新至DRQ中，当该Node处理完成之后，
    会将处于m_deferredNodes中的已无依赖关系的Node移到m_readyNodes中，
    并再次调用DispatchReadyNodes方法从m_readyNodes取出Node进行处理。
与此过程中，当Node的数据处理完成之后会通过CSLFenceCallback通知到Pipeline，
   此时Pipeline会判断当前Node的Output port 是否是Sink Port(输出到CHI)，如果不是，
   则会更新依赖项到DRQ中，并且将不存在依赖项的Node移到m_readyNodes队列中，
   然后调用DispatchReadyNdoes继续进入到DRQ中流转，如果是Sink Port，
   则表示此Node是整个Pipeline的最末端，调用sinkPortFenceSignaled将数据给到Session中，
   最后通过调用Session中的NotifyResult将结果发送到CHI中。
```

DeferredRequestQueue

上述流程里面中涉及到DeferredRequestQueue这个概念，这里简单介绍下：
DeferredRequestQueue继承于IPropertyPoolObserver，实现了OnPropertyUpdate/OnMetadataUpdate/OnPropertyFailure/OnMetadataFailure接口，这几个接口用于接收Meta Data以及Property的更新，另外，DRQ主要包含了以下几个主要方法:

```
Create()
该方法用于创建DRQ，其中创建了用于存储依赖信息的m_pDependencyMap，
    并将自己注册到MetadataPool中，一旦有meta data或者property更新便会通过类中实现的几个
    接口通知到DRQ。
 
DispatchReadyNodes()
该方法主要用于将处于m_readyNodes队列的Node取出，
   将其投递到m_hDeferredWorker线程中进行处理。
 
AddDeferredNode()
该方法主要用于添加依赖项到m_pDependencyMap中。
 
FenceSignaledCallback()
当Node内部针对某次request处理完成之后，会通过一系列回调通知到DRQ，
   而其调用的方法便是该方法，在该方法中，会首先调用UpdateDependency更新依赖项，
   然后调用DispatchReadyNodes触发开始对处于ready状态的Node开始进行处理
 
OnPropertyUpdate()
该方法是定义于IPropertyPoolObserver接口，DRQ实现了它，主要用于接收Property更新的通知，
    并在内部调用UpdateDependency更新依赖项。
 
OnMetadataUpdate()
该方法是定义于IPropertyPoolObserver接口，DRQ实现了它，主要用于接收Meta data更新的通知，
    并在内部调用UpdateDependency更新依赖项。
 
UpdateDependency()
该方法用于更新Node的依赖项信息，并且将没有依赖的Node从m_deferredNodes队列中
    移到m_readyNodes，这样该Node就可以在之后的某次DispatchReadyNodes调用之后投入运行。
 
DeferredWorkerWrapper()
该方法是m_hDeferredWorker线程的处理函数，主要用于处理需要下发request的Node，
    同时再次更新依赖项，最后会再次调用DispatchReadyNodes开始处理。
```

其中需要注意的是，Pipeline首次针对每一个Node通过调用AddDeferredNode方法加入到DRQ中，此时所有的Node都会加入到m_readyNodes中，然后通过调用dispatchReadyNodes方法，触发DRQ开始进行整个内部处理流程，基本流程可以参见下图，接下来就以该图进行深入梳理下：

![DeferredRequestQueue](assets/DeferredRequestQueue.png)

```
当调用了DRQ的dispatchReadyNodes方法后，会从m_readyNodes链表里面依次取出Dependency，
    将其投递到DeferredWorkerWrapper线程中，在该线程会从Dependency取出Node调用
    其ProcessRequest方法开始在Node内部处理本次request，处理完成之后如果当前Node依然存在依赖项，
    则调用AddDeferredNode方法将Node再次加入到m_deferredNodes链表中，
    并且加入新的依赖项，存入m_pDependencyMap hash表中。
 
在Node处理request的过程中，会持续更新meta data以及property，
    此时会通过调用MetadataSlot的PublishMetadata方法更新到MetadataPool中，
    此时MetadataPool会调用之前在DRQ初始化时候注册的几个回调方法OnPropertyUpdate
    以及OnMetadataUpdate方法通知DRQ，此时有新的meta data 和property更新，
    接下来会在这两个方法中调用UpdateDependency方法，去更新meta data 
    和property到m_pDependencyMap中，并且将没有任何依赖项的Node从m_deferredNodes
    取出加入到m_readyNodes，等待处理。
 
与此同时，Node的处理结果也会通过ProcessFenceCallback方法通知pipeline，
    并且调用pipeline的NonSinkPortFenceSignaled方法，在该方法内部又会去
    调用DRQ的FenceSignaledCallback方法，而该方法又会调用UpdateDependency更新依赖，
    并将依赖项都满足的Node从m_deferredNodes取出加入到m_readyNodes，
    然后调用dispatchReadyNodes继续进行处理。
```

5.上传拍照结果
在用户开启了相机应用，相机框架收到某次Request请求之后会开始对其进行处理，一旦有图像数据产生便会通过层层回调最终返回到应用层进行显示，这里我们针对CamX-CHI部分对于拍照结果的上传流程进行一个简单的梳理：

每一个Request对应了三个Result，分别是partial metadata、metadata以及image data，对于每一个Result，上传过程可以大致分为以下两个阶段：

```
Session内部完成图像数据的处理，将结果发送至Usecase中
Usecase接收到来自Session的数据，并将其上传至Provider
```

首先来看下Session内部完成图像数据的处理后是如何将结果发送至Usecase的：

![session-result-to-usecase](assets/session-result-to-usecase.png)

在整个requets流转的过程中，一旦Node中有Partial Meta Data产生，便会调用Node的ProcessPartialMetadataDone方法去通知从属的Pipeline，其内部又调用了pipeline的NotifyNodePartialMetadataDone方法。每次调用Pipeline的NotifyNodePartialMetadataDone方法都会去将pPerRequestInfo→numNodesPartialMetadataDone加一并且判断当前值是否等于pipeline中的Node数量，一旦相等，便说明当前所有的Node都完成了partial meta data的更新动作，此时，便会调用ProcessPartialMetadataRequestIdDone方法，里面会去取出partial meta data，并且重新封装成ResultsData结构体，将其作为参数通过Session的NotifyResult方法传入Session中，之后在Session中经过层层调用最终会调用到内部成员变量m_chiCallBacks的ChiProcessPartialCaptureResult方法，该方法正是创建Session的时候，传入Session中的Usecase的方法（AdvancedCameraUsecase::ProcessDriverPartialCaptureResultCb），通过该方法就将meta data返回到了CHI中。

同样地，Meta data的逻辑和Partial Meta Data很相似，每个Node在处理request的过程中，会调用ProcessMetadataDone方法将数据发送到Pipeline中，一旦所有的Node的meta data否发送完成了，pipeline会调用NotifyNodeMetadataDone方法，将最终的结果发送至Session中，最后经过层层调用，会调用Session 中成员变量m_chiCallBacks的ChiProcessCaptureResult方法，将结果发送到CHI中Usecase中。

图像数据的流转和前两个meta data的流转有点儿差异，一旦Node内部图像数据处理完成后便会调用其ProcessFenceCallback方法，在该方法中会去检查当前输出是否是SInk Buffer，如果是则会调用Pipeline的SinkPortFenceSignaled方法将数据发送到Pipeline中，在该方法中Pipeline又会将数据发送至Session中，最后经过层层调用，会调用Session 中成员变量m_chiCallBacks的ChiProcessCaptureResult方法，将结果发送到CHI中Usecase中。

接下来我们来看下一旦Usecase接收到Session的数据，是如何发送至Provider的：

我们以常用的AdvancedCameraUsecase为例进行代码的梳理：

![usecase-to-provider](assets/usecase-to-provider.png)

如上图所示，整个result的流转逻辑还是比较清晰的，CamX通过回调方法将结果回传给CHI中，而在CHI中，首先判断是否需要发送到具体的Feature的， 如果需要，则调用相应Feature的ProcessDriverPartialCaptureResult或者ProcessResult方法将结果发送到具体的Feature中，一旦处理完成，便会调用调用CameraUsecaseBase的ProcessAndReturnPartialMetadataFinishedResults以及ProcessAndReturnFinishedResults方法将结果发送到Usecase中，如果当前不需要发送到Feature进行处理，就在AdvancedCameraUsecase中调用CameraUsecaseBase的SessionCbPartialCaptureResult以及SessionCbCaptureResult方法，然后通过Usecase::ReturnFrameResult方法将结果发送到ExtensionModule中，之后调用ExtensionModule中存储的CamX中的回调函数process_capture_result将结果发送到CamX中的HALDevice中，之后HALDevice又通过之前存储的上层传入的回调方法，将结果最终发送到CameraDeviceSession中。

通过以上的梳理，可以发现，整个CamX-CHI框架设计的很不错，目录结构清晰明确，框架简单高效，流程控制逻辑分明，比如针对某一图像请求，整个流程经过Usecase、Feature、Session、Pipeline并且给到具体的Node中进行处理，最终输出结果。另外，相比较之前的QCamera & Mm-Camera框架的针对某个算法的扩展需要在整个流程代码中嵌入自定义的修改做法而言，CamX-CHI通过将自定义实现的放入CHI中，提高了其扩展性，降低了开发门槛，使得平台厂商在并不是很熟悉CamX框架的情况下也可以通过小规模的修改成功添加新功能。但是人无完人，框架也是一样，该框架异步化处理太多，加大了定位问题以及解决问题的难度，给开发者带来了不小的压力。另外，框架对于内存的要求较高，所以在一些低端机型尤其是低内存机型上，整个框架的运行效率可能会受到一定的限制，进而导致相机效率低于预期。



# 

# 七、驱动层（V4L2 框架）

## 1、概览

相机驱动层位于HAL Moudle与硬件层之间，借助linux内核驱动框架，以文件节点的方式暴露接口给用户空间，让HAL Module通过标准的文件访问接口，从而能够将请求顺利地下发到内核中，而在内核中，为了更好的支持视频流的操作，早先提出了v4l视频处理框架，但是由于操作复杂，并且代码无法进行较好的重构，难以维护等原因，之后便衍生出了v4l2框架。

按照v4l2标准，它将一个数据流设备抽象成一个videoX节点，从属的子设备都对应着各自的v4l2_subdev实现，并且通过media controller进行统一管理，整个流程复杂但高效，同时代码的扩展性也较高。

而对高通平台而言，高通整个内核相机驱动是建立在v4l2框架上的，并且对其进行了相应的扩展，创建了一个整体相机控制者的CRM，它以节点video0暴露给用户空间，主要用于管理内核中的Session、Request以及与子设备，同时各个子模块都实现了各自的v4l2_subdev设备，并且以v4l2_subdev节点暴露给用户空间，与此同时，高通还创建了另一个video1设备Camera SYNC，该设备主要用于同步数据流，保证用户空间和内核空间的buffer能够高效得进行传递。

再往下与相机驱动交互的便是整个相机框架的最底层Camera Hardware了，驱动部分控制着其上下电逻辑以及寄存器读取时序并按照I2C协议进行与硬件的通信，和根据MIPI CSI协议传递数据，从而达到控制各个硬件设备，并且获取图像数据的目的。

V4L2英文是Video for Linux 2，该框架是诞生于Linux系统，用于提供一个标准的视频控制框架，其中一般默认会嵌入media controller框架中进行统一管理，v4l2提供给用户空间操作节点，media controller控制对于每一个设备的枚举控制能力，于此同时，由于v4l2包含了一定数量的子设备，而这一系列的子设备都是处于平级关系，但是在实际的图像采集过程中，子设备之间往往还存在着包含于被包含的关系，所以为了维护并管理这种关系，media controller针对多个子设备建立了的一个拓扑图，数据流也就按照这个拓扑图进行流转。

## 2、流程简介

整个对于v4l2的操作主要包含了如下几个主要流程：

![V4L2](assets/V4L2.png)

a) 打开video设备
在需要进行视频数据流的操作之前，首先要通过标准的字符设备操作接口open方法来打开一个video设备，并且将返回的字符句柄存在本地，之后的一系列操作都是基于该句柄，而在打开的过程中，会去给每一个子设备的上电，并完成各自的一系列初始化操作。

b) 查看并设置设备
在打开设备获取其文件句柄之后，就需要查询设备的属性，该动作主要通过ioctl传入VIDIOC_QUERYCAP参数来完成，其中该系列属性通过v4l2_capability结构体来表达，除此之外，还可以通过传入VIDIOC_ENUM_FMT来枚举支持的数据格式，通过传入VIDIOC_G_FMT/VIDIOC_S_FMT来分别获取和获取当前的数据格式，通过传入VIDIOC_G_PARM/VIDIOC_S_PARM来分别获取和设置参数。

c) 申请帧缓冲区
完成设备的配置之后，便可以开始向设备申请多个用于盛装图像数据的帧缓冲区，该动作通过调用ioctl并且传入VIDIOC_REQBUFS命令来完成，最后将缓冲区通过mmap方式映射到用户空间。

d) 将帧缓冲区入队
申请好帧缓冲区之后，通过调用ioctl方法传入VIDIOC_QBUF命令来将帧缓冲区加入到v4l2 框架中的缓冲区队列中，静等硬件模块将图像数据填充到缓冲区中。

e) 开启数据流
将所有的缓冲区都加入队列中之后便可以调用ioctl并且传入VIDIOC_STREAMON命令，来通知整个框架开始进行数据传输，其中大致包括了通知各个子设备开始进行工作，最终将数据填充到V4L2框架中的缓冲区队列中。

f) 将帧缓冲区出队
一旦数据流开始进行流转了，我们就可以通过调用ioctl下发VIDIOC_DQBUF命令来获取帧缓冲区，并且将缓冲区的图像数据取出，进行预览、拍照或者录像的处理，处理完成之后，需要将此次缓冲区再次放入V4L2框架中的队列中等待下次的图像数据的填充。

整个采集图像数据的流程现在看来还是比较简单的，接口的控制逻辑很清晰，主要原因是为了提供给用户的接口简单而且抽象，这样方便用户进行集成开发，其中的大部分复杂的业务处理都被V4L2很好的封装了，接下来我们来详细了解下V4L2框架内部是如何表达以及如何运转的。

## 3、关键结构体

![key-struct](assets/key-struct.png)

从上图不难看出，v4l2_device作为顶层管理者，一方面通过嵌入到一个video_device中，暴露video设备节点给用户空间进行控制，另一方面，video_device内部会创建一个media_entity作为在media controller中的抽象体，被加入到media_device中的entitie链表中，此外，为了保持对所从属子设备的控制，内部还维护了一个挂载了所有子设备的subdevs链表。

而对于其中每一个子设备而言，统一采用了v4l2_subdev结构体来进行描述，一方面通过嵌入到video_device，暴露v4l2_subdev子设备节点给用户空间进行控制，另一方面其内部也维护着在media controller中的对应的一个media_entity抽象体，而该抽象体也会链入到media_device中的entities链表中。

通过加入entities链表的方式，media_device保持了对所有的设备信息的查询和控制的能力，而该能力会通过media controller框架在用户空间创建meida设备节点，将这种能力暴露给用户进行控制。

由此可见，V4L2框架都是围绕着以上几个主要结构体来进行的，接下来我们依次简单介绍下：
v4l2_device 源码如下：

```
struct v4l2_device {
    struct device *dev;
#if defined(CONFIG_MEDIA_CONTROLLER)
    struct media_device *mdev;                                                                                                                         
#endif
    struct list_head subdevs;
    spinlock_t lock;
    char name[V4L2_DEVICE_NAME_SIZE];
    void (*notify)(struct v4l2_subdev *sd,
        unsigned int notification, void *arg);
    struct v4l2_ctrl_handler *ctrl_handler;
    struct v4l2_prio_state prio;
    struct kref ref;
    void (*release)(struct v4l2_device *v4l2_dev);
};
```

该结构体代表了一个整个V4L2设备，作为整个V4L2的顶层管理者，内部通过一个链表管理着整个从属的所有的子设备，并且如果将整个框架放入media conntroller进行管理，便在初始化的时候需要将创建成功的media_device赋值给内部变量 mdev，这样便建立了于与media_device的联系，驱动通过调用v4l2_device_register方法和v4l2_device_unregister方法分别向系统注册和释放一个v4l2_device。

v4l2_subdev源码如下：

```
struct v4l2_subdev {
#if defined(CONFIG_MEDIA_CONTROLLER)
    struct media_entity entity;
#endif
    struct list_head list;
    struct module *owner;
    bool owner_v4l2_dev;
    u32 flags;
    struct v4l2_device *v4l2_dev;
    const struct v4l2_subdev_ops *ops;
    const struct v4l2_subdev_internal_ops *internal_ops;
    struct v4l2_ctrl_handler *ctrl_handler;
    char name[V4L2_SUBDEV_NAME_SIZE];
    u32 grp_id;
    void *dev_priv;
    void *host_priv;
    struct video_device *devnode;
    struct device *dev;
    struct fwnode_handle *fwnode;
    struct list_head async_list;
    struct v4l2_async_subdev *asd;
    struct v4l2_async_notifier *notifier;
    struct v4l2_subdev_platform_data *pdata;
};
```

该结构体代表了一个子设备，每一个子设备都需要在初始化的时候挂载到一个总的v4l2_device上，并且将该v4l2设备赋值给内部的v4l2_dev变量，之后将自身加入到v4l2_device中的子设备链表中进行统一管理，这种方式提高了遍历访问所有子设备的效率，同时为了表达不同硬件模块的特殊操作行为，v4l2_subdev定义了一个v4l2_subdev_ops 结构体来进行定义，其实现交由不同的硬件模块来具体完成。其中如果使能了CONFIG_MEDIA_CONTROLLER宏，便会在media_controller中生成一个对应的media_entity，来代表该子设备，而该entity便会存入子设备结构体中的entity变量中，最后，如果需要创建一个设备节点的话，通过video_device调用标准API接口进行实现，而相应的video_device便会存入其内部devnode变量中。

video_device源码如下：

```
struct video_device
{
#if defined(CONFIG_MEDIA_CONTROLLER)
    struct media_entity entity;
    struct media_intf_devnode *intf_devnode;
    struct media_pipeline pipe;
#endif
    const struct v4l2_file_operations *fops;
 
    u32 device_caps;
 
    /* sysfs */
    struct device dev;
    struct cdev *cdev;
 
    struct v4l2_device *v4l2_dev;
    struct device *dev_parent;
 
    struct v4l2_ctrl_handler *ctrl_handler;
 
    struct vb2_queue *queue;
 
    struct v4l2_prio_state *prio;
 
    /* device info */
    char name[32];
    int vfl_type;
    int vfl_dir;
    int minor;
    u16 num;
    unsigned long flags;
    int index;
 
    /* V4L2 file handles */
    spinlock_t      fh_lock;
    struct list_head    fh_list;
 
    int dev_debug;
 
    v4l2_std_id tvnorms;
 
    /* callbacks */
    void (*release)(struct video_device *vdev);
    const struct v4l2_ioctl_ops *ioctl_ops;
    DECLARE_BITMAP(valid_ioctls, BASE_VIDIOC_PRIVATE);
 
    DECLARE_BITMAP(disable_locking, BASE_VIDIOC_PRIVATE);
    struct mutex *lock;
};
```

如果需要给v4l2_device或者v4l2_subdev在系统中创建节点的话，便需要实现该结构体，并且通过video_register_device方法进行创建，而其中的fops便是video_device所对应的操作方法集，在v4l2框架内部，会将video_device嵌入到一个具有特定主设备号的字符设备中，而其方法集会在操作节点时被调用到。除了这些标准的操作集外，还定义了一系列的ioctl操作集，通过内部ioctl_ops来描述。

media_device源码如下:

```
struct media_device {
    /* dev->driver_data points to this struct. */
    struct device *dev;
    struct media_devnode *devnode;
 
    char model[32];
    char driver_name[32];
    char serial[40];
    char bus_info[32];
    u32 hw_revision;
 
    u64 topology_version;
 
    u32 id;
    struct ida entity_internal_idx;
    int entity_internal_idx_max;
 
    struct list_head entities;
    struct list_head interfaces;
    struct list_head pads;
    struct list_head links;
 
    /* notify callback list invoked when a new entity is registered */
    struct list_head entity_notify;
 
    /* Serializes graph operations. */
    struct mutex graph_mutex;
    struct media_graph pm_count_walk;
 
    void *source_priv;
    int (*enable_source)(struct media_entity *entity,
                 struct media_pipeline *pipe);
    void (*disable_source)(struct media_entity *entity);
 
    const struct media_device_ops *ops;
};
```

如果使能了CONFIG_MEDIA_CONTROLLER宏，则当v4l2_device初始化的过程中便会去创建一个media_device，而这个media_device便是整个media controller的抽象管理者，每一个v4l2设备以及从属的子设备都会对应的各自的entity，并且将其存入media_device中进行统一管理，与其它抽象设备一样，media_device也具有自身的行为，比如用户可以通过访问media节点，枚举出所有的从属于同一个v4l2_device的子设备，另外，在开启数据流的时候，media_device通过将各个media_entity按照一定的顺序连接起来，实现了数据流向的整体控制。

vb2_queue源码如下：

```
struct vb2_queue {
    unsigned int            type;
    unsigned int            io_modes;
    struct device           *dev;
    unsigned long           dma_attrs;
    unsigned            bidirectional:1;
    unsigned            fileio_read_once:1;
    unsigned            fileio_write_immediately:1;
    unsigned            allow_zero_bytesused:1;
    unsigned           quirk_poll_must_check_waiting_for_buffers:1;
 
    struct mutex            *lock;
    void                *owner;
 
    const struct vb2_ops        *ops;
    const struct vb2_mem_ops    *mem_ops;
    const struct vb2_buf_ops    *buf_ops;
 
    void                *drv_priv;
    unsigned int            buf_struct_size;
    u32             timestamp_flags;
    gfp_t               gfp_flags;
    u32             min_buffers_needed;
 
    /* private: internal use only */
    struct mutex            mmap_lock;
    unsigned int            memory;
    enum dma_data_direction     dma_dir;
    struct vb2_buffer       *bufs[VB2_MAX_FRAME];
    unsigned int            num_buffers;
 
    struct list_head        queued_list;
    unsigned int            queued_count;
 
    atomic_t            owned_by_drv_count;
    struct list_head        done_list;
    spinlock_t          done_lock;
    wait_queue_head_t       done_wq;
 
    struct device           *alloc_devs[VB2_MAX_PLANES];
 
    unsigned int            streaming:1;
    unsigned int            start_streaming_called:1;
    unsigned int            error:1;
    unsigned int            waiting_for_buffers:1;
    unsigned int            is_multiplanar:1;
    unsigned int            is_output:1;
    unsigned int            copy_timestamp:1;
    unsigned int            last_buffer_dequeued:1;
 
    struct vb2_fileio_data      *fileio;
    struct vb2_threadio_data    *threadio;
 
#ifdef CONFIG_VIDEO_ADV_DEBUG
    /*
     * Counters for how often these queue-related ops are
     * called. Used to check for unbalanced ops.
     */
    u32             cnt_queue_setup;
    u32             cnt_wait_prepare;
    u32             cnt_wait_finish;
    u32             cnt_start_streaming;
    u32             cnt_stop_streaming;
#endif
};
```

在整个V4L2框架运转过程中，最为核心的是图像数据缓冲区的管理，而这个管理工作便是由vb2_queue来完成的，vb2_queue通常在打开设备的时候被创建，其结构体中的vb2_ops可以由驱动自己进行实现，而vb2_mem_ops代表了内存分配的方法集，另外，还有一个用于将管理用户空间和内核空间的相互传递的方法集buf_ops，而该方法集一般都定义为v4l2_buf_ops这一标准方法集。除了这些方法集外，vb2_queue还通过一个vb2_buffer的数组来管理申请的所有数据缓冲区，并且通过queued_list来管理入队状态的所有buffer，通过done_list来管理被填充了数据等待消费的所有buffer。

vb2_buffer源码如下：

```
struct vb2_buffer {
    struct vb2_queue    *vb2_queue;
    unsigned int        index;
    unsigned int        type;
    unsigned int        memory;
    unsigned int        num_planes;
    struct vb2_plane    planes[VB2_MAX_PLANES];
    u64         timestamp;
 
    /* private: internal use only
     *
     * state:       current buffer state; do not change
     * queued_entry:    entry on the queued buffers list, which holds
     *          all buffers queued from userspace
     * done_entry:      entry on the list that stores all buffers ready
     *          to be dequeued to userspace
     */
    enum vb2_buffer_state   state;
 
    struct list_head    queued_entry;
    struct list_head    done_entry;
};
```

该结构体代表了V4L2框架中的图像缓冲区，当处于入队状态时内部queued_entry会被链接到vb2_queue中的queued_list中，当处于等待消费的状态时其内部done_entry会被链接到vb2_queue 中的done_list中，而其中的vb2_queue便是该缓冲区的管理者。

以上便是V4L2框架的几个核心结构体，从上面的简单分析不难看出，v4l2_device作为一个相机内核体系的顶层管理者，内部使用一个链表控制着所有从属子设备v4l2_subdev，使用vb2_queue来申请并管理所有数据缓冲区，并且通过video_device向用户空间暴露设备节点以及控制接口，接收来自用户空间的控制指令，通过将自身嵌入media controller中来实现枚举、连接子设备同时控制数据流走向的目的。

## 4、模块初始化

整个v4l2框架是在linux内核中实现的，所以按照内核驱动的运行机制，会在系统启动的过程中，通过标准的module_init方式进行初始化操作，而其初始化主要包含两个方面，一个是v4l2_device的初始化，一个是子设备的初始化，首先我们来看下v4l2_device的初始化动作的基本流程。

由于驱动的实现都交由各个平台厂商进行实现，所有内部逻辑都各不相同，这里我们抽离出主要方法来进行梳理：

首先对于v4l2_device的初始化而言，在系统启动的过程中，linux内核会找到module_init声明的驱动，调用其probe方法进行探测相应设备，一旦探测成功，便表示初始化工作完成。

而在probe方法内部，主要做了以下操作：

获取dts硬件信息，初始化部分硬件设备。
创建v4l2_device结构体，填充信息，通过v4l2_device_register方法向系统注册并且创建video设备节点。
创建media_device结构体，填充信息，通过media_device_register向系统注册，并创建media设备节点，并将其赋值给v4l2_device中的mdev。
创建v4l2_device的media_entity,并将其添加到media controller进行管理。
类似于v4l2_device的初始化工作，子设备的流程如下：

获取dts硬件信息，初始化子设备硬件模块
创建v4l2_subdev结构体，填充信息，通过v4l2_device_register_subdev向系统注册，并将其挂载到v4l2_device设备中
创建对应的media_entity，并通过media_device_register_entity方法其添加到media controller中进行统一管理。
最后调用v4l2_device_register_subdev_nodes方法，为所有的设置了V4L2_SUBDEV_FL_HAS_DEVNODE属性的子设备创建设备节点。
五、处理用户空间请求
系统启动之后，初始化工作便已经完成，现在一旦用户想要使用图像采集功能，便会触发整个视频采集流程，会通过操作相应的video节点来获取图像数据，一般来讲，标准的V4L2框架只需要通过操作video节点即可，但是由于现在的硬件功能越来越复杂，常规的v4l2_controller已经满足不了采集需求，所以现在的平台厂商通常会暴露子设备的设备节点，在用户空间直接通过标准的字符设备控制接口来控制各个设备，而现在我们的目的是梳理V4L2框架，所以暂时默认不创建子设备节点，简单介绍下整个流程。

在操作之前，还有一个准备工作需要做，那就是需要找到哪些是我们所需要的设备，而它的设备节点是什么，此时便可以通过打开media设备节点，并且通过ioctl注入MEDIA_IOC_ENUM_ENTITIES参数来获取v4l2_device下的video设备节点，该操作会调用到内核中的media_device_ioctl方法，而之后根据传入的命令，进而调用到media_device_enum_entities方法来枚举所有的设备。

整个采集流程，主要使用三个标准字符设备接口来完成，分别是用于打开设备的open方法、用于控制设备的ioctl方法以及关闭设备的close方法。

1. 打开设备(open)
一旦确认了我们需要操作的video节点是哪一个，便可以通过调用字符设备标准接口open方法来打开设备，而这个方法会首先陷入内核空间，然后调用file_operations中的open方法，再到v4l2_file_operations中的open方法，而该方法由驱动自己进行实现，其中主要包括了给各个硬件模块上电，并且调用vb2_queue_init方法创建并初始化一个vb2_queue用于数据缓冲区的管理。

2. 控制设备(ioctl)
在打开设备之后，接下来的大部分操作都是通过ioctl方法来完成的，而在该方法中，会首先陷入到内核空间，之后调用字符设备的v4l2_fops中的v4l2_ioctl方法，而在该方法中又会去调用video_device的video_ioctl2方法，video_ioctl2方法定义了一系列video标准的方法，通过不同的命令在v4l2_ioctls中找到相应的标准方法实现，同时为了满足用户自定义命令的实现，在video_ioctl2方法中会去调用到之前注册video_device时赋予的ioctl_ops中的vidioc_default方法，在该方法中加入用户自己的控制逻辑。

在整个控制流程中，首先通过命令VIDIOC_QUERYCAP来获取设备所具有的属性，通过VIDIOC_G_PARM/VIDIOC_S_PARM来分别获取和设置设备参数，在这一系列操作配置完成之后，便需要向内核申请用于数据流转的缓冲区(Buffer)，该操作通过命令VIDIOC_REQBUFS来完成，在内核部分主要调用了标准方法vb2_reqbufs，进而调用__vb2_queue_alloc来向内核申请已知个数的Buffer，并且将其存入之前创建的vb2_queue中进行管理。

申请好了Buffer之后，便可以通过传入VIDIOC_QBUF命令将申请的Buffer入队，具体操作最终会调用vb2_qbuf方法，而在该方法中会从vb2_queue的bufs数组中取出Buffer，将其加入queued_list链表中，并且更新Buffer状态，等待数据的填充或者来自用户空间的出队操作。

在完成上面的操作后，整个数据流并没有开始流转起来，所以需要下发VIDIOC_STREAMON命令来通知整个框架开始出数据，在驱动中主要会去调用vb2_streamon方法，进而调用vb2_start_streaming方法，其中该方法会去将队列中的的Buffer放入到相应的驱动中，等待被填充，紧接着会去调用vb2_queue.ops.start_streaming方法来通知设备开始出图，而该方法一般由驱动自己实现，最后会调用v4l2_subdev_call(subdev, video, s_stream, mode)方法通知各个子设备开始出图。

当有图像产生时，会填充到之前传入的buffe中，并且调用vb2_buffer_done方法通知vb2_queue将buffer加入到done_list链表中，并更新状态为VB2_BUF_STATE_DONE。

在整个数据流开启之后，并不会自动的将图像传入用户空间，必须通过VIDIOC_DQBUF命令来从设备中读取一个帧图像数据，具体操作是通过层层调用会调用到vb2_dqbuf方法，而在该方法中会调用__vb2_get_done_vb方法去从done_list中获取Buffer，如果当前链表为空则会等待最终数据准备好，如果有准备好的buffer便直接从done_list取出，并且将其从queued_list中去掉，最后通过__vb2_dqbuf方法将Buffer返回用户空间。

获取到图像数据之后，便可以进行后期的图像处理流程了，在处理完成之后，需要下发VIDIOC_QBUF将此次buffer重新加入queued_list中，等待下一次的数据的填充和出队操作。

但不需要进行图像的采集时，可以通过下发VIDIOC_STREAMOFF命令来停止整个流程，具体流程首先会调用v4l2_subdev_call(subdev, video, s_stream, 0)通知所有子设备停止出图操作，其次调用vb2_buffer_done唤醒可能的等待Buffer的线程，同时更新Buffer状态为VB2_BUF_STATE_ERROR，然后调用vb2_streamoff取消所有的数据流并更新vb2_queue.streaming的为disable状态。

3. 关闭设备(close)
但确认不使用当前设备进行图像采集操作之后，便可以调用标准方法close来关闭设备。其中主要包括了调用vb2_queue_release方法释放了vb2_queue以及设备下电操作和相关资源的释放。

通过上面的介绍，我相信我们已经对整个V4L2框架有了一个比较深入的认识， 然而对于一个优秀的软件架构而言，仅仅是支持现有的功能是远远不够的，随着功能的不断完善，势必会出现需要进行扩展的地方，而v4l2在设计之初便很好的考虑到了这一点，所以提供了用于扩展的方法集，开发者可以通过加入自定的命令来扩充整个框架，高通在这一点上做的非常好，在v4l2框架基础上，设计出了一个独特的KMD框架，提供给UMD CSL进行访问的接口。



# 七、驱动层（高通 KMD 框架）

一、概览
利用了V4L2可扩展这一特性，高通在相机驱动部分实现了自有的一套KMD框架，该框架通过V4L2标准方法在系统中创建设备节点，将控制接口直接暴露给UMD CSL进行访问，而其内部主要定义了一系列核心模块，包括CRM(Camera Request Manager)，用于管理整个KMD的Session/Link的创建销毁以及Request的在子设备间的流转，该模块创建video0设备节点暴露关键接口给UMD，此外还包括了Sync模块，主要负责了UMD/KMD之间的数据同步与传输，创建video1设备节点暴露接口给UMD进行访问，除此之外，为了更精细化地控制一系列的硬件图像处理模块，包括ISP/IPE/Sensor等硬件模块，高通也分别为各自子模块创建了设备节点，进而暴露控制接口给UMD进行访问。
其中主要目录如下：

```java
cam_core/： 关于KMD核心函数的实现都放在这，主要包括了subdev、node、context
            的一些诸如创建/注册/销毁等标准方法。
cam_req_mgr/: CRM的具体实现，用于创建v4l2_device，用于管理所有的子设备，
            同时生成video设备节点，暴露控制接口给UMD，主要包括了Session/Link的行为管理
            以及Request的同步与分发，此外，还创建了media_device，
            用于暴露枚举接口给UMD来轮询查找整个KMD的子设备。
cam_sync/: 该部分主要实现了用于保持与UMD的图像数据的同步相关业务逻辑，由于该模块的特殊性，
            高通直接为其创建了一个单独的video设备节点，暴露了用于同步的一些控制接口。
cam_utils/: 一些共有方法的实现，包括debug方法集等
cam_smmu/: 高通自己实现了一套smmu api，供KMD使用
cam_lrme/: 低分辨率运动估计模块的驱动实现
cam_fd/: 人脸识别的驱动程序
cam_isp/: isp的驱动程序
cam_jpeg/: 编码器，可以通过该驱动完成jpeg的编码工作
cam_cdm/: camera data mover，数据移动器的驱动实现，主要用于解析由CSL传入的命令信息，
          其中包括了寄存器的设置以及图像数据的处理等。
cam_cpas/: 该模块主要用于CSL获取camera 平台驱动信息，IPE/BPS电源控制等
cam_icp/: image control processor ，图像处理控制器驱动实现
cam_sensor_module/: 类传感器的系列硬件模块：
```

cam_sensor_module模块又分下面部分

```java
	cam_actuator/: 对焦马达的驱动实现
    cam_cci/: 实现了用于通讯的CCI接口，其中包括了I2C以及gpio的实现
    cam_csiphy: 基于MIPI CSI接口的物理层驱动，用于传输图像数据
    cam_sensor_io: 使用cam_cci，向上实现了控制sensor的IO接口
    cam_sensor: sensor 的驱动实现
    cam_sensor_util: sensor相关的公有方法的实现
    cam_eeprom: eeprom设备的驱动实现
    cam_ois ： 光学防抖设备的驱动实现
    cam_flash: 闪光灯设备的驱动实现
```

# 2、核心模块解析

正如之前介绍的那样，整个框架主要由三个部分组成，CRM/Camera Sync以及子模块，接下来我们以下图为例简单讲解下各自的关系：

![core-module-relation](assets/core-module-relation.png)

在系统初始化时，CRM内部会创建一个v4l2_device结构体，用于管理所有的子设备，与此同时每一个子设备在注册的时候都会创建各自的v4l2_subdev挂载到该v4l2_device上面。此外，CRM会创建一个video0设备节点提供关键接口给CSL来进行访问，而每个子设备也会在系统中生成各自的v4l2-sbudev设备节点，提供接口给CSL进行更为精细化的控制。而其中的Cam Sync在初始化的过程中，也创建了一个v4l2_device设备，并且生成了video1节点给CSL进行控制。这个框架主要就是围绕这三个部分进行的，CRM用于管理Session/Link的创建，控制Request在各个子设备中的流转，子设备受CSL控制进行配置以及图像处理工作，而一旦图像处理完成便会将结果发送至Cam Sync模块，进上传至CSL中。

1.CRM(Camera Request Manager)
该模块本质上是一个软件模块，主要做了以下几个事情：

```java
接收来自CSL的Session/Link/Request请求，并且维护其在内核的状态。
在不同pipeline delay的子模块间，同步每一个Request状态，并按照需要发送给每一个子设备。
如果出现错误，负责上传至CSL。
负责针对实时子模块的flush操作。
```

其中针对Session/Link/Request的请求便是通过之前创建的video设备节点将接口暴露给CSL，一旦接收到命令便开始进行处理，而命令主要有以下几个：

```java
CAM_REQ_MGR_CREATE_SESSION/CAM_REQ_MGR_DESTROY_SESSION： 分别表示了Session的创建和销毁，
        该Session保持着与CamX-CHI的一一对应关系。
CAM_REQ_MGR_LINK/CAM_REQ_MGR_UNLINK： 分别表示了Link的创建和销毁动作，
        每一个Session可以包含多条Link，而每一个Link都连接着此次图像采集过程中所需要的子设备，
        CRM也是通过该Link来管理Request同步与分发的操作。
CAM_REQ_MGR_SCHED_REQ：一旦CSL开始下发Request的时候，便可以通过该命令告知KMD，
        而在KMD中，CRM会将此次Request存入Link中的in_q数组中，
        当子设备告知准备好了此次Request的处理后，便通知子设备进行配置并处理Request。
CAM_REQ_MGR_ALLOC_BUF/CAM_REQ_MGR_RELEASE_BUF: 图像缓冲区的申请与释放，
        CRM中使用cam_mem_table结构体来管理着申请的缓冲区。
```

一旦CRM接收了来自CSL的请求，便会在内部进行处理，而其中的一系列业务处理便会通过接下来的几个结构体来完成：

首先在初始化过程中，会去创建一个cam_req_mgr_device。
该结构体有以下几个主要的成员：

```java
video: 存储着对应的video_device。
v4l2_dev: 保存着初始化过程中创建的v4l2_device。
subdev_nodes_created: 标志着从属于v4l2_device的子设备是否都成功创建了设备节点。
cam_eventq： v4l2文件描述结构体，其中维护着event事件队列。
```

之后会去创建一个cam_req_mgr_core_device，该结构体比较简单主要用于维护一个Session链表，在CSL下发创建Session的动作后，会将创建好的Session放入该量表中，同时通过crm_lock保持着业务处理中的同步。

一个Session可以包含很多条Link，其中变量num_links存储了Link数量，数组links存储着所有link，entry变量作为当前session的实体可以嵌入cam_req_mgr_core_device中的session链表中进行统一管理。

在CSL下发CAM_REQ_MGR_LINK命令的时候，会去创建cam_req_mgr_core_link。
该结构体比较复杂，接下来我们主要介绍下几个主要的变量：

```java
link_hdl：作为该Link的句柄，区别于其它Link。
num_devs： 表示了该条Link上连接了多少个子设备。
max_delay： 表示了从属于该Link上的所有子设备具有的最大的Pipeline delay值。
l_dev： 存储着所有从属于该Link上的子设备，后续对于子设备的控制都是通过该数组来进行的。
req： 该成员主要用于管理下发的request。
state: 标志着该Link的状态，而Link状态主要包括了       
       CAM_CRM_LINK_STATE_AVAILABLE/
       CAM_CRM_LINK_STATE_IDLE/
       CAM_CRM_LINK_STATE_READY/
       CAM_CRM_LINK_STATE_ERR几种状态。
```

创建完Link之后，会将其存入一个存储cam_req_mgr_core_link的全局变量g_links中进行统一管理。

而当下发CAM_REQ_MGR_SCHED_REQ命令的时候，会在内部进行解析，并且将其存入cam_req_mgr_core_link中的cam_req_mgr_req_data中等待后续的流转。
其中in_q变量主要用于存储request，而l_tbl用于记录pipeline delay的相关信息，而apply_data数组用于存储所有的等待处理的request信息。

2.Cam Sync
该模块本质上是一个软件模块，用于保持与UMD的图像数据的同步，主要利用了V4L2框架的event机制，由CSL进行事件的等待，一旦数据处理完毕，该模块便可以向上层发送事件，进而，通知CSL取出数据进行下一步处理，其中包括了几个主要ioctl的命令：

```java
CAM_SYNC_CREATE: 一旦CSL部分需要创建一个用于同步的实体的时候便下发该命令，
        而在Cam Sync中，会将传入的信息存入内部的sync_table_row数组中进行管理，
        并且将生成的sync_obj传入上层。
CAM_SYNC_DESTROY： 销毁用于同步的sync实体。
CAM_SYNC_REGISTER_PAYLOAD： 通过该命令将一些同步的回调方法注册到Cam Sync中，
        这样一当数据处理完成，Cam Sync便可以由之前创建的sync_obj来找到相应的回调方法，
        进而调用该回调方法进行后续处理。
CAM_SYNC_DEREGISTER_PAYLOAD：释放之前注册的相关同步实体的信息，包括其回调方法。
CAM_SYNC_SIGNAL：该命令主要用于CamX-CHI中软件Node处理完数据之后，
        通知Cam Sync进行后续处理的目的。
```

其中包括了几个比较重要的结构体，首先在初始化过程中会去创建sync_device结构体，其主要的几个变量如下：

```java
vdev: 创建的video_device。
v4l2_dev: 创建的v4l2_device设备。
sync_table: 用于存储sync_table_row的数组。
cam_sync_eventq: v4l2设备描述符结构体，其中维护着event事件队列。
```

其中最重要的时sync_table中存储的sync_table_row结构体，它代表了整个对应于CSL中的sync object，其中比较重要的变量含义如下：

```java
sync_id：该sync object的唯一标识，同时该标识于CSL保持同步。
state: 代表了当前sync object的状态。
user_payload_list： 存储着该sync object所对应的来自UMD的payload，
        该payload在KMD中并没有被使用，仅仅存储与KMD中，一旦当前sync object被触发，
        便直接将其再次传入UMD中。
```

## 3、模块初始化

在系统启动初期，整个相机驱动中的各个模块都开始进行加载了，接下来我们依次介绍下：

首先是CRM的初始化，按照linux驱动模块的标准方法，会走到module_init宏声明的驱动结构体中的probe方法，这里是cam_req_mgr_probe方法，在该方法中主要做了以下几个事情：

```java
调用cam_v4l2_device_setup方法，创建并向系统注册用于管理所有子设备的v4l2_device。
调用cam_media_device_setup方法，创建并向系统注册media_device，并且创建了media设备节点，
        用于CSL枚举KMD中所有设备。
调用cam_video_device_setup方法，创建video_device，并将v4l2_device嵌入到该结构体中，
        紧接着，使用标准的video注册方法，创建了video0设备节点，
        其中将g_cam_ioctl_ops方法集作为了video0的扩展方法，
        CSL下发的有关Session/Link/Request的诸多操作都是通过该方法集来进行分发的，
        最后将video0 media_entity中的function赋值CAM_VNODE_DEVICE_TYPE，
        这样CSL便可以通过该function判断出该节点便是CRM了。
调用cam_req_mgr_util_init方法，其中初始化了一个cam_req_mgr_util_hdl_tbl，
        该结构体中存在一个handle数组，而每一个handle主要用于存储Session、Link以及各个子设备的
        相关信息，后期在整个图像采集的过程中，都是通过该结构体来找对应的操作实体，
        进而采取相应的动作。
调用cam_req_mgr_core_device_init方法，该方法中，会去创建并初始化一个cam_req_mgr_core_device
        结构体，作为全局变量g_crm_core_dev存在于整个框架中，
        而该结构体中主要包含了用于存储创建的Session的session_head链表，
        以及用于保护Session临界资源的crm_lock。
```

其次，是Cam Sync的初始化，整个流程最终会走到驱动结构体中的probe方法中，这里是cam_sync_probe方法，在该方法中主要做了以下几个事情：

```java
创建sync_dev结构体，该结构中通过一个sync_table_row数组来维护着所有的sync objects。
调用cam_sync_media_controller_init方法，用于创建media_deivce设备，
    并且创建了media设备节点，提供给CSL枚举子设备的能力。
调用v4l2_device_register方法，创建并像系统注册一个v4l2_device结构体，
    其中用于ioctl的方法集是指向的g_cam_sync_ioctl_ops，
    一旦CSL有创建/注册sync objects需求的时候，便会最终走到该方法中，从而实现相应的功能。
调用video_register_device方法，生成video1设备节点，暴露控制接口给CSL。
调用cam_sync_init_entity方法，
    将video1中的meida_entity中function字段赋值CAM_SYNC_DEVICE_TYPE，
    这样在UMD就可以通过相应的media节点枚举出该模块。
```

以上两个模块都是具有独立的video设备节点的，但是对于子设备而言，由于代表着相应的硬件设备，同时需要嵌入到整个框架中才能正常运行，所以高通将其抽象成了v4l2_subdev来进行管理，这里主要还是介绍两个比较有代表性的子模块，ISP以及Sensor。

首先来看下ISP的初始化阶段，在其相应的probe方法cam_isp_dev_probe中做了如下几个事情：

```java
调用cam_subdev_probe方法，在该方法中，会去注册一个v4l2_subdev，
    并且将其挂载到CRM中的v4l2_device上，同时还创建了一个node，
    并且存入了v4l2_subdev中的token中，方便以后进行读取，另外，将方法集赋值为cam_subdev_ops，
    最后，创建了该v4l2_subdev内部的media_entity， 
    并且为其function字段赋值为CAM_IFE_DEVICE_TYPE，这样也方便在枚举子设备时分辨出当前节点
    代表着isp模块。
调用cam_isp_hw_mgr_init方法，该方法用于初始化isp中的硬件模块。
调用cam_isp_context_init方法，该方法中会初始化node，在node内部创建一定数量的context，
    用于后期的状态维护，并且为每一个context都配置了状态机，
    以及子状态机来用于管理整个isp模块。
```

其次来看下Sensor模块的初始化，在其相应的probe方法cam_sensor_driver_i2c_probe中主要做了以下几个事情：

```java
调用cam_sensor_parse_dt方法获取dts中定义的硬件信息。
调用cam_sensor_init_subdev_params方法，该方法中会创建v4l2_subdev，
    然后挂载到CRM中的v4l2_device中，并且将sensor的私有方法集
    cam_sensor_internal_ops赋值给v4l2_subdev结构体中的ops，这样一旦操作相应的子设备节点，
    便最终会走到该方法集中，关于Sensor的一些操作便可以放到这个里面进行处理。
    最终将创建的v4l2_subdev中的media_entity中functon赋值为CAM_SENSOR_DEVICE_TYPE，
    方便CSL进行枚举Sensor设备。
```

通过上面的两个子设备的初始化代码梳理，不难发现，并没有进行设备节点的创建，那关于节点的创建动作发生在哪一个阶段呢？ 为了解决这个疑问我们不得不先介绍下linux两个宏定义，一个是module_init，另一个便是late_initcall，两者都是为了声明初始化函数，但是执行时间有一个先后顺序，而late_initcall一般在所有module_init定义的方法都运行完成之后才会被运行，而针对所有子设备的节点的创建便是在这里完成的，在该方法中主要做了以下工作：

```java
调用cam_dev_mgr_create_subdev_nodes方法，
    而在该方法中会去调用v4l2标准方法v4l2_device_register_subdev_nodes来统一创建挂载在
    CRM中v4l2_device下的子设备节点。
至此，整个KMD框架便初始化完成，现在便静静等待CSL下发请求。
```

## 4、处理UMD CSL请求

整个KMD的初始化动作在linux内核启动的时候完成的，要稍早于CamX-CHI整个框架的初始化，所以在CamX-CHI进行初始化的时候，KMD框架的各个资源节点都已准备妥当，接下来我们就以CamX-CHI的初始化开始详细描述下整个KMD处理来自CSL请求的流程。

1.获取模块资源
在CamX-CHI初始化的时候，并不知道内核驱动部分是个什么状态，所以需要打开所有的media设备节点来枚举查询每一个驱动模块。

首先，打开media0，根据CAM_VNODE_DEVICE_TYPE信枚举并找到KMD框架中的CRM模块，并调用标准open方法来打开该设备，该动作最终会调用到cam_req_mgr_open方法，该方法主要做了以下几个工作：

```java
调用v4l2_fh_open方法，打开v4l2文件。
调用cam_mem_mgr_init方法，初始化了内存管理模块，为之后的缓冲区的申请与释放做好准备。
更新CRM状态为CAM_MEM_MGR_INITIALIZED。
```

在打开video0之后，会另起一个线程用于监听video的事件，这样就建立了与底层的双向通讯，而在此之前，需要通过ioctl方法将CSL需要监听的事件下发到驱动层，其中包括以下几个事件：

```java
V4L_EVENT_CAM_REQ_MGR_SOF/V4L_EVENT_CAM_REQ_MGR_SOF_BOOT_TS： 
    一旦底层产生的SOF事件，便会向CSL发送该事件。
V4L_EVENT_CAM_REQ_MGR_ERROR： 一旦底层产生了错误，会向上抛出该事件。
```

一旦CSL获取了CRM模块信息成功之后，便开始枚举查找各个子模块了，其中会先去打开Sensor子设备，获取硬件信息，并且存入CSL中，然后再依次获取其它诸如IFE/IPE等硬件子模块并获取各自的信息，并存入CSL中，为之后的数据流转做好准备。

以上动作都完成之后，便开始查询Cam Sync模块了，基本流程与CRM大致相同：

```java
调用open方法打开video1，该方法最终会调用内核部分的cam_sync_open方法，
    而该方法中会调用v4l2_fh_open方法，从而打开v4l2文件。
调用ioctl方法，订阅针对CAM_SYNC_V4L_EVENT_ID_CB_TRIG事件的监听 ，
    而对于该事件，一般是在子模块处理数据完成之后，会触发Cam Sync发送该事件至上层。
```

2.打开Session
好了，到这里，整个CamX初始化过程对于底层的请求都已经完成了，一旦用户打开相机应用之后，经过层层调用最终会去打开Session，进而调用video0的相应的ioctl方法传入CAM_REQ_MGR_CREATE_SESSION命令开始在驱动层打开Session的操作，而在驱动部分，会调用到CRM中的cam_req_mgr_create_session方法，在该方法中，会去创建一个用于代表session的handle，并将其存入全局静态变量hdl_tbl中。紧接着会去初始化该session中的link，其中该session管理着两个link数组，一个是用于初始化的links_init数组，一个是用于运行起来之后使用的links数组，这里的会首先初始化所有的links_init中的link，在使用的时候，会从该数组去取出一个空闲的link放入links中进行管理。

3.打开设备
在打开Session之后，随着Pipeline的创建，CamX会通过调用CSL中的相应Node的ioctl方法，下发CAM_ACQUIRE_DEV命令，来依次打开底层硬件设备，这里我们还是以ISP为例进行分析：

```java
一旦CSL调用了ISP设备节点的ioctl并且下发了CAM_ACQUIRE_DEV命令，
    并会通过层层调用一直调到__cam_node_handle_acquire_dev方法，
    在该方法中会首先去在ISP对应的node中的存储空闲context的队列中获取一个context。
紧接着，调用了cam_context_handle_acquire_dev方法，
    来通过调用之前获取的context的对用的状态机方法集中的acquire_dev方法来打开isp设备，
    而在该方法中，会调用cam_create_device_hdl方法，
    将当前session handle以及isp操作方法集存入存入hdl_tbl中，
    之后crm会通过该方法集操作isp模块。之后会将当前isp context状态更新为CAM_CTX_ACQUIRED，
    并且初始化了用于管理request的active_req_list/wati_req_list/
    pending_req_list/pending_req_list/free_req_list链表，
    并且将初始化好req_list都挂载到free链表中。
```

除了ISP，会根据不同的图像采集需求，打开不同的子设备，基本流程差不多，都是通过下发CAM_ACQUIRE_DEV命令来完成的，这里我们便不进行赘述了。

4.创建Link
在打开所有的子设备之后，紧接着需要将它们链接起来形成一个拓扑结构，方便各个子模块的管理。而这个动作还是通过调用CRM对应的ioctl下发CAM_REQ_MGR_LINK命令来完成的，该动作会经过层层调用，一直调用到CRM中的cam_req_mgr_link方法，接下来我们具体介绍下该方法的主要动作：

```java
调用__cam_req_mgr_reserve_link方法，在该方法中，
    首先会去从当前Session中的links_init数组中取出一个空闲的link，
    将其存入links数组，并且初始化其中的用于管理所有的request的in_q队列。
调用cam_create_device_hdl，创建link对应的handle，并且存入hdl_tbl中。
调用__cam_req_mgr_create_subdevs方法，初始化用于存储处于当前Link中的所有子设备。
调用__cam_req_mgr_setup_link_info方法，
    该方法首先会去调用该link中的所有子设备的get_dev_info方法来获取设备信息，
    然后会去依次调用hdl_tbl中的链接在此Link上的所有子设备的setup_link方法，
    来连接子设备，同时也将CRM的一些回调方法通过该方式注入到子设备中，
    使其具有通知CRM的能力。
更新该Link状态为CAM_CRM_LINK_STATE_READY，并且创建了一个工作队列用于操作的异步处理。
```

5.开启数据流
一旦整个Link创建完成之后，便可以开启数据流了，该动作通过CSL控制每一个子设备来完成，这里还是以ISP为例进行分析：

由于在CamX初始化过程中已经存有打开的ISP文件句柄，所有通过调用起iotcl方法下发CAM_START_DEV命令来通知底层ISP模块开始进行数据流程传输，该命令首先会走到node,然后通过node下发到context，然后调用当前context的状态机对应的start_dev方法，而在该方法中，会首先更新当前context状态为CAM_CTX_ACTIVATED，然后通过操作底层硬件管理模块开始数据流的处理。

除了ISP，还有Sensor/FLash等模块也是需要开启数据流，为之后的Request的下发做好准备。

6.下发Request
一旦开启了整个数据处理流程，便可以接收Request请求了，而该动作依然还是通过CRM来完成，调用其ioctl方法，传入CRM_WORKQ_TASK_SCHED_REQ命令，该动作最终会到达内核CRM中的cam_req_mgr_schedule_request方法，而方法会将此次任务封装成task交由工作队列进行异步处理，而在工作队列中最终会调用其回调方法cam_req_mgr_process_sched_req，该方法主要做了如下工作：

```java
取出该request从属的link，并且将其中的in_q取出，
    找到一个空闲的slot，并将该slot便作为此次request在内核中的实体。
更新该slot的状态为CRM_SLOT_STATUS_REQ_ADDED，并且将link中的open_req_cnt计数加1。
```

从上面的梳理不难看出，下发Request的操作并不复杂，其中并没有一个实际的Request下发到子设备的动作，所以很自然地会产生一个疑问，没有下发Request的动作，那CRM是如何来驱动整个Request的流转的呢? 所以接下来我们来进一步介绍下，整个Request的流转机制。

7.子设备处理数据
当CSL下发Request到KMD之后，便会进入到DRQ中进行流转，通过之前对于CamX的学习，想必大家应该已经熟悉了整个DRQ的运行机制，DRQ的每一个Node都会有一定依赖关系，一旦某个Node满足依赖关系之后，便会调用其ProcessRequest开始进行此次的Request处理，而该动作会将图像数据的以及配置信息打包，通过调用ioctl方法下发CAM_CONFIG_DEV到具体的子设备节点来将配置写入KMD子设备中，而一旦子设备收到此次请求之后，会调用当前context的状态机所对应的config_dev方法，接下来我们具体介绍下其中的所作的动作：

```java
将此次配置信息包括图像数据放入硬件管理模块中，但是此时并不进行处理，等待处理指示。
将此次Request信息封装一下，通过调用之前setup_link传入的回调方法集中的add_req方法通知CRM，
    而在CRM中，会首先通过一系列的判断，
    如果条件满足了便将此次request对应的slot状态更新为CRM_REQ_STATE_READY，
    并将该request存入pending队列中。
```

由上面的分析，发现该过程中并没有进行实际的硬件配置或者处理，此时便需要等待SOF的事件，来驱动接下来的操作，而SOF事件是ISP来通知CRM的，具体流程如下：

```java
EPOCH中断产生，触发回调方法__cam_isp_ctx_notify_sof_in_activated_state，
    在该方法中会封装事件，并且通过调用CRM中传入的回调方法notify_trigger将事件发送至CRM中。
一旦CRM收取到SOF事件，便会去找到对应的满足要求的request，
    并且调用__cam_req_mgr_process_req方法通知相应的子设备进行配置。
最后ISP会将此次SOF事件通过V4L2 event机制发送至UMD，通知到CSL中。
```

8.数据操作完成
当CamX中的各自Node完成了下发Request的操作之后，便会等待数据的处理完成，一旦完成便会触发buf_done中断，进而告知context，最终会调用cam_sync_signal方法来通知Cam Sync，而在Cam Sync中会通过子设备调用cam_sync_signal时传入的sync_id在sync_table_row找到相应的sync object，最终通过event机制，将此次处理完成的事件传入UMD CSL中，进而进行后续处理。

等到最后一个Node处理完成之后，此次Request的处理便宣告完成。

之前QCamera & Mm-Camera架构采用的相机驱动比较简单，主要就承担了硬件的上下电以及读写寄存器的任务，并且控制方向都是从上到下，并且控制逻辑由UMD负责。但是随着时代的发展，相机硬件模块越发复杂，所以用于直接控制硬件的驱动层也需要承担更为复杂的控制任务，通过上面的分析，我们可以看到，高通重新设计了一套优秀的KMD框架，在其中加入了更多复杂的控制逻辑，以达到精细化控制底层硬件模块的目的，其中比较重要的是CRM对于子设备的横向控制，这样的好处很明显，降低了UMD控制驱动的难度，UMD只需要将请求通过V4L2框架中的设备节点下发至KMD中，之后便由KMD中的CRM来统一管理，适时地将请求下发给各个子设备，进而控制着底层硬件模块。



# 八、硬件层

## 1、简介

相机的硬件层，作为整个框架的最底层，通过硬件模块接收来自客观世界的真实光影效果，将其转换为计算机所熟知的数字信号，并按照一定的数据格式向上源源不断提供成稳定并成像效果优秀的图像数据，整个部分复杂且高效，可以说是，一个优秀的硬件基础，就好比为整个相机框架的地基，拥有一个好的地基，便使得建造一座摩天大厦成为可能，接下来我们来详细介绍下，这部分各个组件的基本情况。

## 2、基本硬件结构

而今的相机硬件系统纷繁复杂，但是如果仔细深入研究的话，你会发现，其实核心组件无外乎镜头、感光器、图像处理器三大件，其中镜头用来聚光，感光器件用于光电转换，而图像处理器用来加工处理图像数据，接下来我们就以这三个组件开始展开对于相机系统的世界的探索之旅。

![hardware](assets/hardware.png)

1. 镜头(Lens)
  将时间的转盘向前波动一下，让我们回到各自的小学时代，那时候老师给我们都布置了一个家庭作业，任务是制作一个小孔成像的简单模型，这个简单模型便是我接触的最原始最简单的成像系统，但是那是我一直有一个疑问，成像为什么那么模糊，这个疑问在我接触到真正的相机之后才得以解开，原来一切都是光线惹的祸。

  根据小孔成像原理，小孔的一端是光源，另一端是成像平面，光经过小孔，入射到平面上，无数个光线都入射到这个平面上，便形成了光源的像，但是有一个问题，就是光线是按照发散路径向四周蔓延开来，光源某点所发出的某一束光线通过小孔后会到达成像平面的某一点上，但是很显然，该点也会接收来自另一个光源上的点所发出的另一束光线，这样就形成的光的干扰，进而影响了最终的成像效果。所以为了改善这个问题，镜头便被发明出来，而镜头其实我们日常生活中接触的凸透镜，其根本目的就是为了解决光线互相干扰的问题，其原理就是通过凸透镜的折射原理，将来自同一点的光线，重新汇聚至一点，从而大幅度提升了成像效果。而这里的重新汇聚的一点便是光源那点在透镜后的像点，而由于随着光源点的不断变换，其像点会相应的变化，所以我们常常将来自无限远处的光线，通过透镜之后汇聚而成的那个点称为该镜头的焦点，而焦点到透镜中心的距离，便称为焦距，一旦透镜制作完成，焦距便被确定下来。

2. 光圈快门
  对于一个制作完成的镜头，无法随意调整镜头的直径，所以便在其中加入了一个叫做光圈的部件，该部件一般采用正多边形或者圆形的孔状光栅，通过调整光栅开合大小进而控制这个镜头的瞬时进光量，然而针对总的进光亮的控制仅仅依靠光圈也是不够的，需要再用到另一个叫做快门的部件，它主要决定着曝光的时长，最初的快门是通过调整镜头前的盖子的开关来进行实现，随着时代的进步，现在快门衍生出了多个实现方式，其中包括机械快门，它是作为一种只使用弹簧或者其他机械结构，不靠电力来驱动与控制速度的快门结构，电子快门，该快门结构通过马达和磁铁在电力驱动的作用下进行控制。电子断流快门，一种完全没有机械结构的快门结构，具有高快门速率和很快的影响捕捉频率，但是缺点是容易产生高光溢出现象。

  光圈控制着瞬时进光量，快门控制着曝光时间，通过两者的共同合作，完成了控制光线进入量的目的，进而进一步真实再现了场景的光影效果，避免了过度曝光的情况发生，极大的提升了整个提成像质量。

3. 对焦马达
  正如之前所说，入射光线会在通过透镜之后以锥形路径汇聚到一点，该点叫做像点，之后再以锥形发散开去，而所有的相同距离发射的光线，都会汇聚到各自的像点上时，便形成了一个都是像点组成的一个平面，而这个平面一般叫做像平面，又由于这个平面是所有像点所汇聚而成的，所以该平面是成像清晰的，而现如今的对焦的本质便是通过移动透镜，使像平面与感光器件平面重合，从而在感光器件上形成清晰的像。一般来讲，对焦可以通过手动移动透镜完成，但是更一般地，是通过一个叫做对焦马达的器件来完成。除了手动调整镜头进而完成对焦操作外，现在比较主流的方式是通过自动移动透镜进而完成对焦动作，随着技术的不断发展，而今的对焦又发展出了自动对焦策略，其中包括了相位对焦和对比度对焦。其基本原理是前后调整镜头使像平面与感光器感光平面重合，从而形成清晰的成像效果。另外，针对更为复杂的相机系统，为了获得更加优秀的成像质量，一般都会采用多个透镜组合来实现，一来可以消除色差，二来可以通过马达调整透镜间的距离，来动态的修改整个透镜组的焦距，从而满足更加复杂场景下的成像需求。

4. 感光器(Sensor)
  正如之前所讲，透镜的作用是为了汇聚光线，从而形成像平面，但是如何将这个所谓的像平面转换成计算机所熟知的图像信息呢？这就需要用到这里的感光器了，感光器并不是现代社会的专有发明，其实早在19世界初期的欧洲便有了这个概念，一位名叫尼埃普斯的法国人通过使用沥青加上薰衣草油，再以铅锡合金板作为片基，拍摄了从他家楼上看到的窗户外的场景，名叫《鸽子窝》的照片，而这里的沥青混以薰衣草油便是一种简单的感光物质，从这开始感光技术开始进入快速发展期，在1888年，美国柯达公司生产出了一种新型感光材料，柔软且可卷绕的胶卷，这是感光材料的一个质的飞跃，之后1969年在贝尔实验室，CCD数字感光器件被发明出来，将整个感光技术推入了数字时代，随后技术的不断革新，便于大规模批量生产的CMOS应运而生，将成像系统往更小更好的方向推进了一大步。随着CMOS的技术不断发展，优势明显的它渐渐取代了CCD，成为相机系统的主流感光器件。

5. 滤光片(IR Filter)
  由于感光材料的特性所致，它会感受除了可见光波长范围内的光线，比如部分红外光，由于这部分红外光是不可见的，所以对于我们而言没有实际的用处(当然，这也不绝对，有的情况就是需要采集红外光的信息，比如夜视照相机)，并且可能会干扰之后的ISP的处理，所以往往需要使用一个用于过滤红外光，避免红外光线干扰，修正摄入的光线的滤片，一般分为干涉式的IR/AR-CUT(在低通滤波晶片上镀膜，利用干涉相消的原理)和吸收式的玻璃(利用光谱吸收的原理)。

6. 闪光灯(Flash)
  针对某些特殊场景，比如暗光环境下拍摄需求，此时由于光线本身较少，无法完成充分的感光操作，但是为了获取正常的拍摄需求，往往需要通过外部补光来作为额外的光照补偿，基于此，闪光灯便应运而生，对于手机而言，其主要分为氙气灯与LED灯两种，由于LED闪光灯具有功耗较低、体积较小的优势，作为手机闪光灯的主流选择。另外，现在很多手机采用了双色闪光灯的策略，双色闪光灯可以根据环境的需要调节两灯发光的强度，可以更为逼近自然光的效果，相比单闪光灯强度有所提升，另外色温也较普通双闪光灯要更为准确，总体来讲效果较好。

7. 图像处理器(ISP)
  一旦当感光器件完成光电转换之后，便会将数据给到图像处理器，而ISP第一步需要做的便是去掉暗电流噪声，何为暗电流噪声呢？这要从感光器件说起，针对CCD/CMOS而言，通常并不是全部都用于感光，有一部分是被专门遮挡住，用于采集在并未感光的情况的暗电流情况，通过这种方式消除掉暗电流带来的噪声。

  对于镜头的各处的折射率不同的属性，会随着视场角的慢慢增大，能够通过镜头的斜光束慢慢减少，从而产生了图像中心亮度较边缘部分要高，这个现象在光学系统中叫做渐晕，很显然这种差异性会带成像的不自然，所以ISP接下来需要对于这种偏差进行修正，而修正的算法便是镜头阴影矫正，具体原理便是以图像中间亮度均匀的区域为中心，计算出个点由于衰减带来的图像变暗速度，从而计算出RGB三通道的补偿因子，根据这些补偿因子来对图像进行修正。

  随后，由于感光器件针对光线都是采用红、绿、蓝三基色进行分别采集而成的，所以数据一般会呈现出类似马赛克的排布效果，此时便需要完成去马赛克处理，基本原理便是通过一定的插值算法，通过附近的颜色分量猜测该像素所缺失的颜色分量，力争还原每一个像素的真实颜色效果，从而形成一个颜色真实的图像数据，而此时的数据格式便是RAW数据格式，即最原始的图像数据。

  当感光器进行光电转换的过程中，每一个环节都会产生一定的偏差，而这个偏差到最后便会以噪声的方式表现出来，所以接下来需要对于这个无关信息–噪声进行一定的降噪处理，当前主要采用了非线性去噪算法，比如双边滤波器，在采样时不仅考虑了像素在空间距离上的关系，同时还加入了像素间的相似程度考虑，从而保持了原始图像的大体分块，对于边缘信息保持良好。

  进一步降低了噪声之后，ISP需要对于图像白平衡进行处理，由于不同场景下的外界色温的不同，需要按照一定的比例调整RGB分量的值，从而使得在感光器中，白色依然是呈现白色的效果。白平衡可以采用手动白平衡，通过手动调整三个颜色分量的比例关系，达到白平衡的目的，而更一般地采用了自动白平衡的处理，这里ISP就承担着自动白平衡的使命，通过对当前图像进行分析，得到各颜色分量的比例关系，进而调整其成像效果。

  调整好图像白平衡后，需要进一步地调整颜色误差，这里的误差主要由于滤光片各颜色块之间存在颜色渗透所导致，一般在Tunning过程中会利用相机模组拍摄的图像与标准图像相比较得到的一个矫正矩阵，ISP利用这个矩阵来对拍摄的图像进行图像颜色矫正，从而达到还原拍摄场景中真实颜色的目的。

  以上简单罗列了下，图像处理器的几个基本功能，虽然每个厂商所生产的ISP都不尽相同，但是基本都包括了以上几个步骤，由此可见，图像处理器是用来提升整个相机系统的成像效果的。

## 3、手机相机简介

![camera-hardware](assets/camera-hardware.png)

对于手机上的相机系统，受到尺寸以及功耗的限制，无法像专业相机那样，为了保证成像效果，可以的很方便地更换更大的镜头，加入更大尺寸的CCD/CMOS感光器件，可以放入更加强大的图像处理模块，所以留给手机的发挥空间并不是很大，但是即便如此，各大手机厂商依旧在有限的空间和续航能力下，将相机系统做到了在某些领域媲美专业相机的地步，接下来我们来简单介绍下这套小体积但具有大能量的相机系统。

如图所示，手机的相机系统可以分为两个部分，一个是相机模组，一个是图像处理器ISP，相机模组是用来进行进行光电转换的，而图像处理器正如之前所介绍那样是用于图像处理的，接下来我们分别来看下，两者在手机端是如何运行的。

**1、相机模组**
由于受到体积的限制，手机相机模组往往做得十分精致小巧，里面主要包括了镜头、对焦马达、滤光片以及感光器(Sensor)。

手机中的镜头，一般为了消除色差都会采用多个透镜的组合，手机中的镜头也不例外，其材质多是玻璃和塑料的组合，对于塑料镜头而言，成本较低，适合用于低端产品中的相机系统，而玻璃一般成像质量较高，但是成本也稍高于塑料镜头，所以往往用于一些追求成像质量的手机中，同时其中，镜头主要存在以下几个参数：

1. 视场角FOV，该参数表明了通过镜头可以成像多大范围的场景，一般FOV越大就越能看到大范围的景物，但是有可能会带来严重的畸变，通常使用后期的畸变矫正算法来修正大FOV所带来的畸变。
2. 焦距F ，规定所有平行于透镜主轴的光线汇聚到的那点叫做焦点，而焦点到透镜中心的距离便是这里的焦距，一般焦距越大，镜头的FOV也就越小。而越短的焦距，往往FOV越大。
3. 光圈值f，通过镜头焦距与实际光圈的直径比值来指定，该值越小，说明进光量也就越大，手机镜头一般采用f/2.0的固定光圈。

紧接着是对焦马达，这部分在手机中主要采用的是音圈马达(VCM)，而为了方便调整镜头，一般会将整个镜头集成在马达模组中，主板通过I2C总线传输指令，进而驱动马达的移动调整镜头达到对焦或者变焦的目的，这里我们简单介绍下音圈马达。

音圈马达在电子学中被称为音圈电机，之所以被称为音圈，是因为其实现原理与扬声器类似，都是在一个永久磁场内部，通过改变马达内线圈的直流电流大小，来控制弹簧片的拉升位置，进而带动镜头上下运动，达到对焦或者变焦的目的，由于具有着高灵敏度与高精度的特点，使之成为手机的主流对焦组件。

在手机端，对于音圈马达的使用一般分为两种模式，一种是变焦，一种是对焦，两者原理和目的都不一样。

1. 变焦: 通过马达调整镜头组中某一个透镜的移动，进而改变整个镜头的焦距，引起视场角的变化，从而实现对于景物的放大缩小的目的，这种方式便是我们常说的光学变焦，这种变焦手段的优点是在放大景物的过程中，不会损失图像细节，但是缺点也很明显，受到体积的限制，无法进行大范围的光学变焦，所以手机厂商一般采用光学与数字变焦的组合方式，达到高范围的变焦目的。
2. 对焦: 通过音圈马达直接前后移动整个镜头，使物体的像平面与感光器的感光平面重合，进而得到一幅清晰的图像，这种方式正是对焦的过程。其目的是为了获得清晰的图像。

光线在经过了镜头之后，会首先进入到下一个组件–滤光片，该部分会针对光线做进一步处理，主要有两个目的：

1. 过滤红外线: 由于感光器会感受到部分不可见的红外线，进而干扰后面的图像处理效果，所以需要通过滤光片，将这部分红外线过滤掉，只让可见光透过。
2. 修正光线: 光线通过透镜之后，并不都是平行垂直射向感光器的，还有很多并非直射的光线，很显然如果不对其进行拦截，会对感光器产生一定的干扰，所以滤光片利用石英的物理偏光特性，保留了直射的光线，反射掉斜射部份，避免影响旁边的感光点，进一步提升成像效果。

经过滤光片的过滤与修正，此时入射的光线具有一定的稳定性，此时就需要通过这个相机体系的核心感光器来进行光电转换了。

手机端的感光器主要有CCD与CMOS，但是由于成本较高，体积较大，CCD在手机端已经用的不多了，CMOS成为了这个领域的主流感光器，手机端的CMOS依然采用了三层结构，微透镜/滤光片/感光层，具体定义如下：

1. 微透镜层主要用于扩展单个像素的受光面积。
   滤光片采用的事Bayer模式，类似与RGB模式，都是采用RGB几个颜色分量来分别度量每一个像素的三通道
2. 灰度值，但是基于人眼对于绿色更为敏感的基本规律，Bayer模式进一步强调了绿色分量，从而将绿色分量分别定义了Gr以及Gb，用于更好地表达图像的色彩和亮度。
3. 感光层，用于将光子转换成电子信号，在经过放大电路以及模电转换电路，将其转换成数字信号。

其感光层的核心便是一个个感光二极管，每一个二极管边上都包含了一个放大器和一个数模转换电路。由于每一个感光元件都有一个放大器，虽然在一定程度上加快的速度的读取，但是却无法保证每一个放大器的放大效果一致，所以这种设计会带来可能的噪声。另外，由于CMOS在每一个二极管旁都加入了额外的硬件电路，势必会造成感光面积的缩小，所以这种设计会影响整体感光效果，这种设计被称为前照式，为了解决该问题，CMOS厂商推出了背照式设计，这种设计将感光像素与金属电极晶体管分别放置于感光片的两面，提高了像素占空比，增加了光线感应效率，增加了像素数量，改善了信噪比，极大的提升了成像效果。

**2、图像处理器**
手机端的图像处理器的实现流程基本和非手机端的相机系统中类似，对于高通平台的ISP，其中主要包括了诸如IFE/BPS/IPE/JPEG等硬件模块，他们分别担任了不通过图像处理任务，接下来我们一一简单介绍下：

1. IFE(Image Front End)： Sensor输出的数据首先会到达IFE，该硬件模块会针对preview以及video去做一些颜色校正、下采样、去马赛克统计3A数据的处理。
2. BPS(Bayer processing segment): 该硬件模块主要用于拍照图像数据的坏点去除、相位对焦、 去马赛克，下采样、HDR处理以及Bayer的混合降噪处理。
3. IPE(Image processing engine): 该硬件主要由NPS、PPS两部分组成，承担诸如硬件降噪（MFNR、MFSR）、图像的裁剪、降噪、颜色处理、细节增强等图像处理工作。
4. JPEG: 拍照数据的存储通过该硬件模块进行jpeg编码工作。

相对于专业相机而言，手机相机的受众并不了解太多专业的摄影学知识，但是这类群体具有一个明显不同于专业相机受众的特点，那就是比较关注相机的便携性和可玩性，其中便携性不用多说，整体手机相机的都是以小巧著称，但是可玩性方面，各大手机厂商也是煞费苦心，采用了很多策略来扩展了相机的可玩性，其中多摄便是一个比较典型的例子。

早期的手机相机，一般都是单独的后摄走遍天下，其功能比较单一，之后随着时代的发展以及年轻用户日益增多，对于自拍的需求愈发强烈，其中对于该领域的技术也有所突破。所以手机厂商便顺势推出了双摄模式，在手机前面额外加入一个相机模组来主要用于自拍，其中还在ISP中创新性地加入了美颜算法，进而大幅提升了自拍图像效果。紧接着，手机厂商将多个模组集成到手机上，进而满足了多个场景的拍照需求，接下来简单介绍下，多摄相机系统。

现如今的手机相机，往往采用了多个摄像模组，有专门的用于拍摄微缩景观的微距模组，也有专门拍摄广角场景的广角模组，也有为了满足特定需求开发的双摄系统，由于双摄技术的飞速发展，而今已经产生了很多中成熟的方案。

双摄技术顾名思义，是采用了两个摄像头模组分别成像，并通过特定的算法处理，融合成一张图像，达到特定成像需求的目的。普遍地，现在双摄方案主要用于实现背景虚化、提升暗光/夜景条件下成像质量、光学变焦，接下来依次进行简单的介绍：

a) 背景虚化(RGB + RGB)
为了实现该目的，主要采用了两个RGB的相机模组，同时对景物进行成像，利用三角测量原理，计算出每个点的景深数据，依靠该系列数据，进行前景以及背景的分离，再通过虚化算法针对背景虚化处理，最终营造出背景虚化的成像效果。值得注意的是，这里由于三角测量的原理的限制，需要对两个相机模组进行标定，使得两者成像平面位于同一平面，并且保持像素对齐。

b) 暗光提升(RGB + MONO)
在较暗的环境中，往往拍摄出来的效果不尽如人意，所以手机厂商便采用了一个RGB和一个黑白相机模组(MONO)来提升暗光成像效果，具体原理是，由于黑白相机模组没有Bayer滤光片，所以在暗光条件下，可以获得更多的进光量，进而保存了更多的图像细节，再加之RGB相机模组的颜色份量的补充，这样就可以更好的保证了暗光下的成像质量，同样的由于同样需要对两个相机模组的成像进行融合，所以依然需要进行标定操作，使两个相机模组能够保持像素对齐。

c) 光学变焦(广角 + 长焦)
光学变焦，正如之前介绍的，完全可以在对焦马达中通过调整单个透镜进行焦距变换，从而实现变焦的目的，但是有受到体积的限制，往往无法从单个相机模组中得到更大的变焦范围，所以手机厂商就提出采用两个具有不同焦距(广角和长焦)的相机模组，共同实现光学变焦的目的，其原理是通过广角模组呈现大范围的场景，通过长焦模组看到更远的场景，在拍照是模组切换以及优秀的融合算法实现了相对平滑的变焦操作。

通过上面的介绍，我们可以看到一个相机系统是通过镜头、光圈快门、感光器以及图像处理器组成，而为了提高其成像质量，在发展过程中逐步加入了滤光片、对焦马达以及闪光灯等组件，同时为了将相机系统嵌入手机中，无法避免地对硬件进行了一定的裁剪，比如光圈往往摒弃了可调形式，采用了固定光圈，另外，由于体积以及续航限制，手机上主流感光器主要采用了CMOS，而对焦马达也由于体积限制，对焦范围也有所缩小。但是即便硬件受到不小的限制，通过这这几年图像处理芯片不断发展，以及算法的不断优化，手机相机系统其实正在逐步缩小与专业相机的差距，我相信在不久的将来成像效果手机相机完全可以媲美专业相机。



# 九、Android 相机架构总结

Android 相机体系庞大且复杂，在我刚开始接触到该框架的时候，如盲人摸象一般，一点一点地在代码的世界中探索，在很长的一段时间内，都只能局限于某一个特定的区域，而且在解决问题的过程中，虽然通过对代码的深入梳理，最终都会顺利解决难题，但是到最后依然缺乏一个对于整个框架的理解，正如管中窥豹一般，只见细节而无法把握全貌。但是进入现在的公司之后，通过与相机前辈的沟通，我发现框架思维能力尤为重要，针对整个框架结构需要做到掌控全局，这样在遇到问题的时候便可以迅速定位，此时再进行代码层面的深入研究，发现问题根源，进而达到最终解决问题的目的。

Android相机体系随处可见接口与实现相分离的设计思想，而之前提及的对于体系结构的梳理正是按照其接口的逻辑定义来完成，再结合其接口具体实现，进而完善整个框架体系的代码地图的构建，而在本人六年多的相机开发过程中，经历了多次的Android 相机的框架调整，接口演变，接下来以个人经历为主线，简单为整个相机架构做一个总结。

起初，首先接触到的相机框架部分便是驱动，那时接触的是高通MSM8953平台，该平台还是采用的QCamera & MM-Camera框架，底层驱动并没有负责复杂业务逻辑控制，而是主要用于控制上下电，以及数据流的开启以及停止等，并且依然使用的是vb2进行图像帧缓冲区的管理，但是现如今的7150，其驱动部分俨然发生了翻天覆地的变化，高通为了配合UMD的业务处理，为驱动设计了一套KMD的框架，包含了复杂的业务处理流程，并且数据的管理也摒弃了vb2，采用了新的管理手段，赋予了驱动更多的职能。

之后由于工作需要，进一步将工作重心过渡到Camera HAL层，开发的平台依然是8953平台上，当时采用还是QCamera & MM-Camera框架，在该平台上见证了HAL接口的演变，首先接触最多的便是HAL1接口，该接口使用起来比较简单，通过几个特定接口分别实现预览、拍照以及录像的功能，此时，谷歌已经意识到该接口具有一定的局限，所以自然而然地进行接口的升级，提出了HAL2接口，但是由于接口定义存在问题，所有很快谷歌便摒弃了该接口，迅速推出HAL3接口，并且一直沿用至今。HAL3接口相比于HAL1，优势明显，通过将所有的采集流程高度抽象为一个统一逻辑，所有的场景都可以通过这一统一逻辑进行扩展，使该接口具有很强的灵活性和扩展性，所以通过这几代HAL接口的演变，不难得出一个结论，那便是接口的定义需要高度抽象，而抽象的目的就是为了更好的灵活性和可扩展性，就单单这一点而言，HAL3接口可以说是成功的。

在后续HAL层的开发过程中，也见证了HIDL接口的诞生，在进行Android 8.0系统的升级过程中，发现谷歌将Camera Hal Module从Camera Service 解耦出来，放入一个独立的进程Camera Provider中进行管理，而该进程负责向外暴露HIDL接口，对内完成对其的实现，并且开始针对system分区以及vendor分区进行了严格的权限控制，该目的显而易见，那边是将平台厂商的实现与谷歌Framework相分离，这样便可以进行快速的迭代升级。

进入现在的公司之后，工作内容进一步扩大，涉及到了App部分，而这部分在之前也有所接触，但是并不深入，那个时候还是使用Camera Api v1接口，其定义和HAL1类似，但为了增加其灵活性和扩展性，之后谷歌提出了Camera Api v2接口，现在主要接触的便是该接口，通过简单的几个控制语句便可以实现图像的采集，使用起来比较简单，进一步降低了开发者的门槛，这也从侧面体现出了该接口定义的巧妙。同时，对于Camera Api v2的实现，是通过Camera Framework来完成的，而该层也有着一次不小的演变，刚开始Framework层并不是直接通过AIDL接口与Camera Service进行通信，而是通过一个JNI层来完成从Java到Native的转换，而Native部分作为客户端，保持对Service的通信。这种设计，很显然会比较臃肿，并且代码难以维护，所以之后由于AIDL接口的提出，谷歌直接将其加入到相机框架中，用于保持Framework与Service的通信，进而摈弃了JNI层，进一步减少了不必要的层级结构，保持了整个体系简洁性。

整个相机体系，经历了多次的发展，最终形成了而今的框架结构，一路走来，不难发现都是对于接口的升级，而其升级主要是更新其逻辑定义，完善其具体实现。