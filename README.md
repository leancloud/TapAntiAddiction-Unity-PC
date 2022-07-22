# 实名认证和防沉迷开发指南 Unity PC

## SDK 下载

[下载地址](https://git.gametaptap.com/tds/client-sdk/tds-sdk-all/tapsdk2/tapantiaddiction-unity-pc/uploads/e4fa31bb2eea496ee34086c8a8586b78/pc-anti-addiction.unitypackage)

## 添加依赖

### UPM 依赖

```json
"com.leancloud.storage": "https://github.com/leancloud/csharp-sdk-upm.git#storage-0.10.14",
"com.leancloud.realtime": "https://github.com/leancloud/csharp-sdk-upm.git#realtime-0.10.14",
"com.taptap.tds.bootstrap": "https://github.com/TapTap/TapBootstrap-Unity.git#3.11.0",
"com.taptap.tds.common": "https://github.com/TapTap/TapCommon-Unity.git#3.11.0",
"com.taptap.tds.login": "https://github.com/taptap/TapLogin-Unity.git#3.11.0",
```

### PC 浏览器插件

SDK 中使用到了付费 PC WebView 插件：[3D WebView for Windows and macOS](https://assetstore.unity.com/packages/tools/gui/3d-webview-for-windows-and-macos-web-browser-154144)
内部游戏可以联系 TDS 获得；外部游戏则需要自行购买。将插件导入到 Unity 工程即可。

## 初始化

```cs
using TapTap.AntiAddiction;

string clientId = "游戏的 Client ID";
AntiAddictionDelegate antiAddictionDelegate = new AntiAddictionDelegate {
    OnPlayLimit = (playable) => {
        Debug.Log($"游戏可玩: {playable.CanPlay}");
    },
    OnSwitchAccount = () => {
        Debug.Log("切换账号");
    },
    OnAuthFailed = () => {
        Debug.Log("认证失败");
        AntiAddictionUIKit.Logout();
    }
};

AntiAddictionUIKit.Init(clientId, antiAddictionDelegate);
```

### 参数说明

- clientId 是游戏的 Client ID，可以在控制台查看（开发者中心 > 你的游戏 > 游戏服务 > 应用配置）。
- antiAddictionDelegate 是防沉迷模块的事件委托，事件回调包括：
    - OnPlayLimit 可玩时间受限
    - OnSwitchAccount 切换账号
    - OnAuthFailed 鉴权失败

## 防沉迷认证

SDK 支持在界面中手动输入身份证号等实名信息进行实名制

```cs
int code = await AntiAddictionUIKit.StartUp(userId);
```

### 参数说明

- userId 玩家唯一标识，如果接入 TDS 内建账户系统，可以用玩家的 objectId；如果使用单纯 TapTap 用户认证则可以用 openid 或 unionid。

## 登出

玩家在游戏内退出账号时调用，重置防沉迷状态。

```cs
AntiAddictionUIKit.Logout();
```

## 获取玩家年龄段

调用该接口可获取玩家所处年龄段

```cs
int ageRange = AntiAddictionUIKit.AgeRange;
```

上例中的 ageRange 是一个整数，表示玩家所处年龄段的下限（最低年龄）。 特别地，-1 表示「未实名」。

- -1：未实名
- 0：0 到 7 岁
- 8：8 到 15 岁
- 16：16 到 17 岁
- 18：成年玩家

## 检查消费上限

根据年龄段的不同，未成年玩家的消费金额有不同的上限。 如果启用消费限制功能，开发者需要在未成年玩家消费前检查是否受限，并在成功消费后上报消费金额。

游戏在收到玩家的付费请求后，调用以下接口当前玩家的付费行为是否被限制：

```cs
try {
    int amount = int.Parse(amountStr);
    await AntiAddictionUIKit.CheckPayLimit(amount);
} catch (Exception e) {
    // 不允许充值
    Debug.Log("检查充值异常");
    Debug.Log(e);
}
```

消费金额的单位为分。

检查消费上限需要游戏事先上报未成年玩家的消费金额。 建议开发者在服务端上报，服务端上报方式参见 相关 REST API 用法说明。 开发者也可以调用 SDK 提供的接口，当未成年玩家消费成功后，在客户端上报消费金额，在客户端上报的可靠性低于在服务端上报，主要适用于无服务端的单机游戏。

```cs
try {
    int amount = int.Parse(amountStr);
    await AntiAddictionUIKit.SubmitPayResult(amount);
} catch (Exception e) {
    Debug.Log("上传充值异常");
    Debug.Log(e);
}
```

上报消费金额时，传入的消费金额的单位同样为分。

## 上报游戏时长

PC SDK 不需要额外调用。