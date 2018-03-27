# Laravel5.5 + laravel-echo + EasyWechat 实现微信扫码登录

微信扫码登录作为一种第三方登录方式，

对于接入微信开放平台的公众号应用来说，实现扫码登录是相当容易的。不过本文所要讨论的是另一种情况，有时候我们不需要、不想或者不能接入开放平台，但又想进行微信扫码登录。

这是可以实现的。

## 基本思路：

1. 登录页显示微信二维码（使用 EasyWechat SDK 创建，短时效的临时二维码就够了）。
2. 用户扫码后推送消息到服务器接口，接口中根据业务情况进行判断处理，符合条件时触发 WechatScanLogin 事件。
3. WechatScanLogin 事件实现了 ShouldBroadcast 接口，当他

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
class WechatLogin implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $token;

    public $channel;

    /**
     * Create a new event instance.
     *
     * @param $token
     * @param $channel
     */
    public function __construct($token, $channel)
    {
        $this->token = $token;
        $this->channel = $channel;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new Channel($this->channel);
    }
}
```

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
                event(new WechatLogin('token', 'mychannel'));

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

可见，即使公众号没有接入微信开放平台，也是可以实现微信扫码登录的。

同样的原理也可以用于一些第三方支付场景，例如微信扫码支付，当应用收到支付架设确认支付完成后，让前端知道已经完成，而前端订阅相关频道事件，接收到事件后进行一些处理（通常是一些提示和跳转）。

