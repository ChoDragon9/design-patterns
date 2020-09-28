---
layout: post
title:  "GoF 디자인 패턴 | 생성패턴"
date:   2020-09-27
description: GoF 디자인 패턴 중 생성패턴을 정리합니다.
tags: [pattern]
category: 패턴
sidebar: auto
---

# GoF 디자인 패턴 | 생성패턴
- 인스턴스를 만드는 절차를 추상화하는 패턴이다.
- 객체를 생성 및 합성하는 방법 또는 객체의 표현 방법을 시스템에서 분리해준다.

#### 활용성
- 생성패턴은 시스템이 어떤 구체 클래스를 사용하는 지에 대한 정보를 캡슐화한다.
- 생성패턴은 이들 클래스의 인스턴스들이 어떻게 만들고 서로 맞붙는지에 대한 부분을 완전히 가려준다.

## 추상 팩토리(Abstract Factory)
### 의도
구체적인 클래스를 지정하지 않고 관련성을 갖는 객체들의 집합을 생성하거나 서로 독립적인 객체들의 집합을 생성할 수 있는 인터페이스를 제공하는 패턴

### 활용성
- 여러 제품군 중 하나를 선택해서 시스템을 설정해야 하고 한번 구성한 제품을 다른 것으로 대체할 수 있을 때
- 관련된 제품 객체들이 함께 사용되로록 설계되었고, 이 부분에 대한 제약이 외부에도 지켜지도록 하고 싶을 때
- 제품에 대한 클래스 라이브러리를 제공하고, 그들의 구현이 아닌 인터페이스를 노출시키고 싶을 때

### 구조 및 구현
```ts
interface AbstractFactory {
    createProductA(): AbstractProductA
    createProductB(): AbstractProductB
}

class ConcreteFactory1 implements AbstractFactory {
    createProductA() {
        return new ProductA1()
    }
    createProductB() {
        return new ProductB1()
    }
}

class ConcreteFactory2 implements AbstractFactory {
    createProductA() {
        return new ProductA2()
    }
    createProductB() {
        return new ProductB2()
    }
}

interface AbstractProductA { }
interface AbstractProductB { }

class ProductA1 implements AbstractProductA {}
class ProductA2 implements AbstractProductA {}

class ProductB1 implements AbstractProductB {}
class ProductB2 implements AbstractProductB {}

```

#### 사용자측 코드
```ts
class Main {
    factory: AbstractFactory
    constructor() {
        this.factory = new ConcreteFactory1()
        this.factory.createProductA()
        this.factory.createProductB()
    }
}

// 다른 제품을 사용할 때
class Main {
    factory: AbstractFactory
    constructor() {
        this.factory = new ConcreteFactory2()
        this.factory.createProductA()
        this.factory.createProductB()
    }
}
```

#### 협력 방법
- `ConcreteFactory` 클래스의 인스턴스 한 개가 런타임에 만들어진다.
- `ConcreteFactory`는 어떤 특정 구현을 갖는 제품 객체를 생성한다.
- 서로 다른 제품을 생성하려면 서로 다른 `ConcreteFactory`를 사용해야 한다.

## 빌더(builder)
### 의도
복잡한 객체를 생성하는 방법과 표현하는 방법을 정의하는 클래스를 별도로 분리하여, 서로 다른 표현이라도 이를 생성할 수 있는 동일한 절차를 제공할 수 있도록 합니다.

### 활용성
- 합성할 객체들의 표현이 서로 다르더라도 생성 절차에서 이를 지원해야 할 때
- 복합 객체의 생성 알고리즘이 이를 합성하는 요소 객체들이 무엇인지 이들의 조립 방법에 독립적일 때

### 구조 및 구현
```ts
interface Builder {
    buildPart1(): void
    buildPart2(): void
}

class ConcreteBuilder implements Builder {
    private product = new Product()

    buildPart1() {
        this.product.setState1(10)
    }
    buildPart2() {
        this.product.setState2(20)
    }
    getResult() {
        return this.product
    }
}

class Product {
    private state1: number = 0
    private state2: number = 0

    setState1(state1: number) {
        this.state1 = state1
    }
    setState2(state2: number) {
        this.state2 = state2
    }
}

class Director {
    private builder: Builder

    constructor (builder: Builder) {
        this.builder = builder
    }
    construct() {
        this.builder.buildPart1()
        this.builder.buildPart2()
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const builder = new ConcreteBuilder()
        const director = new Director(builder)

        director.construct()
        builder.getResult()
    }
}
```

#### 협력 방법
- 사용자는 `Director`객체를 생성하고, 이렇게 생성한 객체를 자신이 원하는 `Builder`객체로 합성해 나간다.
- 제품의 일부가 구축될 때마다 `Director`는 Builder에 통보한다.
- `Builder`는 `Director`의 요청을 처리하여 제품에 부품을 추가한다.
- 사용자는 `Builder`에서 제품을 요청한다.

## 팩토리 메서드(Factory Method)
### 의도
객체를 생성하기 위해 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스가 내리도록 합니다.

### 활용성
- 어떤 클래스가 자신이 생성해야 하는 객체의 클래스를 예측할 수 없을 때
- 생성할 객체를 기술하는 책임을 자신의 서브클래스가 지정했으면 할 때

### 구조 및 구현
1. 구현 방법은 크게 두가지다. 추상 클래스 또는 구체 클래스.
2. 팩토리 메서드를 매개변수화 한다.
   - 매개변수를 받아 어떤 종류의 제품을 생성할 지 식별한다.

#### 추상 클래스
```ts
abstract class Creator {
    private product!: Product
    anOperation() {
        this.product = this.factoryMethod()
        this.product.doSomething()
    }
    abstract factoryMethod(): Product
}

interface Product {
    doSomething(): void
}
class ConcreteProduct1 implements Product {
    doSomething() {}
}
class ConcreteProduct2 implements Product {
    doSomething() {}
}

class ConcreteCreator1 extends Creator {
    factoryMethod() {
        return new ConcreteProduct1()
    }
}
class ConcreteCreator2 extends Creator {
    factoryMethod() {
        return new ConcreteProduct2()
    }
}
```
```ts
class Main {
    creator: Creator
    constructor() {
        this.creator = new ConcreteCreator1()
        this.creator.anOperation()
    }
}

// 다른 서브 클래스 사용
class Main {
    creator: Creator
    constructor() {
        this.creator = new ConcreteCreator2()
        this.creator.anOperation()
    }
}
```

#### 매개변수
```ts
type CreatorType = 1 | 2

class Creator {
    private product!: Product
    anOperation(type: CreatorType) {
        this.product = this.factoryMethod(type)
        this.product.doSomething()
    }
    factoryMethod(type: CreatorType): Product {
        switch (type) {
            case 1:
                return new ConcreteProduct1()
            case 2:
                return new ConcreteProduct2()
        }
    }
}

interface Product {
    doSomething(): void
}
class ConcreteProduct1 implements Product {
    doSomething() {}
}
class ConcreteProduct2 implements Product {
    doSomething() {}
}
```
```ts
class Main {
    creator: Creator
    constructor() {
        this.creator = new Creator()

        const product1 = this.creator.anOperation(1)
        const product2 = this.creator.anOperation(2)
    }
}
```

## 원형(Prototype)
### 의도
원형이 되는 인스턴스를 사용하여 생성할 객체의 종류를 명시하고, 이렇게 만든 견본을 복사해서 새로운 객체를 생성합니다.

### 활용성
- 제품의 생성, 복합, 표현 방법에 독립적인 제품을 만들고자 할 때
- 제품 클래스 계통과 병렬적으로 만드는 팩토리 클래스를 피하고 싶을 때
- 클래스로 꼭 정의 할 필요없는 경우, 클래스의 수를 줄이고 싶을 때

### 구조 및 구현
```ts
interface Prototype {
    clone(): Prototype
}

class ConcretePrototype implements Prototype {
    private state = 0
    constructor(origin?: ConcretePrototype) {
        if (origin) {
            this.state = origin.state
        }
    }
    clone() {
        return new ConcretePrototype(this)
    }
    addState (state: number) {
        this.state += state
    }
    log() {
        console.log(this.state)
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const product1 = new ConcretePrototype()
        product1.log() // 0

        const product2 = product1.clone()
        product2.addState(10)
        product2.log() // 10

        const product3 = product2.clone()
        product3.addState(10)
        product3.log() // 20
    }
}
```

## 싱글턴(Singleton)
### 의도
오직 한 개의 클래스 인스턴스만을 갖도록 보장하고, 이에 대한 전역적인 접근점을 제공합니다.

### 활용성
- 클래스의 인스턴스가 오직 하나여야 함을 보장하고, 잘 정의된 접근점으로 모든 사용자가 접근 할 수 있도록 해야 할 때
- 유일한 인스턴스가 서브 클래싱으로 확장 되어야 하며, 사용자는 코드의 수정없이 확장된 서브클래스의 인스턴스를 사용할 수 있어야 할 때
- 전역 변수를 사용하여 이름 공간을 망치는 일을 없애주기 때문에 단일체 패턴을 전역변수보다 이름공간을 좁힐 수 있다.

### 구조 및 구현
```ts
class Singleton {
    private static uniquInstance: Singleton
    
    private constructor() { }

    static instance(): Singleton {
        if (!this.uniquInstance) {
            this.uniquInstance = new Singleton()
        }
        return this.uniquInstance
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const product1 = Singleton.instance()
        const product2 = Singleton.instance()
        console.log(product1 === product2) // true
    }
}
```
