# CodePush 接入流程

## 接入流程

1. 安装 CodePush CLI
2. 注册 CodePush 账号
3. 在CodePush服务器注册App
4. 原生应用中配置 CodePush
5. RN 代码中集成 CodePush
6. 发布更新的版本

### 1.安装 CodePush CLI

```
//全局安装
$ npm install -g code-push-cli
```
```
//验证版本
$ code-push -v
```
### 2. 注册 CodePush 账号

```
$ code-push register
```
执行命令后会打开一个授权网页 "App Center "，选择授权登录方式。

按照提示操作，注册成功后，CodePush 会给一个 token 

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://dl.catch.cc/AuthenticationSucceeded.png">
    <br>
    <div style="color:#999999; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Authentication succeeded</div>
</center>

将其复制到命令行，回车即可，显示如下提示即登录成功。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://dl.catch.cc/CommandLine.png">
    <br>
    <div style="color:#999999; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">命令行</div>
</center>

-
**[注1]**

注册成功后，会自动登录了。除非明确执行 `logout` ，否则不需要再次登录。

**[注2] 关于 Access Key**

当执行 `code-push login` 登录后，会生成一个新的 Access Key 并保存在本地磁盘上，这是登录的凭证(也是[注1]的原因)。

当执行 `code-push logout` 将会删除这个 Access Key。

如果在某台机器上忘记 logout，可以使用`code-push access-key ls`列出和`code-push access-key rm <accessKey>`删除 Access Keys，来解除登录状态。

**[注3] CodePush 登录相关命令：**

- `code-push login` 登录
- `code-push loout` 注销
- `code-push access-key ls` 列出登陆的token
- `code-push access-key rm <accessKye>` 删除某个access-key
- `code-push whoami` 显示当前登录的 e-mail 账号

### 3. 在 CodePush 服务器注册 App
```
$ code-push app add <appName> <os> <platform>
```
`<appName>` - 应用名

`<os>` - `ios`、`windows`、`android`

`<platform>` - `react-native`、`cordova`...

因为 CodePush 在 iOS 和 Android 的更新包内容会有差异，在 iOS 和 Android 使用相同的 App 可能会导致安装异常。所以***iOS 和 Android 要分别创建 App***，两个 App 独立管理和发布更新：

```
code-push app add AppName-Android android react-native
code-push app add AppName-iOS ios react-native
```
执行 `app add` 成功后返回 `Production` 和 `Staging` 两个 Deployment Key。

-
**[注1] 部署环境**

在 CodePush 注册的应用默认包含两个部署环境：`Staging`和`Production`。

也可以自定义部署环境如下：

```
//创建
code-push deployment add <appName> <deploymentName>

//删除
code-push deployment rm <appName> <deploymentName>

//重命名
code-push deployment rename <appName> <deploymentName> <newDeploymentName>

//查看
code-push deployment ls <appName>

//显示秘钥
code-push deployment ls <appName> -k
```

**[注2] deployment ls 结果解读**

`Active` - 成功安装的数量目前运行这个版本。这个数字将会随着用户更新到或离开这个版本分别增加或减少。

`Total` - 该版本更新收到的所有成功安装的总数。这个数字只会随新用户/设备安装它而增加。

`Pending` - 更新被下载了但还没安装的数量。如果已经配置了立即更新但还是有 Pending，很可能是在 app 启动的时候没有调用 notifyApplicationReady/sync。

`Rollbacks` - 该版本被自动回滚的次数。理想情况下这个数应该为0并不会显示。如果发布了一个包含严重问题(Crash)的更新，CodePush 插件将在安装时回滚到上一个版本，同时把问题反馈到服务端。

`Rollout` - 显示可以接收更新的用户百分比，只会被显示在最新版本。

`Disabled` - 该版本是否被标记成禁用，这个属性只有在版本禁用时才显示。

`No installs recorded` - 表示这个版本在服务器上没有任何活动记录。这可能要么是因为被插件阻止了，或者用户还没有跟CodePush服务器同步。

**[注3] CodePush app 相关命令：**

- `code-push app add` 在账号里面添加一个新的 App
- `code-push app rm` 在账号里移除一个 App
- `code-push app rename` 重命名一个存在 App
- `code-push app ls` 列出账号下面的所有 App
- `code-push app transfer` 把app的所有权转移到另外一个账号

### *多人合作与权限管理

多个开发者合作同一个 CodePush 应用，可以把开发者添加为 collaborator，使用如下命令：

```
//添加
code-push collaborator add <appName> <collaboratorEmail>

//查看
code-push collaborator ls <appName>

//移除
code-push collaborator rm <appName> <collaboratorEmail>
```
### 4. 原生应用中配置 CodePush
```
//cd 到工程目录下
//安装 CodePush 组件
npm install react-native-code-push --save

//添加依赖
//如果报 'link' unrecognized 运行 npm install 即可
react-native link react-native-code-push

```
如果要自动设置 key 则根据要求，输入之前 `app add` 时获得的 key 即可。
>可以通过 `code-push deployment ls <appName> -k` 查看 key。

推荐手动设置，方式如下

##### iOS deployment-key 的设置
- 创建 Configurations。
	- Xcode 打开项目 - [PROJECT] - [Info]
	- [Configurations] - 单击"+" - [Duplicate "Release" Configuration]
	- 输入Staging(名称可以自定义)
	 
<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" src="http://dl.catch.cc/ios-1.jpg">
    <br>
    <div style="color: #999999; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        Configuration
    </div>
</center>

- 自定义 User-Defined Setting
	- [Build Setting]，单机"+"选择 [User-Defined Setting] 
	- 输入 CODEPUSH_KEY （可自定义）
	
<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" src="http://dl.catch.cc/ios-2.jpg">
    <br>
    <div style="color: #999999; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        User-Defined Setting
    </div>
</center>

- 添加 Info.plist 设置
	- Key CodePushDeploymentKey
	- Type String
	- Value $(CODEPUSH_KEY)

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" src="http://dl.catch.cc/ios-3.png">
    <br>
    <div style="color: #999999; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        Info.plist
    </div>
</center>

- 版本设置
	- 检查一下版本号是不是三位，如果不是的话，修改为三位（比如：1.0.0）。CodePush 只支持三位的，不然在推包的时候无法确定推包给哪个版本。

##### Android deployment-key 的设置
未尝试

-
**[注] `react-native link react-native-code-push` 做了什么**

1. Info.plist。如果使用了自动设置 Key，会在 Info.plist 下创建一个新的键值对。
2. AppDelegate。如下图，在非 DEBUG 情况下通过 CodePush 获取 bundleURL

<center>
    <img style="border-radius: 0.3125em; box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" src="http://dl.catch.cc/Appdelegate.png">
    <br>
    <div style="color: #999999; border-bottom: 1px solid #d9d9d9; display: inline-block; color: #999; padding: 2px;">
        AppDelegate
    </div>
</center>

### 5. RN 代码中集成 CodePush

首先在 js中加载 CodePush模块：

```js
import codePush from 'react-native-code-push'
```
然后在根 component（通常为 App） 的 `componentDidMount` 中调用 `sync` 方法，再把 export default 包装一层更新策略

```js
class App extends Component<Props> {
  componentDidMount() {
    codePush.sync({
	   updateDialog: true,
      installMode: codePush.InstallMode.IMMEDIATE,
      deploymentKey: '5p0FCe4dj8OH_sr3-BPnBeAuVx3Cda733de2-a33a-4498-820b-cc58ffc73a0b'
  	});
  }
  
  render() { ... }
}

//把 export default 包装一下
const codePushOptions = { checkFrequency: codePush.CheckFrequency.MANUAL };
const AppContainer = codePush(codePushOptions)(App);
export default AppContainer;
```
>这里 checkFrequency 定义了何时进行 bundle 更新，这里的 MANUAL 是说让用户选择是否更新，还有诸如 ON_APP_RESUME 是指在后台静默地将更新下载到本地，等待 App 下一次启动的时候应用更新。 

-
**[注] sync 的一些说明**

sync 是自动更新模式，官方还提供了手动更新模式，我还没试过

sync 支持的参数

* deploymentKey （String）： 部署key，指定你要查询更新的部署秘钥，默认情况下该值来自于Info.plist(Ios)和MianActivity.java(Android)文件，你可以通过设置该属性来动态查询不同部署key下的更新。
* installMode (codePush.InstallMode)： 安装模式，用在向CodePush推送更新时没有设置强制更新(mandatory为true)的情况下，默认codePush.InstallMode.ON_NEXT_RESTART即下一次启动的时候安装。  
* mandatoryInstallMode (codePush.InstallMode):强制更新,默认codePush.InstallMode.IMMEDIATE。
* minimumBackgroundDuration (Number):该属性用于指定app处于后台多少秒才进行重启已完成更新。默认为0。该属性只在`installMode`为`InstallMode.ON_NEXT_RESUME`情况下有效。  
* updateDialog (UpdateDialogOptions) :可选的，更新的对话框，默认是null,包含以下属性
	* appendReleaseDescription (Boolean) - 是否显示更新description，默认false
	* descriptionPrefix (String) - 更新说明的前缀。 默认是” Description: “
	* mandatoryContinueButtonLabel (String) - 强制更新的按钮文字. 默认 to “Continue”.
	* mandatoryUpdateMessage (String) - 强制更新时，更新通知. Defaults to “An update is available that must be installed.”.
	* optionalIgnoreButtonLabel (String) - 非强制更新时，取消按钮文字. Defaults to “Ignore”.
	* optionalInstallButtonLabel (String) - 非强制更新时，确认文字. Defaults to “Install”.
	* optionalUpdateMessage (String) - 非强制更新时，更新通知. Defaults to “An update is available. Would you like to install it?”.
	* title (String) - 要显示的更新通知的标题. Defaults to “Update available”.


### 6. 发布更新

Android

```
code-push release-react <AppName> android
```

iOS

```
code-push release-react <AppName> ios
```

>在命令行输入 code-push release-react 回车，可以查看更多参数，诸如 -des 更新内容描述、-d 推送环境

