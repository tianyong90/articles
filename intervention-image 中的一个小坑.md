# intervention/image 中的一个小坑及其破解之法

事实上 intervention/iamge 用了很有些时日了，它的 api 设计得很简洁，文档也很全面，用起来相当顺手。

不过最近无意间发现了一个小坑。因为需要合成带微信头像的二维码，我使用 `Image::make($avatarUrl)` （这里的 $avatarUrl 是微信头像的链接）来产生头像，然后合成到二维码图像中去（还包括一些其它操作，比如使用模板背景、写入文字）。

写完之后一运行，发现相当慢，平均耗时 23 秒左右。起初以为是因为合成过程中进行的操作比较多、尺寸比较大，本来就应该是这个速度。不过后来闲下来，开始试着优化，即使不能提升速度，至少也搞清楚到底是什么原因这么耗时。

这一通折腾下来，发现真相竟然与合成操作的多少、尺寸没有多大关系。而关键在于我创建头像数据的姿势。

为了说明这个问题，特意写了下面的代码进行对比。

```php
// 记录开始时间
$startTimestamp = microtime(true);

$url = 'http://wx.qlogo.cn/mmopen/XxT9TiaJ1ibf06TNRCMjQADS4opDHvQLguLZHpqkRlvuJYZicvJW4iaOalPsKIs0kpZ3F6864ZzibyObYiaucUQSrdp4pFTNDyIpxw/0';

$avatar = \Image::make($url);

// 记录结束时间
$endTimestamp = microtime(true);

info($startTimestamp);
info($endTimestamp);
info($endTimestamp - $startTimestamp);
```

![file](https://dn-phphub.qbox.me/uploads/images/201711/13/5342/Z2zuslLxtA.png)

上面这段代码使用 `Image::make($url)` 的形式，直接从 url 生成头像。从记录的日志数据来看，耗时基本上在 16 秒左右。

后来，想到了一个新姿势，其实也就是在尝试优化的过程中折腾时想到的。见下面代码：

```php
$startTimestamp = microtime(true);

$client = new \GuzzleHttp\Client();

$url = 'http://wx.qlogo.cn/mmopen/XxT9TiaJ1ibf06TNRCMjQADS4opDHvQLguLZHpqkRlvuJYZicvJW4iaOalPsKIs0kpZ3F6864ZzibyObYiaucUQSrdp4pFTNDyIpxw/0';

$avatarResponse = $client->get($url);

$avatar = \Image::make($avatarResponse->getBody()->getContents());

$endTimestamp = microtime(true);

info($startTimestamp);
info($endTimestamp);
info($endTimestamp - $startTimestamp);
```

在这里我先使用 GuzzleHttp 获取头像，再使用 Image::make($data) 创建头像。

注意，要高潮了…… :sunglasses:

看看下面的日志截图，三次平均耗时在 0.07 秒左右，和前面的 16 秒相比，差了 200 多倍。
![file](https://dn-phphub.qbox.me/uploads/images/201711/13/5342/Zp6xM8DQxB.png)

至于为什么会出现这种现象，自己也没搞清楚，但这无疑是一点比较有用且小众的经验。
