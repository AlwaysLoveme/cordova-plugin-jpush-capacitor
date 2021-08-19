# cordova-plugin-jpush-capacitor
[![platforms](https://img.shields.io/badge/platforms-iOS%7CAndroid-lightgrey.svg)](https://github.com/jpush/jpush-phonegap-plugin)
[![weibo](https://img.shields.io/badge/weibo-JPush-blue.svg)](http://weibo.com/jpush?refer_flag=1001030101_&is_all=1)

在官方基础上，增加对Capacitor电容器的支持(支持Capacitor3)
### Capacitor(2.0/3.0) 安装
```bash
npm install cordova-plugin-jpush-capacitor -S

npm install cordova-plugin-device -S

npm install cordova-plugin-jcore -S
```
同步插件至Native项目：
```bash
npx cap sync
```
#### IOS 使用
配置JPUSH APPkey:
![https://img-blog.csdnimg.cn/2020110211285992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3podXhpYW5kYW4=,size_16,color_FFFFFF,t_70#pic_center](https://img-blog.csdnimg.cn/2020110211285992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3podXhpYW5kYW4=,size_16,color_FFFFFF,t_70#pic_center)
也可以在安装npm包之后，在插件目录 **`node_modules/cordova-plugin-jpush-capacitor/src/ios/JPushConfig.plist`** 中修改，推荐使用这个方式更改，更改之后需要手动在执行一遍`npx cap sync`

IOS项目中添加桥接头文件, 如图：
![https://img-blog.csdnimg.cn/20201102105208957.gif#pic_center](https://img-blog.csdnimg.cn/20201102105208957.gif#pic_center)
生成的`.m`文件可以删除

在生成的头文件中导入jpush:
```h
#import "JPUSHService.h"
```
#### Capacitor 注册极光devicetoken
Capacitor2.0版本：
在Appdelegate.swift 文件的 didRegisterForRemoteNotificationsWithDeviceToken 方法中添加如下代码：
```swift
func application(application: UIApplication,
    didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
  NSNotificationCenter.defaultCenter().postNotificationName("DidRegisterRemoteNotification", object: deviceToken)
  JPUSHService.registerDeviceToken(deviceToken)
}

```
Capacitor3.0版本：
在Appdelegate.swift中加入以下代码：
```swift
func application(_ application: UIApplication,
didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
  JPUSHService.registerDeviceToken(deviceToken)
}
```

以上准备就绪后，需要手动调用初始化SDK：
```ts
window.JPush.startJPushSDK() // 可以打印一下window.JPush, 官方支持的API均可以使用
```

### Android 使用
配置JPUSH APPkey: 
![Android 配置APPKey](https://img-blog.csdnimg.cn/20201102102100967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3podXhpYW5kYW4=,size_16,color_FFFFFF,t_70#pic_center)
也可以在安装npm包之后，在插件目录 **`node_modules/cordova-plugin-jpush-capacitor/plugin.xml`** 中修改，推荐使用这个方式更改
```xml
// 在plugin.xml中将此处的$APP_KEY替换成自己的即可
<meta-data android:name="JPUSH_APPKEY" android:value="$APP_KEY" /> 
```
替换之后需要手动在执行一遍`npx cap sync`，直接在原生项目中做替换的可不必运行

Android无需手动初始化SDK

#### 示例代码(以Ionic为例)
```TS
import { isPlatform } from '@ionic/vue';
class Jpush {
    jpush: any;

    constructor() {
        if (window.JPush) {
            this.jpush = window.JPush;
            this.jpush.setDebugMode(true);
            if (isPlatform('ios')) {
                this.jpush.startJPushSDK();
            }
            this.jpush.init();
        }
    }

    getRegistrationID() {
        return new Promise(resolve => {
            this.jpush.getRegistrationID(function (rId: string) {
                resolve(rId);
                console.log("JPushPlugin:registrationID is " + rId);
            })
        })
    }

    // 设置别名
    setAlias(alias: string) {
      return new Promise(((resolve, reject) => {
        this.jpush.setAlias({
          alias,
          sequence: new Date().valueOf(),
        },  (res: { alias: string, sequence: number }) => {
          console.log('别名设置成功: ', res);
          resolve(res);
        },  (err: { code: number, sequence: number }) => {
            console.log('别名设置失败: ', err);
            setTimeout(() => this.setAlias(alias), 3000);
            reject(err);
        })
      }))

    }

    // 设置角标 只限IOS
    setBadge(badge: number) {
      if (isPlatform('ios')) {
        this.jpush.setBadge(badge);
      }
    }
}
export default Jpush

```

API请查阅官方文档：[https://github.com/jpush/jpush-phonegap-plugin](https://github.com/jpush/jpush-phonegap-plugin)
