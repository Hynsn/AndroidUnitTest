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

## Mockk学习笔记

### 背景
在Java中单元测试主要是Mockito，但在Kotlin + Mockito会出现以下问题
#### Mockito cannot mock/spy because : — final class
原因是kt中任何class都是final的，Mockito不能mock final class
#### java.lang.IllegalStateException: anyObject() must not be null
原因是Mockito 调用any(), eq(), argumentCaptor(), capture() 方法时，Mockito会回传null，而kt默认不接收Null
#### when 要加上反引號才能使用
when是kt的保留字，要在Mockito中使用when，必须要加反引号
> `when`(xxxx).thenReturn(xxx)
#### 不支持测试静态方法（Static Method）
Mockito不支持该功能，mockito-kotlin也不支持。PowerMock可以但是使用上不直观，初始化繁琐，更新速度不如mockito

### Mockk简介
Mockk是一个专门为kt设计的Mocking Library，MockK 可以讓你無痛地在 Kotlin 下使用Mockito和PowerMock 的功能。

#### mock一个类 mockk+every

```kt
@Test
fun wantMoney() {
    // Given
    val mother = mockk<Mother>()
    val kid = Kid(mother)
    every { mother.giveMoney() } returns 30 // when().thenReturn() in Mockito
    // When
    kid.wantMoney()
    // Then
    assertEquals(30, kid.money)
}
```
#### 注解使用
```kt
class KidAnnotationTest {
    @MockK
    lateinit var mother: Mother
    
    lateinit var kid: Kid
    
    @Before
    fun setUp() {
        MockKAnnotations.init(this)
        kid = Kid(mother)
    }
    
    @Test
    fun wantMoney() {
        every { mother.giveMoney() } returns 30
        kid.wantMoney()
        assertEquals(30, kid.money)
    }
}
```
#### Verify 验证一段方法有没有被执行调用
```kt
@Test
fun wantMoney() {
    // When
    val mother = mockk<Mother>()
    val kid = Kid(mother)
    every { mother.giveMoney() } returns 30
    every { mother.inform(any()) } just Runs
    // Given
    kid.wantMoney()
    // Then
    verify { mother.inform(any()) }
    assertEquals(30, kid.money)
}
```

#### Relaxed 该类所有的方法都不需要指定

#### 不需指定無回傳值的方法，有回傳值的方法仍須指定

#### Verify进阶
调用次数控制
```kt
verify(exactly = 10) { mother.inform(any()) }
```
调用顺序控制：verifySequence verifyOrder
```kt
verifySequence {
    mother.inform(any())
    mother.giveMoney()
}
verifyOrder {
    mother.inform(any())
    mother.giveMoney()
}
```

#### Capture 抓取方法的参数
slot 用来抓取一个值，mutableListOf 用来抓取数组(List)
```kt
// Given
val slot = slot<Int>()
every { mother.inform(capture(slot)) } just Runs
// When
kid.wantMoney()
// Then
assertEquals(0, slot.captured)
```
测试静态Static Method, Singleton
#### mockkStatic
```kt
@Test
fun ok() {
    // Given
    val util = Util()
    mockkStatic(UtilJava::class)
    mockkStatic(UtilKotlin::class)
    every { UtilJava.ok() } returns "Joe"
    every { UtilKotlin.ok() } returns "Tsai"
    // When
    util.ok()
    // Then
    verify { UtilJava.ok() }
    verify { UtilKotlin.ok() }
    assertEquals("Joe", UtilJava.ok())
    assertEquals("Tsai", UtilKotlin.ok())
}
```
### mockkObject
```kt
@Test
fun ok() {
    // Given
    val util = Util()
    mockkObject(UtilKotlin)
    mockkObject(UtilKotlin.Companion)
    every { UtilKotlin.ok() } returns "Tsai"
    // When
    util.ok()
    // Then
    verify { UtilKotlin.ok() }
    assertEquals("Tsai", UtilKotlin.ok())
}
```
#### Singleton
```kt
@Test
fun ok() {
    // Given
    val util = Util()
    mockkObject(UtilKotlin)
    every { UtilKotlin.ok() } returns "Tsai"
    // When
    util.ok()
    // Then
    verify { UtilKotlin.ok() }
    assertEquals("Tsai", UtilKotlin.ok())
}
```
还有更多功能
- spy
- Extension functions
- Private functions
- Coroutines