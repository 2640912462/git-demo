# 快速敏感词1过滤

## 本项目不再维护

详见issues，存在一些问题，暂时没有精力修复，不推荐使用。感谢理解
~~即将停止和移除（本项目计划于1月31日停止和移除）~~


### 性能概述

使用60MB大小的小说测试，单核性能超过50M字符每秒（i7 2.3GHz）。

```
敏感词 14553 条
待过滤文本共 599254 行，30613005 字符。
过滤耗时 0.535 秒， 速度为 57220.6字符/毫秒
其中 39691 行有替换
```

### 优化方式

主要的优化目标是速度，从以下方面优化：

1. 敏感词都是2个字以上的，
2. 对于句子中的一个位置，用2个字符的hash在稀疏的hash桶中查找，如果查不到说明一定不是敏感词，则继续下一个位置。
3. 2个字符（2x16位），可以预先组合为1个int（32位）的mix，即使hash命中，如果mix不同则跳过。
4. StringPointer，在不生成新实例的情况下计算任意位置2个字符的hash和mix
5. StringPointer，尽量减少实例生成和char数组的拷贝。

### 敏感词库

默认敏感词库拷贝自 https://github.com/observerss/textfilter ，并删除如`女人`、`然后`这样的几个常用词。
使用默认敏感词库的示例如下

```java
// 使用默认单例（加载默认敏感词库）
SensitiveFilter filter = SensitiveFilter.DEFAULT;
// 向过滤器增加一个词
filter.put("婚礼上唱春天在哪里");
	
// 待过滤的句子
String sentence = "然后，市长在婚礼上唱春天在哪里。";
// 进行过滤
String filted = filter.filter(sentence, '*');
	
// 如果未过滤，则返回输入的String引用
if(sentence != filted){
	// 句子中有敏感词
	System.out.println(filted);
}
```

打印结果

```
然后，**在*********。
```

### 依赖

JDK 1.7版本及以上


