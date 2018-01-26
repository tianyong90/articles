Laravel5.5 是 Laravel 最新的一个 LTS 版本，发布至今已有些时日，眼看着 5.6 都快出来了，最近终于下手将公司项目从 Laravel5.2 升级到 5.5。

因为跨了好几个版本，变化不少，加上其它一些不兼容的包也得相应作调整并进行测试，前后两天折腾下来总算弄完。上线后一切正常，似乎连运行速度都提高了不少（可能只是心理作用 :smile:）。

然而没多久出现了一种奇怪的现象，明明刚刚写入了数据，但查询时却报 No query result ，而且只是偶然性出现，没啥规律。自己直接连上数据库一查，里面明明白白的记录摆在那儿，难道见鬼了不成？

后来好一阵折腾，直到再一次仔细翻看文档, 才发现 Laravel5.5 数据库读写分离配置的部分额外提到了一个 sticky 项，文档里这部分原文如下:

> The sticky Option

> The sticky option is an optional value that can be used to allow the immediate reading of records that have been written to the database during the current request cycle. If the sticky option is enabled and a "write" operation has been performed against the database during the current request cycle, any further "read" operations will use the "write" connection. This ensures that any data written during the request cycle can be immediately read back from the database during that same request. It is up to you to decide if this is the desired behavior for your application.

所以情况一下就明朗了，在没有启用 sticky 的时候，使用 write 连接写入数据后**立即**读取，读取时使用的是 read 连接，这样就有可能出问题。将 sticky 设置为 true 后，在与这个写入操作相同的请求周期内的后续读取操作，仍然使用原来的 write 连接，就不会有这麻烦了。

