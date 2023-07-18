## 单元测试
### 测试用例范围：
1，viewmodel中公用的属性、方法
2、工具类，不同的输入输出
3、对于移动端测试用例尽可能模拟用户UI交互、业务逻辑（编写单测时，关注业务逻辑输入输出）

### 分类：
本地测试(Local tests): 
只在本地机器JVM上运行，以最小化执行时间，这种单元测试不依赖于Android框架，或者即使有依赖，也很方便使用模拟框架来模拟依赖，以达到隔离Android依赖的目的，模拟框架如google推荐的[Mockito][1]；
仪器化测试(Instrumented tests): 
在真机或模拟器上运行的单元测试，由于需要跑到设备上，比较慢，这些测试可以访问仪器（Android系统）信息，比如被测应用程序的上下文，一般地，依赖不太方便通过模拟框架模拟时采用这种方式。

#### 参考：
- https://www.jianshu.com/p/ba887343f72b
- https://blog.csdn.net/yus201120/article/details/118158956
- https://blog.csdn.net/to_perfect/article/details/80738867
- https://blog.csdn.net/qq_17766199/article/details/78881992
- https://www.jianshu.com/p/3aa0e4efcfd3
- https://www.jianshu.com/p/c83a33fda25a
- https://juejin.cn/post/6844904138367582216