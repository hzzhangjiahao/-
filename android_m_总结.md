#Android M 总结

## 概览

### Android M有哪些行为变化？
---
#### 1. 运行时权限：
- 新的权限系统，用户现在可以在运行时直接管理应用的权限。简化了应用安装和更新的过程，用户可以在安装后单独地赋予或撤销应用的权限。

#### 2. 节电优化：
- ##### 睡眠模式：
如果用户拔电息屏并离开设备一段时间，设备会进入睡眠模式以节约电量。不过设备会以一个短暂的时间周期定时地唤醒一些常规的操作，使得应用同步能够执行，也能让系统处理任何挂起的操作。
- ##### 应用待命
当一个应用没有被用户活跃地使用时，比如用户在一段时间内没有碰它，系统可以认为它是闲置的，拔电状态下，系统会禁止改应用的网络访问、同步和任务。

#### 3. 可采用的存储设备
- 用户可以采用外部存储设备，例如SD卡，经过加密和格式化后使其表现得像内部存储一样，这样用户就可以将应用和它的私有数据在存储设备间移动。移动时系统会参照manifest中的`android：installLocation`属性。正因为应用的路径会移动，所以强烈建议使用`Context`和`ApplicationInfo`中的路径API来动态生成路径，不要使用硬编码路径或全限制路径。

#### 4. 移除了Apache HTTP Client Removal
- 如果你的应用不幸使用了它并且适配`Android2.3`及以上版本，最好用`HttpURLConnection`替代。

#### 5. 声音管理
- 不能再使用`AudioManager`直接设置音量或静音了，`setStremSlol()`被废弃，你应该转而使用`requestAudioFocus()`。同样的，`setStreamMute()`也被废弃了，改用`adjustStreamVolume()`并传入方向值`ADJUST_MUTE`或者`ADJUST_UNMUTE`。

#### 6. 通知
- 删除了`Notification.setLatestEventInfo()`方法，使用`Notification.Builder`类替代构造通知，为了反复更新通知，重用`Notification.Builder`实例，调用`build()`方法来更新通知实例。可以使用`adb shell dumpsys notification --noredact`打印通知对象中的文本。

#### 7. 相机服务
- 现在访问相机中共享资源的模式从`“先来先服务”`转变为`“高优先级有利”`，高优先级的进程将能更可靠地获得和使用相机资源。
- 低优先级的进程会被高优先级的进程驱逐，被驱逐后，在`Camera`API中会调用`onError()`，在`Camera2`API中会调用`onDisconnected()`。
- 切换用户也会让之前用户的相机客户端被驱逐。

其他变化不再细说，请查阅官方文档……


## 新的权限系统
---
M采用新的权限系统简化app的安装和更新流程。权限不在安装时一次性请求，而是在需要某个权限时，单独地弹窗向用户请求。
### 概览
- 权限按照功能分成权限组，例如，`CONTACTS`权限组包含读写联系人和配置信息的权限。只要用户授予app某权限组中的任一权限，下一次app申请该组的其他权限时，系统将会自动授予而不提示用户。
- app在安装时会自动被授予`PROTECTION_NORMAL`组中的权限，例如闹钟和连网。
- 当app需要某个权限时，系统会显示一个弹窗，并通过调用回调函数的方式通知用户是否授予该权限。
- app在执行某个需要权限的操作时，首先应该检查是否拥有该权限，如果没有则应请求用户授予。
- 要处理好权限没有被授予的情况。如果某权限只影响一个附加功能，app应禁用该功能，如果某权限对app的功能至关重要，app应禁用它的所有功能并提示用户需要授予该权限。
- 用户可以在任何时候取消app的权限，然而app并不会被通知到。所以，app在执行限制级的操作时必须确认它拥有需要的权限。

### PROTECTION_NORMAL
android . permission . ACCESS_LOCATION_EXTRA_COMMANDS          

android . permission . ACCESS_NETWORK_STATE

android . permission . ACCESS_NOTIFICATION_POLICY

android . permission . ACCESS_WIFI_STATE

android . permission . ACCESS_WIMAX_STATE

android . permission . BLUETOOTH

android . permission . BLUETOOTH_ADMIN

android . permission . BROADCAST_STICKY

android . permission . CHANGE_NETWORK_STATE

android . permission . CHANGE_WIFI_MULTICAST_STATE

android . permission . CHANGE_WIFI_STATE

android . permission . CHANGE_WIMAX_STATE

android . permission . DISABLE_KEYGUARD

android . permission . EXPAND_STATUS_BAR

android . permission . FLASHLIGHT

android . permission . GET_ACCOUNTS

android . permission . GET_PACKAGE_SIZE

android . permission . INTERNET

android . permission . KILL_BACKGROUND_PROCESSES

android . permission . MODIFY_AUDIO_SETTINGS

android . permission . NFC

android . permission . READ_SYNC_SETTINGS

android . permission . READ_SYNC_STATS

android . permission . RECEIVE_BOOT_COMPLETED

android . permission . REORDER_TASKS

android . permission . REQUEST_INSTALL_PACKAGES

android . permission . SET_TIME_ZONE

android . permission . SET_WALLPAPER

android . permission . SET_WALLPAPER_HINTS

android . permission . SUBSCRIBED_FEEDS_READ

android . permission . TRANSMIT_IR

android . permission . USE_FINGERPRINT

android . permission . VIBRATE

android . permission . WAKE_LOCK

android . permission . WRITE_SYNC_SETTINGS

com . android . alarm . permission . SET_ALARM

com . android . launcher . permission . INSTALL_SHORTCUT

com . android . launcher . permission . UNINSTALL_SHORTCUT

### Permission Group

android.permission-group.CALENDAR

- android.permission.READ_CALENDAR

- android.permission.WRITE_CALENDAR

android.permission-group.CAMERA

- android.permission.CAMERA

android.permission-group.CONTACTS

- android.permission.READ_CONTACTS

- android.permission.WRITE_CONTACTS

- android.permission.GET_ACCOUNTS

android.permission-group.LOCATION

- android.permission.ACCESS_FINE_LOCATION

- android.permission.ACCESS_COARSE_LOCATION

android.permission-group.MICROPHONE

- android.permission.RECORD_AUDIO

android.permission-group.PHONE

- android.permission.READ_PHONE_STATE

- android.permission.CALL_PHONE

- android.permission.READ_CALL_LOG

- android.permission.WRITE_CALL_LOG

- com.android.voicemail.permission.ADD_VOICEMAIL

- android.permission.USE_SIP

- android.permission.PROCESS_OUTGOING_CALLS

android.permission-group.SENSORS

- android.permission.BODY_SENSORS

android.permission-group.SMS

- android.permission.SEND_SMS

- android.permission.RECEIVE_SMS

- android.permission.READ_SMS

- android.permission.RECEIVE_WAP_PUSH

- android.permission.RECEIVE_MMS

- android.permission.READ_CELL_BROADCASTS

android.permission-group.STORAGE

- android.permission.READ_EXTERNAL_STORAGE

- android.permission.WRITE_EXTERNAL_STORAGE

### 向前和向后兼容
在运行M系统的设备中，如果app没有适配M，那app将继续使用旧的权限模式，在安装时请求用户授予全部所需权限。But！用户仍旧可以取消任何app的权限，然后系统会默默地禁用相关功能，当app执行需要该权限的操作时，该操作并不会跑出异常，而是返回空数据、error等等意想不到的结果，然后你的app就悲剧了。

在M之前系统的设备中，app仍按照旧的权限系统幸福的生活着。

### 如何使用新的权限系统
1、设置`targetSdkVersion`和`compileSdkVersion`为23。

2、执行受限操作前总是调用`checkSelfPermission(所需权限)`检查权限。

3、用户也许会拒绝授予权限，你可能需要向用户解释为什么需要某个权限。系统提供了`shouldShowRequestPermissionRationale()`方法帮助找出在什么情况下需要向用户解释。当app之前请求过权限但被用户拒绝时，该方法返回true；如果用户拒绝后还选了*不再提醒*，或设备政策禁止该app拥有某权限，该方法返回false。

4、如果app没有所需的权限，则应调用`requestPermissions()`申请权限。
	
	if (checkSelfPermission(Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {

    	// Should we show an explanation?
    	if (shouldShowRequestPermissionRationale(
    	        Manifest.permission.READ_CONTACTS)) {
    	    // Explain to the user why we need to read the contacts
    	}

    	requestPermissions(new String[]{Manifest.permission.READ_CONTACTS},
    	        MY_PERMISSIONS_REQUEST_READ_CONTACTS);

    	// MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
    	// app-defined int constant

    	return;
    }
    
    @Override
	public void onRequestPermissionsResult(int requestCode,
    	    String permissions[], int[] grantResults) {
    	switch (requestCode) {
        	case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            	if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                	// permission was granted, yay! do the
                	// calendar task you need to do.

            	} else {

                	// permission denied, boo! Disable the
                	// functionality that depends on this permission.
            	}
            	return;
        	}

        	// other 'switch' lines to check for other
        	// permissions this app might request
    	}
    }  
### 在V4支持库中处理权限
`ContextCompat.checkSelfPermission()`:不管设备是否允许M系统，当app拥有权限时返回`PERMISSION_GRANTED`，否则返回`PERMISSION_DENIED`。

`ActivityCompat.requestPermissions()`:如果设备运行的不是M系统，将会调用`ActivityCompat.OnRequestPersimmsionResultCallback`，当app已经拥有权限时传递`PERMISSION_GRANTED`，否则传递`PERMISSION_DENIED`。

`ActivityCompat.shouldShowRequestPermissionRationale()`:如果设备运行的不是M系统，总是返回false。

### 在V13支持库中处理权限
`FragmentCompat.requestPermissions()`:如果设备运行的不是M系统，将会调用`FragmentCompat.OnRequestPersimmsionResultCallback`，当app已经拥有权限时传递`PERMISSION_GRANTED`，否则传递`PERMISSION_DENIED`。

`FragmentCompat.shouldShowRequestPermissionRationale()`:如果设备运行的不是M系统，总是返回false。

扩展阅读:

http://segmentfault.com/a/1190000003694249

Android M新的运行时权限开发者需要知道的一切

## APP自动备份
---
在M系统中，系统会自动备份app数据到Google Drive，当用户更换设备后app的数据会自动地恢复。

**然而这需要依赖谷歌服务和Gmail账号，谷歌服务被阉割，所以国行安卓享受不到这个福利了，所以暂不深入研究。**

## APP链接
M为处理网站链接提供了一种新的选择，它允许点击链接后立即跳转到网站的官方app，这样节省了用户的时间，给用户带来更好的体验，当然用户也可以选择app是否应该自动打开特殊类型的链接，还是每次都提示。

**实现这一特技需要以下四步：**

1、在你的app中为网站的URL创建intent filters。

2、配置你的app，让其请求app链接的验证。

3、在你的网站发布一个数字资产链接(Digital Asset Links)JSON文件。

4、测试安卓系统是否能验证你的app。

### 创建Intent Filter
	
	<activity ...>
    	<intent-filter>
          	<action android:name="android.intent.action.VIEW" />
          	<category android:name="android.intent.category.DEFAULT" />
          	<category android:name="android.intent.category.BROWSABLE" />
          	<data android:scheme="http" />
          	<data android:scheme="https" />
          	<data android:host="www.android.com" />
    	</intent-filter>
	</activity>
以上就是一个最简单Intent filter，注意android:scheme的值必须包含http或https或二者都包含，不允许再有其他声明。然而，要让系统把你的app作为一系列url的默认处理者，你要需要请求系统验证这个链接。
### 请求App链接验证
当你在app的manifest中声明了自动验证，安卓系统会在你安装应用之后尝试验证你的app，如果验证成功，系统就会自动将那些url指向你的app，不需要用户指定。系统通过比较Intent filter中的hostnames和各自网站中的数字资产链接文件(assetlinks.json)来执行app链接验证。
	
	<activity ...>
    	<intent-filter android:autoVerify="true">
        	<action android:name="android.intent.action.VIEW" />
        	<category android:name="android.intent.category.DEFAULT" />
        	<category android:name="android.intent.category.BROWSABLE" />
        	<data android:scheme="http" android:host="www.android.com" />
        	<data android:scheme="https" android:host="example.android.com" />
    	</intent-filter>
	</activity>
`www.android.com`和`example.android.com`都需要发布assetlinks.json。
### 声明网站协作
Digital Asset Links JSON 文件放在这儿，确保它能在HTTPS连接上被访问。
	
	https://domain[:optional_port]/.well-known/assetlinks.json
该文件通过如下两个域来识别协作的app：

- 包名
- 签名证书(sha256_cert_fingerprints)

签名证书可以用keytool生成
	
	keytool -list -v -keystore my-release-key.keystore
签名证书可以包含多个，用来支持app的不同版本，如debug版和正式版。例子：
	
	[{
      "relation": ["delegate_permission/common.handle_all_urls"],
      "target": {
      "namespace": "android_app",
      "package_name": "com.example",
      "sha256_cert_fingerprints":
      ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
      }
	}]
