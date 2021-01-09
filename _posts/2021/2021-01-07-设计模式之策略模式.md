最近在项目开发过程中，有一个类是承接各种任务入口，里面有各种的的条件判断，代码中堆砌着大量的`ifelse`代码，当新增一个任务类型的时候，需要在后面添加新的逻辑处理，因此想要把里面每个独立的任务处理逻辑独立出来，使得代码更加简洁易懂。这个时候`策略模式`就派上用场了！

### 意图
实现一系列的算法，将它们封装起来，使它们之间可以相互替换。符合面向对象编程中的里氏替换原则(父类出现的地方，其子类都可以出现)。

### 角色
策略模式中包括三种角色，分别为`Context`、`Strategy`和`ConcreteStrategy`。

**Context(环境类)**：环境类是使用算法的角色，它在解决某个问题（即实现某个方法）时可以采用多种策略。在环境类中维持一个对抽象策略类的引用实例，用于定义所采用的策略。

**Strategy(抽象策略类)**：它为所支持的算法声明了抽象方法，是所有策略类的父类，它可以是抽象类或具体类，也可以是接口。环境类通过抽象策略类中声明的方法在运行时调用具体策略类中实现的算法。

**ConcreteStrategy（具体策略类）**：它实现了在抽象策略类中声明的算法。在运行时，具体策略类将覆盖在环境类中定义的抽象策略类对象，使用一种具体的算法实现某个业务处理。


### 示例
假设现在有一个需求是将Docx文件转为其他格式的内容，比如JSON文件、PDF文件、图片等等，传统的写法如下：

```go
package strategy

// 策略模式

// 抽象策略类
type IStrategy interface {
	do(int, int) int
}

type add struct{}

// 具体策略类，实现具体的某一种算法
func (*add) do(a, b int) int {
	return a + b
}

type sub struct {
}

func (*sub) do(a, b int) int {
	return a - b
}

// 环境类，保持一个算法的引用，具体的算法执行者
type Operator struct {
	strategy IStrategy
}

func (operator *Operator) setStrategy(strategy IStrategy) {
	operator.strategy = strategy
}

func (operator *Operator) calculate(a, b int) int {
	return operator.strategy.do(a, b)
}


```
测试类
```go
package strategy

import (
	"fmt"
	"testing"
)

func TestStrategy(t *testing.T) {
	operator := &Operator{}

	operator.setStrategy(&add{})
	result := operator.calculate(1, 2)
	fmt.Println(result)

	operator.setStrategy(&sub{})
	result = operator.calculate(10, 9)
	fmt.Println(result)
}

```

运行结果
```go
=== RUN   TestStrategy
3
1
--- PASS: TestStrategy (0.00s)
PASS

Process finished with exit code 0
```



