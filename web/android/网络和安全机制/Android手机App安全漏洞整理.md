APP安全漏洞整理

1.源码安全漏洞

1.1 代码混淆漏洞

当前APK文件的安全性是非常令人堪忧的。APK运行环境依赖的文件/文件夹 res、DEX、主配文件Lib 只有简单的加密或者甚至没有任何加密。诸如apktool这类工具可轻易将其破解，再配合其他例如dex2jar、jd-gui等工具基本可以做到：源码暴露、资源文件暴露、主配文件篡改、核心SO库暴露、暴力破解恶意利用等。因此需要对安卓代码进行代码混淆。

代码混淆(Obfuscated code)亦称花指令，是将计算机程序的代码，转换成一种功能上等价，但是难于阅读和理解的形式的行为。将代码中的各种元素，如变量，函数，类的名字改写成无意义的名字。比如改写成无意义的单个或多个字母，如a，Ac，甚至改写成“__”这样的符号，使得阅读的人无法根据名字猜测其用途。代码混淆并不能真正阻止反向工程，只能增大其难度。因此，对于对安全性要求很高的场合，仅仅使用代码混淆并不能保证源代码的安全，但是可以在一定程度上保护开发者的劳动成果。

1.2 Dex保护漏洞

Dex是Dalvik VM executes的全称，即Android Dalvik执行程序，相当于安卓中的.exe文件，Dex为Android应用的核心，保护不当容易被反编译，暴露程序重要信息，面临被植入广告、恶意代码、病毒等风险。

另外当使用DexClassLoader加载外部的 apk、jar 或 dex文件，当外部文件的来源无法控制时或是被篡改，此时无法保证加载的文件是否安全。加载恶意的dex文件将会导致任意命令的执行。

1.3 so保护漏洞

so库一般是程序里面核心代码块，通过Android提供的NDK技术将核心代码用安全性更高的C/C++语言实现并提供给Java层调用来保证程序核心代码的安全。高性能的代码一般都会采取C/C++实现，通过Android的NDK技术来让Java层直接使用。其安全性相对于Java会高很多，相对于Java代码来说其反编译难度要大很多，但对于经验丰富的破解者来说，仍然是很容易的事，可以通过暴力破解或国外高价工具来将其破解。应用的关键性功能或算法，都会在so中实现，如果so被逆向，应用的关键性代码和算法都将会暴露。

1.4 调试设置漏洞

如果在AndroidManifest.xml配置文件中设置了application属性为debuggable=“true”，则应用可以被任意调试，这就为攻击者调试和破解程序提供了极大方便。如果开启，可被Java调试工具例如jdb进行调试，获取和篡改用户敏感信息，甚至分析并且修改代码实现的业务逻辑，例如窃取用户密码，绕过验证码防护等。

2. 组件安全漏洞

2.1组件导出漏洞

组成Apk的四个组件，Activity，Service，Broadcast Receiver 和Content Provider，如果设置了导出权限，都可能被系统或者第三方的应用程序直接调出并使用。组件导出可能导致登录界面被绕过、信息泄露、数据库SQL注入、DOS、恶意调用等风险。

2.2 Activity组件漏洞

Activity是Android组件中最基本也是最为常见用的四大组件之一，是一个负责与用户交互的组件。Activity组件中存在以下常见的漏洞。

 (1)activity绑定browserable与自定义协议

activity设置“android.intent.category.BROWSABLE”属性并同时设置了自定义的协议android:scheme意味着可以通过浏览器使用自定义协议打开此activity。可能通过浏览器对app进行越权调用。

(2)ActivityManager漏洞

ActivityManager类中的killBackgroundProcesses函数，用于杀死进程，属于风险API。

还有通过ActivityManager被动嗅探intent。Intent嗅探脚本首先调用一个Context.getSystemService()函数，并传给它一个ACTIVITY_SERVICE标志的标识符，该函数返回一个ActivityManager类的实例，它使得该脚本能够与activity manager进行交互，并通过这个对象调用ActivityManager.getRecentTasks()方法。最后把intent相关的信息格式化成字符串返回出来。

2.3 Service组件漏洞

Service作为Android中四大组件之一，拥有重要的地位。Service具有和Activity一样的级别，只是没有界面，是运行于后台的服务。其他应用组件能够启动Service，并且当用户切换到另外的应用场景，Service将持续在后台运行。另外，一个组件能够绑定到一个service与之交互（IPC机制），例如，一个service可能会处理网络操作，播放音乐，操作文件I/O或者与内容提供者（content provider）交互，所有这些活动都是在后台进行。从表面上看service并不具备危害性，但实际上service可以在后台执行一些敏感的操作。

Service存在的安全漏洞包括：权限提升，拒绝服务攻击。没有声明任何权限的应用即可在没有任何提示的情况下启动该服务，完成该服务所作操作，对系统安全性产生极大影响。

2.4 Broadcast Receiver组件漏洞

Broadcast Receiver是“广播接收者”的意思，就是用来接收来自系统和应用中的广播。

（1）权限管理不当

Broadcast Receiver执行一些敏感操作时，会通过intent来传递这些信息，这种传递数据的方式是容易被恶意攻击的。在发掘broadcastreceiver中的漏洞时，最大的问题是确定输入是否可信，以及破坏性有多强。

        以下面这段代码为例（有缩略）：

        <receiver

android:name=”.broadcastreceivers.SendSMSNowReceiver”

android:label=”Send SMS”>

        …………

                 <actionandroid:name=”org.owasp.goatdroid.SOCIAL_SMS” />

        …………

        <uses-permissionandroid:name=”android.permission.SEND_SMS”/>

        …………

在上面这段代码中，滥用了一个没有进行适当权限保护的broadcast receiver，由于这个组件缺乏权限保护，使得攻击者可以在没有发送短信权限的情况下发送短信，这个漏洞的危害在于，攻击者可以在用户不知情的情况下发送短信去订购一个付费服务，或者泄漏设备中的信息。

（2）BroadcastReceiver导出漏洞

当应用广播接收器默认设置exported='true'，导致应用可能接收到第三方恶意应用伪造的广播，利用这一漏洞，攻击者可以在用户手机通知栏上推送任意消息，通过配合其它漏洞盗取本地隐私文件和执行任意代码。

（3）动态注册广播组件暴露漏洞

Android 可以在配置文件中声明一个receiver或者动态注册一个receiver来接收广播信息，攻击者假冒APP构造广播发送给被攻击的receiver，是被攻击的APP执行某些敏感行为或者返回敏感信息等，如果receiver接收到有害的数据或者命令时可能泄露数据或者做一些不当的操作，会造成用户的信息泄漏甚至是财产损失。

2.5 Content Provider组件漏洞

Content Provider为存储和获取数据提供统一的接口。可以在不同的应用程序之间共享数据。

（1）读写权限漏洞

Content Provider中通常都含有大量有价值的信息，比如用的电话号码或者社交帐号登录口令，而确认一个content provider是否有能被攻击的漏洞的最好的办法，就是尝试攻击它一下。

可以用drozer来寻找一些不需要权限的contentprovider:

dz> runapp.provider.info –permission null

这条命令能列出所有不需要任何读写权限的Content Provider，然后找到相对应的包，去访问给定包存放在它的Content Provider中的数据。如果一些Content Provider的URI不需要读权限，那就可以通过drozer工具提取其中的数据。在某些情况下，设置和执行读写权限不当，也会将ContentProvider中的数据暴露给攻击者。除了提取数据，对于写权限管理不当的Content Provider还可以向其中写入数据，使得攻击者可以将恶意数据插入到数据库中。

（2）Content Provider中的SQL注入漏洞

和Web漏洞类似，安卓APP也要使用数据库，那就也有可能存在SQL注入漏洞。主要有两类，第一类是SQL语句中的查询条件子语句是可注入的，第二类是投影操作子句是可注入的。使用drozer可以很容易的找出查询条件子句可注入的content provider。

dz> runapp.provider.query [URI] –selection “1=1”

也可以使用其他恒为真的值，例如“1-1=0”，“0=0”等等。如果APP存在SQL注入漏洞，那么输入这行指令后就会返回数据库中的整张表。

（3）Provider文件目录遍历漏洞

当Provider被导出且覆写了openFile方法时，没有对Content Query Uri进行有效判断或过滤。攻击者可以利用openFile()接口进行文件目录遍历以达到访问任意可读文件的目的。

2.6 Intent组件漏洞

Intent（意图）主要是解决Android应用的各项组件之间的通讯。

（1）隐式意图调用漏洞

封装Intent时采用隐式设置，只设定action，未限定具体的接收对象，导致Intent可被其他应用获取并读取其中数据。Intent隐式调用发送的意图可能被第三方劫持，可能导致内部隐私数据泄露。

（2）意图协议URL漏洞

intent schemeURLs(意图协议URL)，可以通过解析特定格式的URL直接向系统发送意图，导致自身的未导出的组件可被调用,隐私信息泄露。

2.7 WebView组件漏洞

WebView是一个基于webkit引擎、展现web页面的控件。

（1）Webview明文存储密码风险

Android的Webview组件中默认打开了提示用户是否保存密码的功能，如果用户选择保存，用户名和密码将被明文存储到该应用目录databases/webview.db中。而本地明文存储的用户名和密码，不仅会被该应用随意浏览，其他恶意程序也可能通过提权或者root的方式访问该应用的webview数据库，从而窃取用户登录过的用户名信息以及密码。

（2）Webview远程代码执行漏洞

Webview是Android用于浏览网页的组件，其包含的接口函数addJavascriptInterface可以将Java类或方法导出以供JavaScript调用，实现网页JS与本地JAVA的交互。由于系统没有限制已注册JAVA类的方法调用，因此未注册的其它任何JAVA类也可以被反射机制调用，这样可能导致被篡改的URL中存在的恶意代码被执行，用户手机被安装木马程序，发送扣费短信，通信录或者短信被窃取，甚至手机被远程控制。

（3）Webview绕过证书校验漏洞

客户端的Webview组件访问使用HTTPS协议加密的url时，如果服务器证书校验错误，客户端应该拒绝继续加载页面。但如果重载WebView的onReceivedSslError()函数并在其中执行handler.proceed()，客户端可以绕过证书校验错误继续访问此非法URL。这样将会导致“中间人攻击”，攻击者冒充服务器与银行客户端进行交互，同时冒充银行客户端与银行服务器进行交互，在充当中间人转发信息的时候，窃取手机号，账号，密码等敏感信息。

（4）未移除有风险的Webview系统隐藏接口

android webview组件包含3个隐藏的系统接口：searchBoxJavaBridge, accessibilityTraversal以及accessibility，恶意程序可以利用它们实现远程代码执行。需通过显示调用removeJavascriptInterface移除这三个系统隐藏接口。

（5）WebView忽略SSL证书错误

WebView调用onReceivedSslError方法时，直接执行handler.proceed()来忽略该证书错误。忽略SSL证书错误可能引起中间人攻击。  

3.数据安全漏洞

3.1 数据存储漏洞

（1）SharedPreferences漏洞

当使用getSharedPreferences打开文件，第二个参数设置为MODE_WORLD_READABLE或MODE_WORLD_WRITEABLE。当前文件可以被其他应用读取或写入篡改，导致信息泄漏或更严重的问题。

（2）File任意读写漏洞

如果开发者使用openFileOutput(String name,int mode)方法创建内部文件时，使用MODE_WORLD_READABLE或 MODE_WORLD_WRITEABLE模式，就会让这个文件变为全局可读或全局可写的。

3.2 数据加密漏洞

（1）明文数字证书漏洞

Apk中使用的数字证书可被用来校验服务器的合法身份，以及在与服务器进行通信的过程中对传输数据进行加密、解密运算，保证传输数据的保密性、完整性。明文存储的数字证书如果被篡改，客户端可能连接到假冒的服务端上，导致用户名、密码等信息被窃取；如果明文证书被盗取，可能造成传输数据被截获解密，用户信息泄露，或者伪造客户端向服务器发送请求，篡改服务器中的用户数据或造成服务器响应异常。

（2）AES/DES弱加密

在AES加密时，使用“AES/ECB/NoPadding”或“AES/ECB/PKCS5padding”的模式。ECB是将文件分块后对文件块做同一加密，破解加密只需要针对一个文件块进行解密，降低了破解难度和文件安全性。

（3）setSeed伪随机数漏洞

本地加密时如果使用SecureRandom中的setSeed方法设置种子将会造成生成的随机数不随机，使加密数据容易被破解。在SecureRandom生成随机数时，如果不调用setSeed方法，SecureRandom会从系统中找到一个默认随机源。每次生成随机数时都会从这个随机源中取seed。

3.3 数据传输漏洞

        （1）SSL通信服务端检测信任任意证书

        自定义SSL x509 TrustManager，重写checkServerTrusted方法，方法内不做任何服务端的证书校验。黑客可以使用中间人攻击获取加密内容。

（2）未使用HTTPS协议的数据传输

无线传输的数据能被第三方轻易截获，由于客户端与服务器之间的传输数据遵循通信协议指定的格式和内容类型，如果未使用加密措施，传输数据可被还原成网络层的数据包并进行解包分析，直接暴露用户的各种关键数据，例如用户名，密码等。加入了SSL（Secure SocketLayer）子层实现的HTTPS协议可确保数据在网络上加密传输，即使传输的数据被截获，也无法解密和还原。

（3）HTTPS未校验服务器证书漏洞

使用HTTPS协议时，客户端必须对服务器证书进行完整校验，以验证服务器是真实合法的目标服务器。如果没有校验，客户端可能与仿冒的服务器建立通信链接，即“中间人攻击”。仿冒的中间人可以冒充服务器与银行客户端进行交互，同时冒充银行客户端与银行服务器进行交互，在充当中间人转发信息的时候，窃取手机号，账号，密码等敏感信息。

（4）HTTPS关闭主机名验证

构造HttpClient时，设置HostnameVerifier时参数使用ALLOW_ALL_HOSTNAME_VERIFIER或空的HostnameVerifier。关闭主机名校验可以导致黑客使用中间人攻击获取加密内容。

（5）开放socket端口

app绑定端口进行监听，建立连接后可接收外部发送的数据。攻击者可构造恶意数据对端口进行测试，对于绑定了IP 0.0.0.0的app可发起远程攻击。

3.4 日志信息漏洞

（1）调试日志函数调用风险

调试日志函数可能输出重要的日志文件，其中包含的信息可能导致客户端用户信息泄露，暴露客户端代码逻辑等，为发起攻击提供便利，例如：Activity的组件名，是Activity劫持需要的信息；通信交互的日志，会成为发动服务器攻击的依据；跟踪的变量值，可能泄露一些敏感数据，输入的账号、密码等。

4.业务逻辑漏洞

4.1 权限漏洞

（1）全局文件可读写

APP在创建内部存储文件时，将文件设置了全局的可读权限。攻击者恶意读取文件内容，获取敏感信息，或恶意写文件，破坏完整性。

（2）敏感权限调用

在Manifest文件中调用一些敏感的用户权限，敏感行为包括发送、拦截短信，读取、修改通讯录、通话记录，拨打电话，发送地理位置，使用摄像头，访问浏览器历史记录等。函数调用这些敏感行为，可能导致用户隐私数据泄露，钓鱼扣费等风险。

（3）冗余权限

如果调用了非必须的权限，就会出现冗余权限，冗余权限可导致串谋攻击，串权限攻击的核心思想是程序A有某个特定的执行权限，程序B没有这个权限。但是B可以利用A的权限来执行需要A权限才能完成的功能。

4.2 业务漏洞

业务漏洞需要依靠机器和人共同检测，需要根据应用功能作用的不同来进行判断。机器可以检测一些通用的业务漏洞，例如广告、非授权下载、扣费短信等业务，而人工则判断应用在面向不同业务逻辑时产生的漏洞，例如登录验证不完善、不可信的敏感数据交付等