# Laravel + laravel-echo + EasyWechat 实现微信扫码登录

对于接入微信开放平台的公众号应用来说，实现扫码登录是相当容易的，有 EasyWechat SDK 加持，再按着官方的文档一把梭，很快就能完成。
不过本文所要讨论的是另一种情况，有时候出于某些原因，自己的公众号不能接入开放平台，但又想进行微信扫码登录，这种情况下显示就不能再换官方的套路来了。但只要我们稍作变通，就能实现这一需求。

## 基本思路：

1. 登录页显示微信二维码（使用 EasyWechat SDK 创建，短时效的临时二维码）
2. 用户扫码后推送消息到服务器接口，接口中根据业务情况进行判断处理，符合条件时触发 WechatScanLogin 事件
3. WechatScanLogin 事件实现 ShouldBroadcast 接口，所以当它被触发时也会向指定的频道进行广播
4. 前端 laravel-echo 监听频道中用户扫码登录的消息并进行处理

## 具体实现

1. 安装依赖

```shell
npm install -g laravel-echo-server

npm install laravel-echo --save

composer require overtrue/laravel-wechat:dev-master
```

2. 创建 WechatScanLogin 事件

```shell
php artisan make:event WechatScanLogin
```

WechatScanLogin 逻辑如下

```php
class WechatScanLogin implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $token;

    /**
     * Create a new event instance.
     *
     * @param $token
     */
    public function __construct($token)
    {
        $this->token = $token;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new Channel(‘scan-login’);
    }
}
```

上面最关键的就是事件要实现 ShouldBroadcast 接口并在 broadcastOn 方法中指定要广播的频道。WechatScanLogin 的公开属性 token 会自动包含在广播数据中。

3. 

4. 对接微信消息服务器

laravel-wechat 的相关配置和对接，请阅读 EasyWechat SDK 官方文档。

5. 接收扫码的消息并进行相关处理。

```php
public function serve()
{
    $app = app('wechat.official_account');

    $app->server->push(function ($message) {
        if ($message['Event'] === 'SCAN') {
            $user = User::where('openid', $message['FromUserName'])->first();

            if ($user && User::hasRole('admin')) {
                event(new WechatScanLogin('token'));

                return '登录成功';
            } else {
                return '用户不存在或者无登录权限';
            }
        }
    }, EasyWeChat\Kernel\Messages\Message::EVENT);

    return $app->server->serve();
}
```

5. 配置并启动 laravel-echo-server

```shell
laravel-echo-server start
```

6. 前端登录页使用 laravel-echo 订阅微信登录频道的事件，接收后存储用户凭据并进行跳转。

```js
echo.channel('scanlogin').listen('WechatScanLogin', (e) => {
    this.$message({
    message: '登录成功',
    type: 'success'
    })

    this.$router.push('/')
})
```

## 总结


