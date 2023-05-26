@[toc]
# Flink
## 一、基本处理函数 ProcessFunction 解析
在源码中我们可以看到，抽象类 ProcessFunction 继承了 AbstractRichFunction，有两个泛
型类型参数：I 表示 Input，也就是输入的数据类型；O 表示 Output，也就是处理完成之后输出
的数据类型。
内部单独定义了两个方法：一个是必须要实现的抽象方法.processElement()；另一个是非
抽象方法.onTimer()。
public abstract class ProcessFunction<I, O> extends AbstractRichFunction {
 ...
public abstract void processElement(I value, Context ctx, Collector<O> out) 
185
throws Exception;
public void onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 
throws Exception {}
...
}
### 1. 抽象方法.processElement()
用于“处理元素”，定义了处理的核心逻辑。这个方法对于流中的每个元素都会调用一次，
参数包括三个：输入数据值 value，上下文 ctx，以及“收集器”（Collector）out。方法没有返
回值，处理之后的输出数据是通过收集器 out 来定义的。
⚫ value：当前流中的输入元素，也就是正在处理的数据，类型与流中数据类
型一致。
⚫ ctx：类型是 ProcessFunction 中定义的内部抽象类 Context，表示当前运行的
上下文，可以获取到当前的时间戳，并提供了用于查询时间和注册定时器的“定时服
务”(TimerService)，以及可以将数据发送到“侧输出流”（side output）的方法.output()。
Context 抽象类定义如下：
public abstract class Context {
 public abstract Long timestamp();
 public abstract TimerService timerService();
 public abstract <X> void output(OutputTag<X> outputTag, X value);
} ⚫ out：“收集器”（类型为 Collector），用于返回输出数据。使用方式与 flatMap
算子中的收集器完全一样，直接调用 out.collect()方法就可以向下游发出一个数据。
这个方法可以多次调用，也可以不调用。
通过几个参数的分析不难发现，ProcessFunction 可以轻松实现 flatMap 这样的基本转换功
能（当然 map、filter 更不在话下）；而通过富函数提供的获取上下文方法.getRuntimeContext()，
也可以自定义状态（state）进行处理，这也就能实现聚合操作的功能了。关于自定义状态的具
体实现，我们会在后续“状态管理”一章中详细介绍。
### 2. 非抽象方法.onTimer()
用于定义定时触发的操作，这是一个非常强大、也非常有趣的功能。这个方法只有在注册
好的定时器触发的时候才会调用，而定时器是通过“定时服务”TimerService 来注册的。打个
比方，注册定时器（timer）就是设了一个闹钟，到了设定时间就会响；而.onTimer()中定义的，
就是闹钟响的时候要做的事。所以它本质上是一个基于时间的“回调”（callback）方法，通过
时间的进展来触发；在事件时间语义下就是由水位线（watermark）来触发了。
与.processElement()类似，定时方法.onTimer()也有三个参数：时间戳（timestamp），上下
文（ctx），以及收集器（out）。这里的 timestamp 是指设定好的触发时间，事件时间语义下当
186
然就是水位线了。另外这里同样有上下文和收集器，所以也可以调用定时服务（TimerService），
以及任意输出处理之后的数据。
既然有.onTimer()方法做定时触发，我们用 ProcessFunction 也可以自定义数据按照时间分
组、定时触发计算输出结果；这其实就实现了窗口（window）的功能。所以说 ProcessFunction
是真正意义上的终极奥义，用它可以实现一切功能。
我们也可以看到，处理函数都是基于事件触发的。水位线就如同插入流中的一条数据一样；
只不过处理真正的数据事件调用的是.processElement()方法，而处理水位线事件调用的
是.onTimer()。
这里需要注意的是，上面的.onTimer()方法只是定时器触发时的操作，而定时器（timer）
真正的设置需要用到上下文 ctx 中的定时服务。在 Flink 中，只有“按键分区流”KeyedStream
才支持设置定时器的操作，所以之前的代码中我们并没有使用定时器。所以基于不同类型的流，
可以使用不同的处理函数，它们之间还是有一些微小的区别的。接下来我们就介绍一下处理函
数的分类



说明 				Stream名称				
				DataStream
.keyBy->				KeyedStream,
.window->			WindowedStream
.allWindow->			AllWindowedStream
合并connect->			ConnectedStream
广播流再调connect变成广播连接流  	BroadcastStream
间隔连接interval join->IntervalJoined    JoinedStream
广播连接流			BroadcastConnectedStream
广播连接流总共只有两个process方法调用，BroadcastProcessFunction和keyedBroadcastProcessFunction

## 二、Flink 提供了 8 个不同的处理函数：
### （1）ProcessFunction
最基本的处理函数，基于 DataStream 直接调用.process()时作为参数传入。
### （2）KeyedProcessFunction
对流按键分区后的处理函数，基于 KeyedStream 调用.process()时作为参数传入。要想使用
定时器，比如基于 KeyedStream。
### （3）ProcessWindowFunction
开窗之后的处理函数，也是全窗口函数的代表。基于 WindowedStream 调用.process()时作
为参数传入。
### （4）ProcessAllWindowFunction
同样是开窗之后的处理函数，基于 AllWindowedStream 调用.process()时作为参数传入。
### （5）CoProcessFunction
187
合并（connect）两条流之后的处理函数，基于 ConnectedStreams 调用.process()时作为参
数传入。关于流的连接合并操作，我们会在后续章节详细介绍。
### （6）ProcessJoinFunction
间隔连接（interval join）两条流之后的处理函数，基于 IntervalJoined 调用.process()时作为
参数传入。
### （7）BroadcastProcessFunction
广播连接流处理函数，基于 BroadcastConnectedStream 调用.process()时作为参数传入。这
里的“广播连接流”BroadcastConnectedStream，是一个未 keyBy 的普通 DataStream 与一个广
播流（BroadcastStream）做连接（conncet）之后的产物。关于广播流的相关操作，我们会在后
续章节详细介绍。
### （8）KeyedBroadcastProcessFunction
按键分区的广播连接流处理函数，同样是基于 BroadcastConnectedStream 调用.process()时
作为参数传入。与 BroadcastProcessFunction 不同的是，这时的广播连接流，是一个 KeyedStream
与广播流（BroadcastStream）做连接之后的产物。


## 三、处理函数之间没有直接的继承关系
使用处理函数process时的时候，通常用KeyedStream.process调用，传入KeyedProcessFunction参数
ProcessWindowFunction即全窗口函数，也是处理函数的一种，是开窗后的处理函数
所以常用KeyedProcessFunction和ProcessWindowFunction