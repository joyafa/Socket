# Socket
TCP通信的那些事

记录国元证券问题排查的点点滴滴。
最近查关于国元证券通信异常的问题。
起初总是在怀疑是不是socket通信的问题，因为程序上午运行的好好的，下午就会出现问题，而且每次都是下午时候，重启后恢复正常。
 主要从以下几点分析：
 1 socket句柄太多，因不再使用的句柄没有关闭，超出服务器端最大可以接受的句柄数，导致新的连接无法再次创建；
    查异常时候，句柄数情况，并无异常；
 2 内存使用情况，是否异常，通过查任务管理器，在连接报错的时候，也未发现异常；

 后面始终查不到原因，最后把后台的界面截图分析后，才知道原来是包需要不一致的问题。
 起初还有怀疑过包序号是否为服务器自行生成并维护的，在某种状态下没有正常更新维护，导致下次收到的跟前一次比较不一致，从而在接口程序中报错；
 后面分析原来这个包序是接口自己生成的，在每个包请求里面发送给服务器，服务器仅仅是将包序号原路返回，用来定位请求和应答，因为如果异步通信的情况下，是只能通过一个类似包编号的字段来对应请求和应答，但是对于同步阻塞模式，包序号的判断似乎有些多余，暂且不考虑，还是比较一下稳妥。
  然后将请求应答包数据完全打印出来后发现文档所在，因服务器端解析数据是通过 |分隔符来区分的，如果请求中多了　｜　会导致解析错误，修正后ｏｋ。一切正常。
   　　通过这次的问题，排查了好久，一直未能定位，首先要方法正确，如果早从界面上看出现异常时候，界面的显示，可能会有更好的思路，会更快的定位问题；
　　其次，还是得把完整的请求应答包答应出来，会更加好分析问题。
　　
　　通过这次排查问题，了解了很多关于ｔｃｐ通信底层的东西，如设置阻塞模式，也可以设置成非阻塞模式；
　　ｓｏｃｋｅｔ通信的很多配置设置，请求应答缓存区大小设置等等；
　　创建ｓｏｃｋｅｔ的流程等都有较深刻的认识。