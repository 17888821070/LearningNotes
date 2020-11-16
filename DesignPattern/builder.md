# 建造者模式

假设有个类，有几个成员变量是可配置项，这些可配置项可以放入构造函数，但如果可配置项太多，造成构造函数复杂且也会带来参数传递错误的可能，于是可以把初始化时必要的变量放入构造函数，其他参数通过 `set()` 函数来设置，让使用者自主选择填写或不填写

如果可选参数之间有约束条件则没有地方放检验逻辑的代码

使用 `set()` 函数使得对象有处于无效的状态的风险，因为只有当所有参数都被 `set()` 之后对象才有效

暴露出的 `set()` 函数又会导致参数可能在初始化后再次被修改

建造者模式先一步一步设置建造者的变量，然后再一次性地创建对象，让对象一直处于有效状态

建造者模式可以更精细的控制构建过程

```cpp

class Builder {
public:
	virtual void BuildHead() = 0;
	virtual void BuildBody() = 0;
	virtual void BuildLeftArm() = 0;
	virtual void BuildRightArm() = 0;
	virtual void BuildLeftLeg() = 0;
	virtual void BuildRightLeg() = 0;
};
//构造瘦人
class ThinBuilder : public Builder {
public:
	void BuildHead() {}
	void BuildBody() {}
	void BuildLeftArm() {}
	void BuildRightArm() {}
	void BuildLeftLeg() {}
	void BuildRightLeg() {}
};
//构造胖人
class FatBuilder : public Builder {
public:
	void BuildHead() {}
	void BuildBody() {}
	void BuildLeftArm() {}
	void BuildRightArm() {}
	void BuildLeftLeg() {}
	void BuildRightLeg() {}
};
//构造的指挥官
class Director {
private:
	Builder *m_pBuilder;
public:
	Director(Builder *builder) {
        m_pBuilder = builder; 
    }
	void Create(){
		m_pBuilder->BuildHead();
		m_pBuilder->BuildBody();
		m_pBuilder->BuildLeftArm();
		m_pBuilder->BuildRightArm();
		m_pBuilder->BuildLeftLeg();
		m_pBuilder->BuildRightLeg();
	}
};

int main() {
	FatBuilder thin;
	Director director(&thin);
	director.Create();
	return 0;
}
```

工厂模式是用来创建不同但是相关类型的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象

建造者模式是用来创建一种类型的复杂对象，通过设置不同的可选参数，定制化地创建不同的对象