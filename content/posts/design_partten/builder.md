{
  "title":"Golang设计模式--构建者模式",
  "tags":[
    "设计模式"
  ],
  "date":"2021-08-16",
  "lastmod":"2021-08-16",
  "draft":"false",
  "author":"kingram"
}

# 建造者模式

在软件开发中有时需要创建一个复杂的对象，这个对象有很多小对象按照一定的步骤组成，例如计算机是由主板、内存、cpu、硬盘、显卡等组件组成，采购员不可能自己组装计算机，而是将计算机的配置要求告诉计算机销售公司，计算机销售公司再安排技术人员组装计算机，再交给采购员

生活中这种例子也有很多，比如游戏中不同角色的性格、能力、脸型，体型、发型、服装等各有差异，汽车的方向盘，轮胎、发动机、轮胎、也有多钟多样。每封邮件的主题，内容附件等内容也各不相同。

## 建造者模式的定义
是指将一个复杂的对象的构造与表象分离，使得同样的构造过程可以创建不同的表示，他将变与不变分离，即使产品的组成部分是不变的，但是每一部分是可以灵活选择的。

### 建造者模式的优点
- 封装性好，构建和表示分离
- 扩展性好，各个不同的建造者相互独立，有利于系统的解耦
- 调用者不必知道产品的内部组成细节，建造者可以对创建过程逐步细化，也不会影响到其他模块

### 建造者模式的缺点
- 产品的组成部分必须相同，限制了其使用范围
- 如果产品的内部变化过大，则建造者也要同步进行修改，后期维护成本比较大

建造者模式和工厂模式关注点不同，建造者模式更关注零部件的组装过程，而工厂模式更关注的是创建过程。但是两者可以结合使用

> 这里可以理解为：每个零件需要工厂来生产，但是实际负责组装的工厂，这里叫做建造者。

## 建造者模式的要素
建造者模式由产品（Product）、指挥者（Director）、抽象创建者（Builder）、具体创建者（Concrete Builder）4个要素组成


## 具体代码分析
- 产品  
包含getter setter方法
```go
// Product 是产品 
type Product struct {
	partA string
	partB string
	partC string
}

func (p *Product) PartA() string {
	return p.partA
}

func (p *Product) SetPartA(partA string) {
	p.partA = partA
}

func (p *Product) PartB() string {
	return p.partB
}

func (p *Product) SetPartB(partB string) {
	p.partB = partB
}

func (p *Product) PartC() string {
	return p.partC
}

func (p *Product) SetPartC(partC string) {
	p.partC = partC
}
```
- 指挥者   
包含一个建造函数Construct()返回产品
```go
// Director 是指挥者
type Director struct {
	builder Builder
}

func (d *Director)Construct() interface{} {
	d.builder.newProduct()
	d.builder.buildPartA()
	d.builder.buildPartB()
	d.builder.buildPartC()
	return d.builder.getResult()
}

func NewDirector(builder Builder) *Director {
	return &Director{
		builder: builder,
	}
}
```
- 抽象建造者
```go
// Builder 是一个抽象建造者
type Builder interface {
	// 创建新产品
	newProduct()
	buildPartA()
	buildPartB()
	buildPartC()
	// 返回产品对象
	getResult() interface{}
}
```
- 具体建造者   
实现Builder接口
```go
// ConcreteBuilder 是具体的建造者
type ConcreteBuilder struct {
	product *Product
}

func (cb *ConcreteBuilder)newProduct() {
	cb.product = &Product{}
}

func (cb *ConcreteBuilder)buildPartA() {
	cb.product.SetPartA("A部分已构建")
}
func (cb *ConcreteBuilder) buildPartB() {
	cb.product.SetPartB("B部分已构建")
}

func (cb *ConcreteBuilder)buildPartC() {
	cb.product.SetPartC("C部分已构建")
}
// 返回产品对象
func (cb *ConcreteBuilder)getResult() interface{} {
	return cb.product
}
```
- 实际调用
```go
b:= &builder.ConcreteBuilder{}
director := builder.NewDirector(b)
product := director.Construct()
```

## 适用场景
- 相同的方法，不同的执行顺序，产生不同的结果。
- 多个部件或零件，都可以装配到一个对象中，但是产生的结果又不相同。
- 产品类非常复杂，或者产品类中不同的调用顺序产生不同的作用。
- 初始化一个对象特别复杂，参数多，而且很多参数都具有默认值。

[完整代码传送门](https://github.com/K1ngram4/my-design-parttern)