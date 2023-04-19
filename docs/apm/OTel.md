# OpenTelementry

[官网手册](https://opentelemetry.io/docs/)

https://www.dynatrace.cn/resources/blog/what-is-opentelemetry-2/

APM 系统（Application Performance Management，即应用性能管理）

- 链路Tracing

- 指标Metrics：可聚合（aggregatable）

- 日志Log

> 基于Metrics的告警发现异常，通过Tracing定位问题（可疑）模块，根据模块具体的Logging定位到错误根源，最后再基于这次问题调查经验调整Metrics（增加或者调整报警阈值等）以便下次可疑更早发现/预防此类问题。

- SLI (Service Level Indicator)，即服务水平指标，代表了对服务行为的一种测量。一个好的SLI是从用户的角度来衡量你的服务。一个SLI的例子可以是一个网页的加载速度。

- SLO (Service Level Objective)，即服务水平目标，是将可靠性传达给组织/其他团队的手段。这是通过将一个或多个SLI附加到业务价值上实现的。



[什么是链路追踪](https://zhuanlan.zhihu.com/p/344020712)

## [Trace](https://goframe.org/pages/viewpage.action?pageId=73224357)

`Tracer`表示一次完整的追踪链路，`tracer`由一个或多个`span`组成。下图示例表示了一个由`8`个`span`组成的`tracer`:

```
单个Trace中，span间的因果关系


        [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C 是 Span A 的孩子节点, ChildOf)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G 在 Span F 后被调用, FollowsFrom)
```

或者使用时序图

```
单个Trace中，span间的时间关系


––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

每个**Span**包含以下的状态:（译者注：由于这些状态会反映在OpenTracing API中，所以会保留部分英文说明）

- An operation name，操作名称
- A start timestamp，起始时间
- A finish timestamp，结束时间
- **Span Tag**，一组键值对构成的Span标签集合。键值对中，键必须为string，值可以是字符串，布尔，或者数字类型。
- **Span Log**，一组span的日志集合。 每次log操作包含一个键值对，以及一个时间戳。 键值对中，键必须为string，值可以是任意类型。 但是需要注意，不是所有的支持OpenTracing的Tracer,都需要支持所有的值类型。
- **SpanContext**，Span上下文对象 (下面会详细说明)
- **References**(Span间关系)，相关的零个或者多个Span（**Span**间通过**SpanContext**建立这种关系）

每一个**SpanContext**包含以下状态：

- 任何一个OpenTracing的实现，都需要将当前调用链的状态（例如：trace和span的id），依赖一个独特的Span去跨进程边界传输
- **Baggage Items**，Trace的随行数据，是一个键值对集合，它存在于trace中，也需要跨进程边界传输

### Span

`Span`是一条追踪链路中的基本组成要素，一个`span`表示一个独立的工作单元，比如可以表示一次函数调用，一次`http`请求等等。`span`会记录如下基本要素:

- 服务名称（`operation name`）
- 服务的开始时间和结束时间
- `K/V`形式的`Tags`
- `K/V`形式的`Logs`
- `SpanContext`

`Span`是这么多对象中使用频率最高的，因此创建`Span`也非常简便，例如：

```go
gtrace.NewSpan(ctx, spanName, opts...)
```

### Attributes

`Attributes`以`K/V`键值对的形式保存**用户自定义标签**，主要用于链路追踪结果的查询过滤。例如： `http.method="GET"`，`http.status_code=200`。其中`key`值必须为字符串，`value`必须是字符串，布尔型或者数值型。 `span`中的`Attributes`仅自己可见，不会随着 `SpanContext`传递给后续`span`。 设置`Attributes`方式例如：

```go
span.SetAttributes(
	label.String("http.remote", conn.RemoteAddr().String()),
	label.String("http.local", conn.LocalAddr().String()),
)
```

### Events

`Events`与`Attributes`类似，也是`K/V`键值对形式。与`Attributes`不同的是，`Events`还会记录写入`Events`的时间，因此`Events`主要用于记录某些事件发生的时间。`Events`的`key`值同样必须为字符串，但对`value`类型则没有限制。例如：

```go
span.AddEvent("http.request", trace.WithAttributes(
	label.Any("http.request.header", headers),
	label.Any("http.request.baggage", gtrace.GetBaggageMap(ctx)),
	label.String("http.request.body", bodyContent),
))
```

### Links

Span Links是一种能够将调用链关联起来的技术，通过配置关联的Span，可以在页面中展现关联的调用链信息。**不过请注意Span Links必须要在Span创建时才能添加**，不像Events和Attributes一样能在Span创建之后添加。

## SpanContext

`SpanContext`携带着一些用于**跨服务通信的（跨进程）**数据，主要包含：

- 足够在系统中标识该`span`的信息，比如：`span_id, trace_id`。
- `Baggage` - 为整条追踪连保存跨服务（跨进程）的`K/V`格式的用户自定义数据。`Baggage` 与 `Attributes` 类似，也是 `K/V` 键值对。与 `Attributes` 不同的是：

- - 其`key`跟`value`都只能是字符串格式
	- `Baggage`不仅当前`span`可见，其会随着`SpanContext`传递给后续所有的子`span`。要小心谨慎的使用`Baggage` - 因为在所有的`span`中传递这些`K,V`会带来不小的网络和`CPU`开销。



# Span与Trace创建

### 创建嵌套的span

下面将`childSpan`嵌套在了`parentSpan`中，表示串行执行：

```golang
func parentFunction(ctx context.Context) {
	ctx, parentSpan := tracer.Start(ctx, "parent")
	defer parentSpan.End()

	// call the child function and start a nested span in there
	childFunction(ctx)

	// do more work - when this function ends, parentSpan will complete.
}

func childFunction(ctx context.Context) {
	// Create a span to track `childFunction()` - this is a nested span whose parent is `parentSpan`
	ctx, childSpan := tracer.Start(ctx, "child")
	defer childSpan.End()

	// do work here, when this function returns, childSpan will complete.
}
```