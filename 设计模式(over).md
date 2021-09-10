代码由C++实现，具体方法并不严格遵照UML图
### 原则
- 开闭原则  
> 软件实体应当对扩展开放，对修改关闭

- 里氏替换原则
> 子类可以扩展父类的功能，尽量不要重写父类的方法

- 依赖倒置原则
> 面向接口编程，不要面向实现编程

- 单一职责原则
> 一个类应该有且仅有一个引起它变化的原因，否则类就应该被拆分

- 接口隔离原则
> 客户端不应该被迫依赖于它不使用的方法。一个类对另一个类的以来应该建立在最小的接口上

- 迪米特法则
> 如果两个软件实体无需直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。降低类的耦合度，提高模块的相对独立性

- 合成复用原则
> 在软件复用时，尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现

### 创建型
将对象的创建与使用分离
##### 单例模式
某个类只能生成一个实例，该类提供了一个全局访问点供外部获取该实例，其拓展时有限多例模式。
`优点`
- 保证内存里只有一个实例，减少内存开销
- 避免对资源的多重占用
- 设置全局访问点，可以优化和共享资源的访问

`缺点`
- 一般没有接口，扩展困难
- 无法生成多个对象，并发测试中不利于代码调试

![Singleton](http://c.biancheng.net/uploads/allimg/181113/3-1Q1131K441K2.gif)

```c++
//懒汉式单例
class LazySingleton {
public:
	static LazySingleton* getInstance() {
		if (instance == nullptr)
			return new LazySingleton;
		return instance;
	}
private:
	LazySingleton() {};
	~LazySingleton() {};
	LazySingleton(const LazySingleton&);
	LazySingleton& operator=(const LazySingleton&);
	static LazySingleton *instance;

//引入嵌套类解决内存泄漏问题，或者使用智能指针
private:
	class Deletor {
	public:
		~Deletor() {
			if (LazySingleton::instance != nullptr)
				delete LazySingleton::instance;
		}
	};
};
//init
LazySingleton *LazySingleton::instance = nullptr;
```
懒汉式单例只有在第一次调用getInstance方法时才会去创建。还有另一种饿汉式单例则在类加载的时候就创建一个单例,实现的时候将new放置为私有静态即可。这两种单例在多线程下有差异。
##### 原型模式
将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例。

`优点`
- 深克隆方式保存对象状态，简化对象的创建过程，可辅助实现撤销操作

`缺点`
- 需要为每个类都配置clone方法
- clone方法位于类内部，修改时违背开闭原则
- 存在多重嵌套引用时，实现复杂

![Prototype](http://c.biancheng.net/uploads/allimg/181114/3-1Q114101Fa22.gif)

```c++
//深拷贝
class Test {
public:
	Test() {};
	Test(const Test&) {};
	Test* clone() {
		Test* cn = new Test(*this);
		return cn;
	}
};
```
##### 工厂方法模式
定义一个用于创建产品的接口，由子类决定生产什么产品
`优点`
- 用户只需直到具体工厂名即可得到产品，无需知道产品的具体创建过程
- 灵活性强，创建新产品时只需多一个相应的工厂类
- 典型解耦框架，高层模块只需知道产品的抽象类

`缺点`
- 类的数量多，增加复杂度
- 增加系统的抽象性和理解难度
- 抽象产品单一，可通过抽享工厂模式解决

![ConcreteFactory](http://c.biancheng.net/uploads/allimg/181114/3-1Q114135A2M3.gif)

```c++
class abstractProduct {
public:
	virtual void ope() = 0;
};

class ProductA : public abstractProduct {
public:
	void ope() {
		cout << "ProductA" << endl;
	}
};

class ProductB : public abstractProduct {
public:
	void ope() {
		cout << "ProductB" << endl;
	}
};

class Factory {
public:
	abstractProduct* createProduct(bool num) {
		if (num)
			return new ProductA;
		else
			return new ProductB;
	}
};
```
##### 抽象工厂模式
提供一个创建产品族的接口，其每个子类可以生产产品族内一系列相关的产品
`优点`
- 在类内部实现对多等级产品的共同管理
- 保证客户端只使用同一产品的产品组

`缺点`
- 产品组中增加新产品时，所有工厂类都要修改

![AbstractFactory](http://c.biancheng.net/uploads/allimg/181114/3-1Q1141559151S.gif)

```c++
class KeyBoard {
public:
	virtual void show() {};
};

class KeyBoardMicro : public KeyBoard {
public:
	virtual void show() {
		cout << "micro keyboard" << endl;
	}
};

class KeyBoardMac : public KeyBoard {
public:
	virtual void show() {
		cout << "mac kayboard" << endl;
	}
};

class Mouse {
public:
	virtual void show() {};
};

class MouseMicro : public Mouse {
public:
	virtual void show() {
		cout << "micro mouse" << endl;
	}
};

class MouseMac : public Mouse {
public:
	virtual void show() {
		cout << "mac mouse" << endl;
	}
};

class Factory {
public:
	virtual KeyBoard* createKeyBoard() {};
	virtual Mouse* createMouse() {};
};

class FactoryMicro : public Factory {
public:
	virtual KeyBoard* createKeyBoard() {
		return new KeyBoardMicro;
	}
	virtual Mouse* createMouse() {
		return new MouseMicro;
	}
};

class FactoryMac : public Factory {
public:
	virtual KeyBoard* createKeyBoard() {
		return new KeyBoardMac;
	}
	virtual Mouse* createMouse() {
		return new MouseMac;
	}
};
```
##### 建造者模式
将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。
> 建造者模式与工厂模式的关注点不同：即按照这模式注重零部件的组装过程，而工厂方法模式更注重零部件的创建过程，但两者可以结合使用。

`优点`
- 扩展性好，各个具体的建造者相互独立

 `缺点`
 - 产品的组成部分必须相同，限制了使用范围
 - 产品内部发生变化，建造者要同步修改，维护成本大

![Build](http://c.biancheng.net/uploads/allimg/181114/3-1Q1141H441X4.gif)

```c++
class Product {
public:
	void setPartA(const string TpartA) {
		partA = TpartA;
	}
	void setPartB(const string TpartB) {
		partB = TpartB;
	}
	void setPartC(const string TpartC) {
		partC = TpartC;
	}
private:
	string partA;
	string partB;
	string partC;
};

class Builder {
public:
	Builder() {
		pro = new Product;
	}
	virtual ~Builder() {
		if (pro != nullptr)
			delete pro;
	}
	virtual void buildPartA() {};
	virtual void buildPartB() {};
	virtual void buildPartC() {};
	Product* getPro() {
		return pro;
	}
protected:
	Product* pro;
};

class ConcreteBuilder : public Builder {
public:
	virtual void buildPartA() {
		pro->setPartA("partA");
	};
	virtual void buildPartB() {
		pro->setPartB("partB");
	};
	virtual void buildPartC() {
		pro->setPartC("partC");
	}
};

class Diretor {
public:
	Diretor() {
		builder = new Builder;
	}
	virtual ~Diretor() {
		if (builder != nullptr)
			delete builder;
	}
	Product* construct() {
		builder->buildPartA();
		builder->buildPartB();
		builder->buildPartC();
		return builder->getPro();
	}
private:
	Builder* builder;
};
```

### 结构型
分为类结构型模式和对象结构型模式，前者采用继承机制来组织接口和类，后者采用组合或聚合来组合对象

##### 代理模式
为某对象提供一种代理以控制对该对象的访问。即客户端通过代理间接地访问该对象，从而限制、增强或修改该对象的一些特性。
`优点`
- 在客户端和目标对象之间起中介作用和保护目标对象的作用
- 扩展目标对象的功能
- 将客户端和目标对象分离，降低耦合度

`缺点`
- 造成系统设计中类的数量增加
- 造成请求处理速度变慢

![proxy](http://c.biancheng.net/uploads/allimg/181115/3-1Q115093011523.gif)

```c++
//抽象主题
class subject {
public:
	virtual void request() const {
		cout << "Proxy Test" << endl;
	};
	virtual ~subject() {};
};

//真实主题
class realSubject : public subject {
public:
	void request() {
		cout << "real subject" << endl;
	};
};

//代理，代码增强
class proxy : public subject {
public:
	void request(){
		if (realsubject == nullptr) {
			realsubject = new realSubject();
		}
		cout << "访问前的预处理" << endl;
		realsubject->request();
		cout << "访问后的后续处理" << endl;
	}
	~proxy() {
		if (realsubject != nullptr)
			delete realsubject;
	}
private:
	realSubject* realsubject = nullptr;
};
```
##### 适配器模式
将一个类的借口转换成客户希望的另外一个借口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作
`优点`
- 客户端通过适配器可以透明地调用目标接口
- 复用现存的类
- 将目标类和适配者类解耦

`缺点`
- 过多适配器会使系统代码凌乱

类适配器模式
![class adapter](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151045351c.gif)
对象适配器模式
![object adapter](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151046105A.gif)

```c++
//目标接口
class Target {
public:
	virtual void request() {};
};

//适配者接口
class Adaptee {
public:
	virtual int specificRequest(int num) {
		return num;
	}
};

//类适配器
class ClassAdapter : public Target, public Adaptee {
public:
	virtual void request() {
		Adaptee::specificRequest(0);
	}
};

//对象适配器
class ObjectAdapter : public Target {
public:
	ObjectAdapter(Adaptee Tadaptee) {
		this->adaptee = Tadaptee;
	}
	void request() {
		adaptee.specificRequest(0);
	}
private:
	Adaptee adaptee;
};
```

##### 桥接模式
将抽象与现实分离，使它们可以独立变化。它是用组合关系替代继承关系来实现的，从而降低了抽象与实现这两个可变维度的耦合度。
`优点`
- 扩展力强，实现细节对客户透明

`缺点`
- 聚合关系在抽象层，增加系统理解和设计难度

![Bridge](http://c.biancheng.net/uploads/allimg/181115/3-1Q115125253H1.gif)

```c++
class Implementor {
public:
	virtual void Ope() {};
};

class ConcreteImplementor : public Implementor {
public:
	virtual void Ope() {
		cout << "ConcreteImplementor" << endl;
	}
};

//组合关系，抽象化
class Abstraction {
public:
	Abstraction(Implementor* Timple) {
		imple = Timple;
	}
	virtual ~Abstraction() {
		if (imple != nullptr)
			delete imple;
	}
	virtual void Ope() {};
protected:
	Implementor* imple;
};

class RefinedAbstraction : public Abstraction {
public:
	explicit RefinedAbstraction(Implementor* Timple) :
		Abstraction(Timple) {}
	//扩展抽象化
	virtual void Ope() {
		cout << "refined abstraction  ";
		imple->Ope();
	}
};
```
##### 装饰模式
动态地给对象增加一些职责，即增加其额外的功能
> 与桥接模式的区别
装饰器模式是为了动态地给一个对象增加功能，桥接模式是为了让类在多个维度上自由扩展
装饰器模式的装饰者和被装饰者需要继承自同一父类，桥接模式通常不需要
装饰器模式通常可以嵌套使用，桥接模式不能

`优点`
- 比继承灵活，动态扩展对象功能，即插即用

`缺点`
- 过度使用会增加程序复杂性

![Decorator](http://c.biancheng.net/uploads/allimg/181115/3-1Q115142115M2.gif)

```c++
class Component {
public:
	virtual void Ope() {};
};

class ConcreteImplementor : public Component {
public:
	ConcreteImplementor() {};
	virtual void Ope() {
		cout << "concrete implementor ope" << endl;
	}
};

//抽象装饰角色
class Decorator : public Component {
public:
	Decorator(Component* Tcomponent) {
		component = Tcomponent;
	}
	~Decorator() {
		if (component != nullptr)
			delete component;
	}
	virtual void Ope() {
		cout << "Decorator   ";
		component->Ope();
	}
private:
	Component* component;
};

//具体装饰角色
class ConcreteDecorator : public Decorator {
public:
	ConcreteDecorator(Component* Tconponent) :
		Decorator(Tconponent) {};
	virtual void Ope() override {
		Decorator::Ope();
		cout << "orther operation" << endl;
		//...
	}
};

int main() {
	Component* c = new ConcreteImplementor();
	c->Ope();
	cout << "=====" << endl;
	Component* d = new ConcreteDecorator(c);
	d->Ope();
	return 0;
}
```
##### 外观模式
为多个复杂的子系统提供一个一致的接口，使这些子系统更加容易被访问
`优点`
- 降低子系统和客户端之间的耦合度
- 对客户端屏蔽子系统组件
`缺点`
- 无法限制客户使用子系统类
-增加子系统可能需要修改外观类或客户端代码

![Facade](http://c.biancheng.net/uploads/allimg/181115/3-1Q115152143509.gif)

```c++
class SubSystem01 {
public:
	void method() {
		cout << "subsystem01" << endl;
	}
};

class SubSystem02 {
public:
	void method() {
		cout << "subsystem02" << endl;
	}
};

class SubSystem03 {
public:
	void method() {
		cout << "subsystem03" << endl;
	}
};

class Facade {
public:
	Facade() {
		s1 = new SubSystem01;
		s2 = new SubSystem02;
		s3 = new SubSystem03;
	}
	virtual ~Facade() {
		if (s1 != nullptr)
			delete s1;
		if (s2 != nullptr)
			delete s2;
		if (s3 != nullptr)
			delete s3;
	}
	void method() {
		s1->method();
		s2->method();
		s3->method();
	}
private:
	SubSystem01* s1;
	SubSystem02* s2;
	SubSystem03* s3;
};
```
##### 享元模式
运用共享技术来有效地支持大量细粒度对象的复用
`优点`
- 共享相同对象，降低系统中细粒度对象给内存的压力

`缺点`
- 不能共享的状态外部化，增加程序的复杂性

![Flyweifht](http://c.biancheng.net/uploads/allimg/181115/3-1Q115161342242.gif)

```c++
//非享元角色
//不可以共享的外部状态，它以参数的形式注入具体享元的相关方法中
class UnsharedConcreateFlyweigth {
public:
	UnsharedConcreateFlyweigth(string Tinfo) {
		info = Tinfo;
	}
	string getInfo() {
		return info;
	}
	void setInfo(string Tinfo) {
		info = Tinfo;
	}
private:
	string info;
};

//抽象享元接口
class Flyweight {
public:
	virtual void Ope(UnsharedConcreateFlyweigth*) {};
};

class ConcreteFlyweight : public Flyweight {
public:
	ConcreteFlyweight(string Tkey) {
		key = Tkey;
	}
	void Ope(UnsharedConcreateFlyweigth* state) {
		cout << "unshared concreate flyweigth " << state->getInfo();
	}
private:
	string key;
};

//享元工厂角色
class FlyweightFactory {
public:
	Flyweight* getFlyweight(string key) {
		auto eFly = mp.find(key);
		if (eFly != mp.end()) {
			cout << "key已存在并成功获取" << endl;
			return eFly->second;
		}
		else {
			Flyweight* fly = new ConcreteFlyweight(key);
			mp.insert(make_pair(key, fly));
			cout << "add key " << key << endl;
			return fly;
		}
	}
private:
	unordered_map<string, Flyweight*> mp;
};

int main() {
	FlyweightFactory ff;
	Flyweight* p1 = ff.getFlyweight("I am A");
	Flyweight* p2 = ff.getFlyweight("I am B");
	Flyweight* p3 = ff.getFlyweight("I am A");
	cout << typeid(*p1).name() << endl;//ConcreteFlyweight
	p1->Ope(new UnsharedConcreateFlyweigth("unshared"));
	return 0;
}
```
##### 组合模式
将对象组合成树状层次结构，使用户对单个对象和组合对象具有一致的访问性
`优点`
- 客户端可以一致处理单个对象和组合对象
- 组合体中可加入新对象，而不影响客户端

`缺点`
- 不容易限制容器中的构件
- 不容易用继承方法来增加构件的新功能

透明方式  
树叶构件没有抽象构建的部分方法，却要实现它们，存在安全性问题
![Composite Pattern](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151G62L17.gif)
安全方式
失去透明性
![Composite Pattern](http://c.biancheng.net/uploads/allimg/181115/3-1Q1151GF5221.gif)

```c++
//透明方式
class Component {
public:
	Component() {};
	Component(const Component&) = default;
	virtual void add(Component*) {};
	virtual void remove(Component*) {};
	virtual void operation() = 0;
	bool operator==(const Component& c) {
		if (typeid(c) == typeid(Component))
			return true;
		return false;
	}
};

class Leaf : public Component {
public:
	Leaf(string Tname) {
		name = Tname;
	}
	Leaf(const Leaf& lf) {
		name = lf.name;
	};
	virtual void operation() {
		cout << "Leaf  " << name << endl;
	}
	bool operator==(const Leaf& l) {
		if (name == l.getName())
			return true;
		return false;
	}

	string getName() const {
		return name;
	}
private:
	string name;
};

//树枝构件
class Composite : public Component {
public:
	Composite() {
		child = new vector<Component*>;
	}
	Composite(const Composite& cp) {
		child = new vector<Component*>(*cp.child);
	};

	virtual ~Composite() {
		if (child != nullptr)
			delete child;
	}
	virtual void add(Component* c) {
		child->emplace_back(c);
	}
	//想处理Composite，需要给该类赋予特定标识
	//如Leaf的string字段
	virtual void remove(Component* c) {
		for (auto Itor = child->begin(); Itor != child->end(); ++Itor) {
			//only leaf can be remove
			if (typeid(**Itor) == typeid(Leaf) &&
				*dynamic_cast<Leaf*>(*Itor) == *dynamic_cast<Leaf*>(c)) {
				child->erase(Itor);
				break;
			}
			(*Itor)->remove(c);
		}
	}
	void operation() {
		cout << "I am composite" << endl;
		for (auto& i : (*child)) {
			i->operation();
		}
	}
	bool operator==(const Composite& c) {
		if (typeid(c) == typeid(Composite))
			return true;
		return false;
	}
private:
	vector<Component*>* child;
};

int main() {
	Composite* c0 = new Composite();
	Composite* c1 = new Composite();
	Leaf* leaf1 = new Leaf("1");
	Leaf* leaf2 = new Leaf("2");
	Leaf* leaf3 = new Leaf("3");
	c0->add(leaf1);
	c0->add(c1);
	c1->add(leaf2);
	c1->add(leaf3);
	c0->operation();
	cout << "==========" << endl;
	c0->remove(leaf3);
	c0->operation();
	return 0;
}
```
```c++
//安全方式只需修改以下代码
//Component
class Component {
public:
	Component() {};
	Component(const Component&) {};
	virtual void operation() = 0;
	bool operator==(const Component& c) {
		if (typeid(c) == typeid(Component))
			return true;
		return false;
	}
};
//树枝构件下的
	void remove(Component* c) {
		for (auto Itor = child->begin(); Itor != child->end(); ++Itor) {
			//only leaf can be remove
			if (typeid(**Itor) == typeid(Leaf) &&
				*dynamic_cast<Leaf*>(*Itor) == *dynamic_cast<Leaf*>(c)) {
				child->erase(Itor);
				break;
			}
			if(typeid(**Itor) == typeid(Composite))
				dynamic_cast<Composite*>(*Itor)->remove(c);
		}
	}
```
### 行为型
分为类行为模式和对象行为模式，前者采用继承机制来在类间分派行为，后者采用组合或聚合在对象间分配行为
##### 模板方法模式
定义一个操作中的算法骨架，将算法的一些步骤延迟到子类中，使得子类在可以不改变该算法结构的情况下重定义该算法的某些特定步骤
`优点`
- 父类中提取公共部分代码，便于复用
- 部分方法由子类实现，便于扩展

`缺点`
- 对于不同的实现都需要定义子类，增加类的个数
- 继承关系的缺点，父类增加抽象方法，子类都要改

![Template](http://c.biancheng.net/uploads/allimg/181116/3-1Q116095405308.gif)

```c++
class AbstractClass {
public:
	//模板方法
	virtual void TemplateMethod() {
		SpecificMethod();
		abstractMethod1();
		abstractMethod2();
	};
	//具体方法
	virtual void SpecificMethod() {
		cout << "abstract class method" << endl;
	};
	//抽象方法
	virtual void abstractMethod1() = 0;
	virtual void abstractMethod2() = 0;
};

//具体子类
class ConcreteClass : public AbstractClass {
public:
	virtual void abstractMethod1() {
		cout << "abstract method 1" << endl;
	};
	virtual void abstractMethod2() {
		cout << "abstract method 2" << endl;
	}
};
```
##### 策略模式
定义了一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的改变不会影响使用算法的客户
`优点`
- 多种策略可避免多重条件语句
- 把算法的使用放到环境类，实现移到具体策略类中，实现二者的分离

`缺点`
- 策略类的增多，造成维护困难

![Strategy](http://c.biancheng.net/uploads/allimg/181116/3-1Q116103K1205.gif)

```c++
//抽象策略类
class Strategy {
public:
	Strategy() = default;
	virtual void strategyMethod() {};
};

class ConcreteStragyA : public Strategy {
public:
	virtual void strategyMethod() {
		cout << "concrete stragy A" << endl;
	};
};

class ConcreteStragyB : public Strategy {
public:
	virtual void strategyMethod() {
		cout << "concrete stragy B" << endl;
	}
};

//环境类
class Context {
public:
	Strategy* getStrategy() {
		return strategy;
	}
	void setStrategy(Strategy* strategy) {
		this->strategy = strategy;
	}
	void strategyMethod() {
		strategy->strategyMethod();
	}
private:
	Strategy* strategy;
};
```
##### 命令模式
将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分隔开
`优点`
- 引入中间件（抽象接口）降低系统的耦合度
- 增删命令不影响其他类
- 可实现宏命令。命令模式与组合模式结合，将多个命令装配成组合命令，即宏命令
- 方便实现Undo和Redo操作。与备忘录模式结合，实现命令的撤销和恢复

`缺点`
- 可能产生大量具体的命令类

![command](http://c.biancheng.net/uploads/allimg/181116/3-1Q11611335E44.gif)

```c++
//接受者
class Receiver {
public:
	void action() {
		cout << "I am receiver" << endl;
	}
};

//抽象命令
class Command {
public:
	virtual void execute() {};
};

//具体命令
class ConcreteCommand : public Command {
public:
	virtual void execute() {
		rec.action();
	};
private:
	Receiver rec;
};

//调用者
class Invoker {
public:
	Invoker(Command* command) {
		this->command = command;
	}
	void setCommand(Command* command) {
		this->command = command;
	}
	void call() {
		cout << "Ivoker excuting command" << endl;
		command->execute();
	}
private:
	Command* command;
};
```
##### 职责链模式（责任链模式）
把请求从链中的一个对象传到下一个对象，直到请求被响应为止。通过这种方法去除对象之间的耦合
> 责任链模式的独到之处是将其节点处理者组合成了链式结构，并允许节点自身决定是否进行请求处理和转发，相当于让请求流动起来

`优点`
- 简化对象之间的连接
- 责任分担

`缺点`
- 不保证每个请求一定被处理
- 较长的职责链影响性能
- 职责链建立的合理性要靠客户端来保证，要避免循环调用

结构图
![responsibility](http://c.biancheng.net/uploads/allimg/181116/3-1Q116135Z11C.gif)
责任链
![](http://c.biancheng.net/uploads/allimg/181116/3-1Q11613592TF.gif)

```c++
class Handler {
public:
	void setNext(Handler* next) {
		this->next = next;
	}
	Handler* getNext() {
		return next;
	}
	virtual void handleRequest(string request) = 0;
private:
	Handler* next;
};

class ConcreteHandler1 : public Handler {
public:
	virtual void handleRequest(string request) {
		if (request == "1") {
			cout << "handle 1" << endl;
		}
		else {
			if (getNext() == nullptr)
				cout << "nobody resolve the question" << endl;
			else
				getNext()->handleRequest(request);
		}
	}
};

class ConcreteHandler2 : public Handler {
public:
	virtual void handleRequest(string request) {
		if (request == "2") {
			cout << "handle 2" << endl;
		}
		else {
			if (getNext() == nullptr)
				cout << "nobody resolve the question" << endl;
			else
				getNext()->handleRequest(request);
		}
	}
};

int main() {
	//组装责任链
	Handler* h1 = new ConcreteHandler1();
	Handler* h2 = new ConcreteHandler2();
	h1->setNext(h2);

	h1->handleRequest("2");
	return 0;
}
```

##### 状态模式
允许一个对象在其内部状态发生改变时改变其行为能力

`优点`
- 将状态转换显示化，减少对象间的互相依赖

`缺点`
- 对开闭原则的支持不太好

![state](http://c.biancheng.net/uploads/allimg/181116/3-1Q11615412U55.gif)

```c++
//环境类
class Context {
public:
	Context();
	void setState(State* state);
	State* getState() {
		return state;
	}
	void Handle();
private:
	State* state;
};

class State {
public:
	virtual void Handle(Context* context) = 0;
};

class ConcreteStateA : public State {
public:
	virtual void Handle(Context* context);
};

class ConcreteStateB : public State {
public:
	virtual void Handle(Context* context);
};

Context::Context() {
	state = new ConcreteStateA();//init
}
void Context::setState(State* state) {
	this->state = state;
};
void Context::Handle() {
	state->Handle(this);
};
void ConcreteStateA::Handle(Context* context) {
	cout << "I am StateA" << endl;
	context->setState(new ConcreteStateB());
};
void ConcreteStateB::Handle(Context* context) {
	cout << "I am StateB" << endl;
	context->setState(new ConcreteStateA());
};
```
##### 观察者模式
多个对象间存在一对多关系，当一个对象发生改变时，把这种改变通知给其他多个对象，从而影响其他对象的行为
`优点`
- 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系，符合依赖倒置原则
- 目标与观察者之间建立了一套触发机制

`缺点`
- 目标与观察值之间的依赖关系并没有完全解除，有可能出现循环引用
- 观察者对象很多时，通知的发布费时

![observer](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161A6221S.gif)

```c++
class Observer {
public:
	virtual void response() {};
};

class ConcreteObserve1 : public Observer {
public:
	virtual void response() {
		cout << "observer 1 response" << endl;
	}
};

class ConcreteObserve2 : public Observer {
public:
	virtual void response() {
		cout << "observer 2 response" << endl;
	}
};

//抽象目标
class Subject {
public:
	void add(Observer* observer) {
		observers.emplace_back(observer);
	}
	void remove(int i) {
		if(i < observers.size())
			observers.erase(observers.begin() + i);
	}
	virtual void notifyObserver() = 0;
protected:
	vector<Observer*> observers;
};

class ConcreteSubject : public Subject {
public:
	virtual void notifyObserver() {
		cout << "具体目标发生改变 ========" << endl;
		for (auto& i : observers) {
			i->response();
		}
	}
};

int main() {
	Subject* s = new ConcreteSubject();
	Observer* obs1 = new ConcreteObserve1();
	Observer* obs2 = new ConcreteObserve2();
	s->add(obs1);
	s->add(obs2);
	s->notifyObserver();
	cout << "============" << endl;
	s->remove(1);
	s->notifyObserver();
	return 0;
}
```
##### 中介者模式
定义一个中介对象来简化原有对象之间的交互关系，降低系统中对象间的耦合度，使原有对象之间不必相互了解
`优点`
- 将对象间的一对多关联转变为一对一关联，提高系统的灵活性

`缺点`
- 中介者模式将原本多个对象的直接相互依赖变成了中介者和多个同事类的依赖关系。当同事类越多，中介者就会越臃肿

![meidator](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161I532V0.gif)

```c++
class Colleague;

//抽象中介者
class Mediator {
public:
    virtual void reg(Colleague* colleague) {};
    virtual void relay(Colleague* cl) = 0;//转发
};

//具体中介者
class ConcreteMediator : public Mediator {
public:
    void reg(Colleague* col) override;
    void relay(Colleague* cl) override;
private:
    set<Colleague*> colleagues;
};

class Colleague {
public:
    void setMedium(Mediator* Tmediator) {
        this->mediator = Tmediator;
    }
    virtual void receive() = 0;
    virtual void send() = 0;
protected:
    Mediator* mediator{};
};

//具体同事类
class ConcreteColleague1 : public Colleague {
public:
    void receive() override {
        cout << "I am concrete colleague 1" << endl;
    };
    void send() override {
        cout << "colleague 1 send" << endl;
        mediator->relay(this);
    }
};
class ConcreteColleague2 : public Colleague {
public:
    void receive() override {
        cout << "I am concrete colleague 2" << endl;
    };
    void send() override {
        cout << "colleague 2 send" << endl;
        mediator->relay(this);
    }
};

void ConcreteMediator::reg(Colleague *col) {
    auto p = colleagues.insert(col);
    if(p.second)
        col->setMedium(this);
};

void ConcreteMediator::relay(Colleague *cl) {
    for(auto Itor : colleagues) {
        if (Itor != cl)
            Itor->receive();
    }
}

int main() {
    Mediator* md = new ConcreteMediator();
    Colleague* c1 = new ConcreteColleague1();
    Colleague* c2 = new ConcreteColleague2();
    md->reg(c1);
    md->reg(c2);
    c1->send();
    cout << "======" << endl;
    c2->send();
    return 0;
}
```
##### 迭代器模式
提供一种方法来顺序访问聚合对象中的一系列数据，而不暴露聚合对象的内部表示
`优点`
- 访问一个聚合对象的内容无需暴漏其内部表示
- 遍历任务交由迭代器完成，这简化了聚合类
- 支持以不同方式遍历一个聚合，甚至可以自定义迭代器的子类以支持新的遍历

![iterator](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161PU9528.gif)

```c++
template<typename T>
class Iterator {
public:
    virtual T firstItem() = 0;
    virtual T nextItem() = 0;
    virtual T currentItem() = 0;
    virtual bool isEnd() = 0;
};

//抽象聚合类
template<typename T>
class Aggregate {
public:
    virtual unsigned count() = 0;
    virtual void pushItem(const T& str) = 0;
    virtual T removeItem(const int index) = 0;
    virtual Iterator<T>* createIterator() = 0;
};

template<typename T>
class ConcreteAggregate : public Aggregate<T> {
public:
    ConcreteAggregate(): m_pIter(nullptr){};
    ~ConcreteAggregate() {
        if(m_pIter) {
            delete m_pIter;
            m_pIter = nullptr;
        }
    }
    unsigned count() override {
        return m_listItem.size();
    }
    void pushItem(const T& str) override {
        m_listItem.template emplace_back(str);
    }
    T removeItem(const int index) override {
        if(index < count())
            return m_listItem[index];
        return T{};
    };
    Iterator<T>* createIterator() override;
private:
    vector<T> m_listItem;
    Iterator<T> *m_pIter;
};

template<typename T>
class ConcreteIterator : public Iterator<T> {
public:
    explicit ConcreteIterator(Aggregate<T> *pa):
    cnt(0), m_pConcreteA(pa) {}
    T firstItem() override {
        return m_pConcreteA->removeItem(0);
    }
    T nextItem() override {
        if(cnt >= m_pConcreteA->count())
            return {};
        return m_pConcreteA->removeItem(cnt++);
    }
    T currentItem() override {
        return m_pConcreteA->removeItem(cnt);
    }
    bool isEnd() override {
        return (cnt >= m_pConcreteA->count());
    }
private:
    Aggregate<T> *m_pConcreteA;
    int cnt;
};

template<typename T>
Iterator<T>* ConcreteAggregate<T>::createIterator()  {
    return new ConcreteIterator<T>(this);
}

int main() {
    ConcreteAggregate<string>* p = new ConcreteAggregate<string>();
    p->pushItem("aaa");
    p->pushItem("bbb");
    Iterator<string> *it = p->createIterator();
    while(!it->isEnd()) {
        cout << it->currentItem() << endl;
        it->nextItem();
    }
    return 0;
}
```
##### 访问者模式
在不改变集合元素的前提下，为一个集合中的每个元素提供多种访问方式，即每个元素有多个访问者对象访问
`优点`
- 扩展性，复用性，灵活性好

`缺点`
- 新增元素类很困难
- 破坏封装
- 违反依赖倒置原则

![visitor](http://c.biancheng.net/uploads/allimg/181119/3-1Q11910135Y25.gif)

```c++
class Element;
//抽象访问者
class Visitor {
public:
    virtual void visit(Element*) = 0;
};

//具体访问者
class ConcreteVisitorA : public Visitor {
public:
    void visit(Element* el) override;
};
class ConcreteVisitorB : public Visitor {
public:
    void visit(Element* el) override;
};

//抽象元素类
class Element {
public:
    virtual void accept(Visitor*) = 0;
};
//具体元素类
class ConcreteElementA : public Element {
public:
    void accept(Visitor* vis) override {
        vis->visit(this);
    };
    string operationA() {
        return "具体元素A的操作";
    }
};
class ConcreteElementB : public Element {
public:
    void accept(Visitor* vis) override {
        vis->visit(this);
    };
    string operationB() {
        return "具体元素B的操作";
    }
};

//对象结构
class ObjectStructure {
public:
    void accept(Visitor* vis) {
        for(auto& Iter: list)
            Iter->accept(vis);
    }
    void add(Element* ele) {
        list.insert(ele);
    }
    void remove(Element* ele) {
        list.erase(ele);
    }
private:
    set<Element*> list;
};

void ConcreteVisitorA::visit(Element*el) {
    if(typeid(el) == typeid(ConcreteElementA))
        cout << "具体访问者A访问  " << dynamic_cast<ConcreteElementA*>(el)->operationA() << endl;
    else
        cout << "具体访问者A访问  " << dynamic_cast<ConcreteElementB*>(el)->operationB() << endl;
};
void ConcreteVisitorB::visit(Element*el) {
    if(typeid(el) == typeid(ConcreteElementA))
        cout << "具体访问者B访问  " << dynamic_cast<ConcreteElementA*>(el)->operationA() << endl;
    else
        cout << "具体访问者B访问  " << dynamic_cast<ConcreteElementB*>(el)->operationB() << endl;
};

int main() {
    ObjectStructure os;
    os.add(new ConcreteElementA());
    os.add(new ConcreteElementB());
    os.accept(new ConcreteVisitorA());
    cout << "================" << endl;
    os.accept(new ConcreteVisitorB());
    return 0;
}
```
##### 备忘录模式（快照模式）
在不破坏封装性的前提下，获取并保存一个对象的内部状态，以便以后恢复它
`优点`
- 提供了一种可恢复状态的机制
- 简化了发起人类

`缺点`
- 资源消耗大

![memento](http://c.biancheng.net/uploads/allimg/181119/3-1Q119130413927.gif)

```c++
//备忘录
class Memento {
public:
    Memento() = default;
    Memento(string state) {
        this->state = state;
    }
    void setState(string state) {
        this->state = state;
    }
    string getState() {
        return state;
    }
private:
    string state;
};

//发起人
class Originator {
public:
    void setState(string state) {
        this->state = state;
    }
    string getState() {
        return state;
    }
    Memento createMemento() {
        return Memento(state);
    }
    void restoreMemento(Memento m) {
        this->setState(m.getState());
    }
private:
    string state;
};

//管理者
class Caretaker {
public:
    void setMemento(Memento m) {
        mem = m;
    }
    Memento getMemento() {
        return mem;
    }
private:
    Memento mem;
};

int main() {
    Originator og;
    Caretaker ct;
    og.setState("A");
    cout << og.getState() << endl;
    ct.setMemento(og.createMemento());//保存状态
    og.setState("B");
    cout << og.getState() << endl;
    og.restoreMemento(ct.getMemento());//回复状态
    cout << og.getState() << endl;
    return 0;
}
```
##### 解释器模式
提供如何定义语言的文法，以及对语言句子的解释方法，即解释器
`优点`
- 扩展性好

`缺点`
- 执行效率低
- 引起类膨胀

![expression](http://c.biancheng.net/uploads/allimg/181119/3-1Q119150626422.gif)

```c++
class AbstractExpression {
public:
    virtual void interpret(string info);//解释方法
};

class TerminalExpression: public AbstractExpression {
public:
    void interpret(string info) override {
        //对终结符表达式的处理
    }
};

class NonterminalExpression: public AbstractExpression {
public:
    void interpret(string info) override {
        //对非终结符表达式的处理
    }
};

class Context {
public:
    Context() = default;//init
    void operation(string info) {
        //解释方法
    }
};
```