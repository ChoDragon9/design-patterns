---
layout: post
title:  "GoF 디자인 패턴 | 구조패턴"
date:   2020-09-27
description: GoF 디자인 패턴 중 구조패턴을 정리합니다.
tags: [pattern]
category: 패턴
sidebar: auto
---

# GoF 디자인 패턴 | 구조패턴

## 어뎁터(Adapter)
### 의도
클래스의 인터페이스를 사용자가 기대하는 인터페이스 형태로 적응시킵니다. 서로 일치하지 않는 인터페이스를 갖는 클래스들을 함께 동작시킵니다.

### 활용성
- 기존 클래스를 사용하고 싶은 데 인터페이스가 맞지 않을 때
- 이미 만든 것을 재사용하고 싶지만 이 재사용 가능한 라이브러리를 수정할 수 없을 때

### 구조 및 구현
#### 클래스 적응자
다중 상속을 활용해서 한 인터페이스를 다른 인터페이스로 적응시킵니다.
```ts
interface Target {
    request(): void
}

class Adaptee {
    specificRequest() {}
}

class Adapter extends Adaptee implements Target {
    request() {
        this.specificRequest()
    }
}
```

#### 사용자측 코드
```ts
class Main {
    target: Target
    constructor () {
        this.target = new Adapter()
        this.target.request()
    }
}
```

#### 객체 적응자
객체 합성을 써서 이루어져 있습니다.
```ts
interface Target {
    request(): void
}

class Adaptee {
    specificRequest() {}
}

class Adapter implements Target {
    private adaptee!: Adaptee
    constructor () {
        this.adaptee = new Adaptee()
    }
    request() {
        this.adaptee.specificRequest()
    }
}
```
#### 사용자측 코드
```ts
class Main {
    target: Target
    constructor () {
        this.target = new Adapter()
        this.target.request()
    }
}
```

## 브릿지(Bridge)
### 의도
구현에서 추상을 분리하여, 이들이 독립적으로 다양성을 가질 수 있도록 합니다.

### 활용성
- 추상적인 개념과 이에 대한 구현 사이의 지속적인 종속 관계를 피하고 싶을 때. 이를 테면, 런타임에 구현 방법을 선택하거나 구현 내용을 변경하고 싶을 때
- 추상적 개념과 구현 모두가 독립적으로 서브클래싱을 통해 확장되어야 할 때

### 구조 및 구현
```ts
interface Implementor {
    operationImp(): void
}

class ConcreteImplementor implements Implementor {
    operationImp() {}
}

class Abstraction {
    private imp: Implementor
    constructor() {
        this.imp = new ConcreteImplementor()
    }
    operation() {
        this.imp.operationImp()
    }
}
```

#### 사용자측 코드
```ts
class Main {
    abstraction: Abstraction
    constructor () {
        this.abstraction = new Abstraction()
        this.abstraction.operation()
    }
}
```

## 컴포지트(Composite)
### 의도
부분과 전체의 계층을 표현하기 위해 객체들을 모아 트리 구조로 구성합니다. 사용자로 하여금 개별 객체와 복합 객체를 모두 동일하게 다룰 수 있도록 하는 패턴입니다.

### 활용성
- 부분-전체의 객체 계통을 표현하고 싶을 때
- 사용자가 객체의 합성으로 생긴 복합 객체와 개개의 객체 사이의 차이를 알지 않고도 자기 일을 할 수 있도록 만들고 싶을 때, 사용자는 복합 구조의 모든 객체를 똑같이 취급하게 됩니다.

### 구조 및 구현
```ts
interface Component {
    operation(): void
    add?(component: Component): void
    remove?(component: Component): void
    getChild?(num: number): Component
}

class Leaf implements Component {
    operation() {}
}

class Composite implements Component {
    private children: Component[] = []
    operation() {}
    add(component: Component) {
        this.children.push(component)
    }
    remove(component: Component) {
        const index = this.children.indexOf(component)
        if (index > -1) {        
           this.children.splice(index, 1)
        }
    }
    getChild (num: number) {
        return this.children[num]
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor () {
        const root = new Composite()
        const parent = new Composite()

        root.add(new Leaf())
        root.add(new Leaf())
        root.add(parent)

        parent.add(new Leaf())

        console.log(root)
        // Composite
        //     children: Array(3)
        //         0: Leaf
        //         1: Leaf
        //         2: Composite
        //             children: Array(1)
        //                 0: Leaf
    }
}
```

## 데코레이터(Decorator)
### 의도
객체에 동적으로 새로운 책임을 추가할 수 있게 합니다. 기능을 추가하려면, 서브클래스를 생성하는 것보다 융통성 있는 방법을 제공합니다.

### 활용성
- 다른 객체에 영향을 주지 않고 각각의 객체에 새로운 책임을 추가하기 위해 사용합니다.
- 제거 될 수 있는 책임에 대해 사용합니다.
- 실제 상속으로 서브클래스를 계속 만드는 방법이 실질적이지 못할 때 사용합니다. 
  - 너무 많은 수의 독립된 확장이 가능할 때 모든 조합을 지원하기 위해 상속으로 해결하면 클래스 수가 폭발적으로 많아지게 된다.

### 구조 및 구현
1. 인터페이스 일치 시키기
   - 반드시 자신을 둘러싼 구성 요소의 인터페이스를 만족해야 한다.
2. Component 클래스는 가벼운 무게를 유지하기
   - 가볍게 정의한다는 의미는 연산에 해당하는 인터페이스만을 정의하고 무언가 저장할 수 있는 변수는 정의하지 말라는 의미다.

```ts
interface Component {
    operation(): void
}

class ConcreteComponent implements Component {
    operation() {
        console.log('ConcreteComponent.operation')
    }
}

abstract class Decorator implements Component {
    component: Component
    constructor (component: Component) {
        this.component = component
    }
    operation() {
        component.operation()
    }
}

class ConcreteDecorator extends Decorator {
    constructor(component: Component) {
        super(component)
    }
    operation() {
        super.operation()
        this.addedBehavior()
    }
    addedBehavior() {
        console.log('ConcreteDecorator.addedBehavior')
    }
}
```

#### 사용자측 코드
```ts
class Main {
    component: Component
    constructor() {
        this.component = new ConcreteComponent()
        this.component.operation()
        // ConcreteComponent.operation
    }
}

// Decorator 사용
class Main {
    component: Component
    constructor() {
        const concreteComponent = new ConcreteComponent()
        this.component = new ConcreteDecorator(component)
        this.component.operation()
        // ConcreteComponent.operation
        // ConcreteDecorator.addedBehavior
    }
}
```

## 퍼사드(Facade)
### 의도
한 서브시스템 내의 인터페이스 집합에 대한 획일화된 하나의 인터페이스를 제공하는 패턴으로, 서브시스템을 사용하기 쉽도록 상위 수준의 인터페이스를 정의합니다.

### 활용성
- 복잡한 서브시스템에 대한 단순한 인터페이스 제공이 필요할 때
- 추상 개념에 대한 구현 클래스와 사용자 사이에 너무 많은 종속성이 존재할 때
- 퍼사드의 사용을 통해 사용자와 다른 서브시스템 간의 결합도를 줄일 수 있다

### 구조 및 구현
```ts
class Facade {
    private subClass1 = new SubClass1()
    private subClass2 = new SubClass2()

    operation() {
        this.subClass1.operation()
    }
    request() {
        this.subClass2.request()
    }
}

class SubClass1 {
    operation() {}
}
class SubClass2 {
    request() {}
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const facade = new Facade()
        facade.operation()
        facade.request()
    }
}
```

## 플라이웨이트(Flyweight)
### 의도
공유를 통해 많은 수의 소립 객체들을 효과적으로 지원한다.

Flyweight 패턴에서 중요한 개념은 본질적 상태와 부가적 상태의 구분이다.
본질적 상태는 Flyweight객체에 저장되어야 하며, 이것이 적용되는 상황과
상관없는 본질적 특성 정보들이 객체를 구성한다.

본질적이지 않는 부가적 상태는 Flyweight객체가 사용된 상황에 따라 달라질 수 있고,
그 상황에 종속적이다. 그러므로 공유될 수 없습니다.

### 활용성
- 응용프로그램이 대량의 객체를 사용해야 할 때
- 객체의 수가 너무 많아져 저장 비용이 너무 높아질때

### 구조 및 구현
```ts
interface Flyweight {
    state: string
    operation(state: string): void
}

class ConcreteFlyweight implements Flyweight {
    state!: string
    operation(state: string) {
        this.state = state
    }
}

class UnshareConcreteFlyweight implements Flyweight {
    state!: string
    operation(state: string) {
        this.state = state
    }
}

class FlyweightFactory {
    flyweights: Map<string, Flyweight> = new Map()

    getFlyweight(key: string): Flyweight {
        const flyweight = this.flyweights.get(key)
        if (flyweight) {
            return flyweight
        } else {
            const newFlyweight = new ConcreteFlyweight()
            this.flyweights.set(key, newFlyweight)
            return newFlyweight
        }
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const flyweightFactory = new FlyweightFactory()
        const unsharedFlyweight = new UnshareConcreteFlyweight()
        const concreteFlyweight1 = flyweightFactory.getFlyweight('something')
        const concreteFlyweight2 = flyweightFactory.getFlyweight('something')

        console.log(concreteFlyweight1 === concreteFlyweight2) // true
    }
}
```

#### 협력 방법
- `Flyweight`객체가 기능을 수행하는 데 필요한 상태가 본질적인 것인지 부가적인것인지를 구분해야 한다.
  - 본질적인 상태는 `ConcreteFlyweight`에 저장해야 한다.
  - 부가적인 상태는 사용자가 저장하거나, 연산되어야 하는 다른 상태로 관리 해야 한다.
  - 사용자는 연산을 호출할 때 자신에게만 필요한 부가적 상태를 `Flyweight` 객체에 매개변수로 전달한다.
- 사용자는 `ConcreteFlyweight`의 인스턴스를 직접 만들 수 없다.
  - 사용자는 `ConcreteFlyweight`객체를 `FlyweightFactory`객체에서 언어야 한다. 이렇게 해야 `Flyweight`객체가 공유될 수 있다.

## 프록시(Proxy)
### 의도
다른 객체에 대한 접근을 제어하기 위한 대리자 또는 자리체움자 역할을 하는 객체를 둔다.

어떤 객체에 대한 접근을 제어하는 한 가지 이유는 실제로 그 객체를 사용할 수 있을 때까지
객체 생성과 초기화에 들어가는 비용 및 시간을 묻지 않겠다는 것이다.

### 활용성
- **Remote Proxy**는 서로 다른 주소 공간에 존재하는 객체를 가리키는 대표 객체로, 로컬 환경에 위치한다.
- **Virtual Proxy**는 요청이 있을 때만 필요한 고비용 객체를 생성한다.
- **Protection Proxy**는 원래 객체에 대한 실제 접근을 제어한다.
  - 이는 객체 별로 접근 제어 권한이 다를 때 유용하게 사용할 수 있다.
- **Smart Reference**는 원시 포인터의 대채용 객체로, 실제 객체에 접근이 일어날 때 추가적인 행동을 수행한다.

### 구조 및 구현
`Proxy`클래스는 자신이 받을 요청을 `RealSubject`객체에 전달한다.

```ts
interface Subject {
    request(): void
}

class RealSubject implements Subject {
    request() {}
}

class ProxySubject implements Subject {
    realSubject: Subject
    constructor(subject: Subject) {
        this.realSubject = subject
    }
    request() {
        this.realSubject.request()
    }
}
```

#### 사용자측 코드
```ts
class Main {
    subject: Subject
    constructor() {
        this.subject = new RealSubject()
        this.subject.request()
    }
}

// Proxy 사용
class Main {
    subject: Subject
    constructor() {
        const realSubject = new RealSubject()
        this.subject = new ProxySubject(realSubject)
        this.subject.request()
    }
}
```
