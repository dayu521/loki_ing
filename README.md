### AbstractFactory 

假如产品类如下所示:

```cpp
struct FUK
{
    virtual FUK * Clone()=0;
};

struct A :FUK
{
    A(){std::cout<<"A"<<std::endl;}
    virtual A * Clone()
    {
        return new A(*this);
    }
};

struct B:FUK
{
    B(){std::cout<<"B"<<std::endl;}
    virtual B * Clone()
    {
        return new B(*this);
    }
};

struct C:FUK
{
    C(){std::cout<<"C"<<std::endl;}

    // FUK interface
public:
    virtual auto Clone()->C * override
    {
        return new auto(*this);
    }
};

struct A1:A
{
    A1(){std::cout<<"A1"<<std::endl;}
};

struct B1:B
{
    B1(){std::cout<<"B1"<<std::endl;}
};

struct C1:C
{
    C1(){std::cout<<"C1"<<std::endl;}
};
```



- 基于`OpNewFactoryUnit`:

```cpp
Loki::ConcreteFactory<Loki::AbstractFactory<LOKI_TYPELIST_3(A,B,C)>,
	Loki::OpNewFactoryUnit,
	LOKI_TYPELIST_3(A1,B1,C1)> factory;	
auto a1=factory.Create<C>();	//直接通过new 产生对象
auto a2=factory.Create<A>();
auto a2=factory.Create<B>();
```

- 基于`PrototypeFactoryUnit:`

```cpp
Loki::ConcreteFactory<Loki::AbstractFactory<LOKI_TYPELIST_3(A,B,C)>,Loki::PrototypeFactoryUnit> factory;
Loki::DoSetPrototype<A>(factory,new A1);	//注册原型对象
Loki::DoSetPrototype<C>(factory,new C1);	//..
Loki::DoSetPrototype<B>(factory,new B1);	//..
auto a1=factory.Create<C>();	//通过Clone虚函数返回克隆的对象
auto a2=factory.Create<A>();
auto a2=factory.Create<B>();
```

> 我们可以在`PrototypeFactoryUnit`中发现SetPrototype模板函数,好像我们可以
>
> `factory.SetPrototype<A>(new A1);`
>
> 但我们只能设置最后一个类型,在这种情况下是
>
> `factory.SetPrototype<C>(new C1);`
>
> 因为类型推断只推断为C兼容的`PrototypeFactoryUnit`,所以我们继承来修改`PrototypeFactoryUnit`类

```cpp
template <class ConcreteProduct, class Base>
class MyPrototype:public Loki::PrototypeFactoryUnit <ConcreteProduct,Base>
{
public:
    template <class U>
    void SetPrototype(U* pObj);
};

//重新实现我们的SetPrototype()函数
template<class ConcreteProduct, class Base>
template<typename U>
void MyPrototype<ConcreteProduct,Base>::SetPrototype(U* pObj)
{ Loki::DoSetPrototype<U>(*this, pObj); }
```

从而可以如下使用:

```cpp
Loki::ConcreteFactory<Loki::AbstractFactory<LOKI_TYPELIST_3(A,B,C)>,Loki::PrototypeFactoryUnit> factory;
factory.SetPrototype<A>(new A1);
factory.SetPrototype<C>(new C1);
factory.SetPrototype<B>(new B1);
auto a1=factory.Create<C>();
auto a2=factory.Create<A>();
auto a2=factory.Create<B>();
```

