# 使用 Laravel 数据填充功能生成中文测试数据

今晚……不对，是昨晚，折腾一个的小项目，发现自动填充的中文数据显示起来总不太美观，于是开始琢磨如何填充中文数据进行测试。

然而一番搜索后惊奇的发现，官方、以及一些非官方的文档均未提及这一功能。期间看到一篇他人的“经验”文章，虽然可以实现这一需求，却要求修改 vendor 目录下 fzaninotto/Faker 包的源码，对于一个中了 Laravel 的“优雅之毒”的人来说，怎能容忍如此风骚的操作？

一定有更好的办法……

继续理清 Laravel 模型工厂原理之后，终于有所进展。发现其实只需要一个小小的修改就可以实现这一功能。

- 根据官方示例的模型工厂代码

```php
$factory->define(App\Product::class, function (Faker\Generator $faker) {
    return [
        'user_id' => 1,
        'name' => $faker->name,
        'mobile' => $faker->phoneNumber,
        'province' => $faker->state,
        'city' => $faker->city,
        'area' => $faker->area,
        'address' => $faker->streetAddress,
        'postcode' => $faker->postcode,
    ];
});
```

- 调整后的代码

```php
$factory->define(App\Address::class, function () {
    $faker = Faker\Factory::create('zh_CN');

    return [
        'user_id' => 1,
        'name' => $faker->name,
        'mobile' => $faker->phoneNumber,
        'province' => $faker->state,
        'city' => $faker->city,
        'area' => $faker->area,
        'address' => $faker->streetAddress,
        'postcode' => $faker->postcode,
    ];
});
```

调整前，使用依赖注入的 `Faker\Generator` 是使用的默认语言，即英文。

调整后， `Faker\Factory::create('zh_CN')` 也会返回一个 `Faker\Generator`， 但它是使用汉语初始化的。

** 事实上 Faker 本地化对于中文的支持仍有部分待完善，使用暂时不支持生成随机中文句子或者段落（相应的方法返回的仍然会是英文的），但我相信不久之后会有大牛实现这一些功能。 **

最后，上图，实际生成数据效果：

> 请别纠结省市区从属关系，数据仅供测试 :smile: