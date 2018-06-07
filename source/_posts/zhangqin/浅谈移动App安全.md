titile:浅谈Android App安全  
date：2018-06-06  17:00:00
category：  
zhangqin  
 tags:   

- 移动APP
- 安全

------------
近10年来，移动设备在消费领域和企业领域的应用中得到飞速的发展，全球的移动设备数量早已超越了世界人口总和。移动安全的问题也逐渐凸显，相对PC安全移动应用的安全起步很晚  

<!--more-->

# 浅谈移动App安全
从2007年苹果公司发布iOS1，2008年谷歌第一次I/O大会第一次对外发布安卓系统以来，移动终端数量迅速增长，移动应用也同样呈现飞速的增长。刚开始人们并未关注移动应用安全方面的问题，随着移动应用的普及移动应用的安全问题也逐渐凸显。近几年，各界媒体对移动应用的漏洞关注度越来越高，漏洞的产生不仅给用户设备和信息的安全产生影响，也给企业带来业务和声誉的损失。  
## 一、漏洞类型  
从OWASP(Open Web Application Security Project)数据来看，2017年的Mobile top 10主要有以下类型：  
1. Activity公开组件漏洞  
2. Broadcast Receier  
3. Service组件任意调用漏洞  
4. 运行其他可执行程序漏洞  
5. 应用反编译  
6. 硬编码敏感信息泄露  
7. 本地拒绝服务漏洞  
8. 外部存储设备信息泄露漏洞  
9. PendingIntent包含隐式Intent信息泄露漏洞  
10. Android App allowBackup安全漏洞  
可以发现主要包括组件漏洞、应用反编译、信息泄露漏洞三种类别。  
## 二、漏洞等级  
移动App漏洞危害等级标准按照CVE标准进行分类。CVE(Common Vulnerabilities & Exposures)是每个的MITRE公司开发的一个为公开的信息安全漏洞进行命名的国际标准。针对移动App漏洞的严重程度将其分为三个等级：高危、中危、低危。  
  
**高危：**此类安全漏洞存在较大的安全风险，可被直接利用，能够直接或潜在造成系统破坏，或对数据完整性、机密性或可用性造成潜在威胁。  
**中危：**此类安全漏洞可对应用产生一定程度的影响、破坏或无法直接利用，但能够协助攻击者发动更多的攻击。  
**低危：**并不构成直接的威胁，本身造成破坏性的概率较小，在与其它安全漏洞配合的情况下课发动更多的攻击。  
## 三、常见漏洞分析
### 1、组件漏洞
组件分为公有组件和私有组件，公有组件就是activity组件可以被外部程序调用，私有组件就是不能被其他程序启动或调用。因此在创建组件时，如果是私有的组件，android:exported属性一律设置为false.如果是公有的，就设置android:exported为true。不管公有的还是私有的组件，处理接收的intent时都应该进行验证的数据验证。公有组件防止信息泄露和接收外部数据时进行严格的处理。如果对私有组件没有进行相应的配置，可能导致组件被其他程序调用，敏感信息泄露，拒绝服务器攻击和权限绕过等漏洞。  
一个简单的PrivateActivity如下：  
  
	public class PrivateActivity extends Activity{
    private static fina String SAMPLE_STRING_TEST = "sample_string_test";
    @Override
    protected void onCreate(Bundle saveInstanceState){
        super.onCreate(saveInstanceState);
        setContentView(R.layout_activity_main);
        
        String value getIntent().getStringExtra(SAMPLE_STRING_TEST);
        String subValue = value.substring(0);
        Toast.makeText(getApplicationContext(),subValue,Toast.LENGTH_SHORT).show();
    }
}
在PrivateActivity中，从Intent中直接获取值后，并没有做任何异常处理，这可能会存在问题。如果PrivateActivity是私有的，并且能够保证传入到其中的Intent一定有值的话是没有安全问题的。如果存在一个如下的MainActivity：  
  
	public class MainActivity extends Activity{
		@Override
		protected void onCreate(Bundle saveInstanceState){
			super.onCreate(saveInstanceState);
			setContentView(R.layout_activity_main);

			if(intent.getBooleanExtra("allowed" , false)){
				CompontentName compontentName = new CompontentName(this , "PrivateActivity");
				intent.setCompontent(compontentName);
				this.startActivity(this.getIntent());
			}
		}
	}
这样有可能存在问题，如果PrivateActivity里面存在很重要的逻辑业务，那么攻击者可以通过MainActivity去控制PrivateActivity，进而控制PrivateActivity里面的逻辑走向。
###2、应用反编译
现在的Android APK加固防破解主要从以下几种实现方式：  
1).  代码混淆技术  
2). 签名比对技术  
3). NDK so动态库技术  
4). 多态加载技术  
如代码混淆技术，以Android Studio的Proguard为例，它会检测和移除封装应用中未使用的类、字段、方法和属性，包括自带代码库中的未使用项，还可以移除未使用的代码指令，以及用短名称混淆其余的类、字段和方法。 代码混淆过的代码增加了被逆向分析的难度。使用Android Studio在build.gradle中的混淆配置如下：  
  
	buildTypes {
        debug {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			 consumerProguardFiles 'proguard-rules.pro'
        }
        release {
            minifyEnabled true   //未true表示开启混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
配置后通过编辑proguard-rules.pro文件即可。Android官网提供的语法规则如下：  
  
	# Add project specific ProGuard rules here.  
	# By default, the flags in this file are appended to flags specified  
	# in /Users/liuqiang/Development/android-sdk-macosx/tools/proguard/proguard-android.txt  
	# You can edit the include path and order by changing the proguardFiles  
	# directive in build.gradle.  
	#  
	# For more details, see  
	#   http://developer.android.com/guide/developing/tools/proguard.html  
  
	# Add any project specific keep options here:  
  
	# If your project uses WebView with JS, uncomment the following  
	# and specify the fully qualified class name to the JavaScript interface  
	# class:  
	#-keepclassmembers class fqcn.of.javascript.interface.for.webview {  
	#   public *;  
	#}  
  
  
	#-ignorewarnings                     # 忽略警告，避免打包时某些警告出现  
	-optimizationpasses 5               # 指定代码的压缩级别  
	-dontusemixedcaseclassnames         # 是否使用大小写混合 混淆时不会产生形形色色的类名  
	-dontskipnonpubliclibraryclasses    # 是否混淆第三方jar  
	-dontpreverify                      # 混淆时是否做预校验  
	-verbose                            # 混淆时是否记录日志  
	-dontoptimize                       # 不优化输入的类文件  
  
	-keepattributes *Annotation*, SourceFile, InnerClasses, LineNumberTable, Signature, 	EnclosingMethod  
	-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*    #优化 混淆时采用的算法  
  
	-keep public class * extends android.app.Activity    # 未指定成员，仅仅保持类名不被混淆  
	-keep public class * extends android.app.Application  
	-keep public class * extends android.app.Service  
	-keep public class * extends android.app.View  
	-keep public class * extends android.app.IntentService  
	-keep public class * extends android.content.BroadcastReceiver  
	-keep public class * extends android.content.ContentProvider  
	-keep public class * extends android.app.backup.BackupAgentHelper  
	-keep public class * extends android.preference.Preference  
	-keep public class * extends android.hardware.display.DisplayManager  
	-keep public class * extends android.os.UserManager  
	-keep public class com.android.vending.licensing.ILicensingService  
	-keep public class * extends android.app.Fragment  
  
	-keep public class * extends android.support.v4.widget  
	-keep public class * extends android.support.v4.**    #  *匹配任意字符不包括.  **匹配任意字符  
	-keep interface android.support.v4.app.** { *; }    #{ *;}    表示一个类中的所有的东西  
	-keep class android.support.v4.** { *; }        # 保持一个完整的包不被混淆  
	-keep class android.os.**{*;}  
  
	-keep class **.R$* { *; }  
	-keep class **.R{ *; }  
  
	#实现了android.os.Parcelable接口类的任何类，以及其内部定义的Creator内部类类型的public final静态成员变量，都不能被混淆和删除  
	-keep class * implements android.os.Parcelable {    # 保持Parcelable不被混淆  
 	 public static final android.os.Parcelable$Creator *;  
	}  
  
	-keepclasseswithmembernames class * {     # 保持 native 方法不被混淆  
    	native ;  
	}  
  
	-keepclasseswithmembers class * {         # 保持自定义控件类不被混淆  
    	public (android.content.Context, android.util.AttributeSet);  
	}  
  
	-keepclasseswithmembers class * {         # 保持自定义控件类不被混淆  
    	public (android.content.Context, android.util.AttributeSet, int);  
	}  
  
	-keepclasseswithmembers class * {  
 	 public (android.content.Context, android.util.AttributeSet, int, int);  
	}  
  
	-keepclassmembers class * extends android.app.Activity { #保持类成员  
  	 public void *(android.view.View);  
	}  
  
	-keepclassmembers class * extends android.content.Context {  
  	public void *(android.view.View);  
  	public void *(android.view.MenuItem);  
	}  
  
	-keepclassmembers enum * {                  # 保持枚举 enum 类不被混淆  
   	 public static **[] values();  
    	public static ** valueOf(java.lang.String);  
	}  
  
	# Explicitly preserve all serialization members. The Serializable interface  
	# is only a marker interface, so it wouldn't save them.  
	-keepnames class * implements java.io.Serializable  
  
	-keepclassmembers class * implements java.io.Serializable {  
 	 static final long serialVersionUID;  
 	 private static final java.io.ObjectStreamField[] serialPersistentFields;  
  	private void writeObject(java.io.ObjectOutputStream);  
  	private void readObject(java.io.ObjectInputStream);  
  	java.lang.Object writeReplace();  
 	 java.lang.Object readResolve();  
	}  
  
	-libraryjars   libs/treecore.jar   #缺省proguard 会检查每一个引用是否正确，但是第三方库里面往往有些不会用到的类，没有正确引用。如果不配置的话，系统就会报错。  
	-dontwarn android.support.v4.**  
	-dontwarn android.os.**  
  经过代码混淆后的代码逆向分析后的代码可读性非常差，如下面代码片段：  
  
	public final class a{
		public static int a = 12;
		public static String b = "VER1";  
		private static ao a(int paramInt1 , int paramInt2 , String paramString){
				...
		}
	}
可见代码中的类名、变量、参数名都经过了混淆，很难分析出他们所代表的意义。不过即使这样，通过逆向工程中的静态分析方法和动态分析方法还是有可能破解。  

### 3、信息泄露
App敏感信息泄露可分为产品信息泄露和用户敏感信息泄露两类。前者指泄露后直接对企业安全造成影响或帮助攻击者尝试更多的攻击路径的信息；后者是指用户的隐私数据，可识别出自热人的信息。  
信息泄露的方式有很多种，如本地明文存储账号密码方式：  
  
	<?xml version='1.0'  encoding = 'utf-8'  
	standalone = 'yes' ?>
	<map>
		<string name = "username">name</string>
		<string name = "password">password</string>
		<boolean name = "isSave"  value = "value" />
		<boolean name = "isNewVersion" value = "false" />
	</map> 
使用http传输明文，例如使用burpsuit抓抓到如下形式的数据包：  
  
	get  /xxxxx/api/login/applogin?cellphone = xxxxxxx & password = xxxxxxxx http/1.1
	If-Modified-Since:Tue ,5, Jun ,2018  19:30:24 GMT+00:00
	User-Agent:  ...
	Host：  www.XXXXX
	...
可见密码和电话号码未加密，而且传输使用的是http，可以在传输数据包中直接获取明文的密码和电话号码。  
## 四、总结
移动App的种类很多，每个种类所面临的安全问题也有所差别。例如常规应用App、游戏类App和金融类App所关注的侧重点不一样，他们面临的安全威胁也不一样。但是这些App都会存在组件安全问题、信息泄露问题和反编译等安全问题，作为编程人员在App研发过程中需要注重安全因素的考虑，同时也需要提高用户的安全意识。