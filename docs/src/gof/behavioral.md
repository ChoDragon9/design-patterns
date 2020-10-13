---
sidebar: auto
title:  "GoF 디자인 패턴 | 행동패턴"
date:   2018-12-09
description: GoF 디자인 패턴 중 행동패턴을 정리합니다.
tags: [pattern]
category: 패턴
---
> 출처: 에릭 감마 , 리처드 헬름 , 랄프 존슨 , 존 블리시디스. 『Gof의 디자인 패턴』. 김정아(역). 프로텍미디어, 2015.

# GoF 디자인 패턴 | 행동패턴
- 어떤 처리의 책임을 어느 객체에 할당하는 것이 좋은지, 알고리즘은 어느 객체에 정의하는 것이 좋은지 다룹니다.
- 객체들 간의 교류 방법에 대하여 정의한다.
- **행동 클래스 패턴**은 클래스 사이에 행동 책임을 분산하기 위해서 상속을 사용합니다.
- **행동 객체 패턴**은 상속보다는 복합을 통해서 객체 사이에 행동처리의 책임을 분산합니다.

## 책임연쇄(Chain of responsibility)
### 의도
메세지를 보내는 객체와 이를 받아 처리하는 객체들 간의 결합도를 없애기 위한 패턴입니다.
하나의 요청에 대한 처리가 반드시 한 객체에서만 되지 않고, 여러 객체에게 그 처리 기회를 주려는 것입니다.

### 활용성
- 하나 이상의 객체가 요청을 처리해야 하고, 그 요청 처리자중 어떤 것이 선행자인지 모를 때, 처리자가 자동으로 확정되어야 합니다.
- 메시지를 받을 객체를 명시하지 않은 채 여러 객체 중 하나에게 처리를 요청하고 싶을 때
- 요청을 처리할 수 있는 객체 집합이 동적으로 정의되어야 할 때

### 구조 및 구현
```ts
interface Handler {
    next?: Handler
    handle(request: string): void
    setNext(next: Handler): void
}

abstract class BaseHandler implements Handler {
    next?: Handler
    abstract handle(request: string): void
    handleNext(request: string) {
       if (this.next) {
            this.next.handle(request)
        } 
    }
    setNext(next: Handler) {
        this.next = next
    }
}

class ConcreteHandler1 extends BaseHandler {
    handle(request: string) {
        if (request === 'First') {
            console.log('ConcreteHandler1!')
        } else {
            this.handleNext(request)
        }
    }
}

class ConcreteHandler2 extends BaseHandler {
    handle(request: string) {
        if (request === 'Second') {
            console.log('ConcreteHandler2!')
        } else {
            this.handleNext(request)
        }
    }
}

class ConcreteHandler3 extends BaseHandler {
    handle(request: string) {
        if (request === 'Third') {
            console.log('ConcreteHandler3!')
        } else {
            this.handleNext(request)
        }
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const handle1 = new ConcreteHandler1()
        const handle2 = new ConcreteHandler2()
        const handle3 = new ConcreteHandler3()

        handle1.setNext(handle2)
        handle2.setNext(handle3)

        handle1.handle('First') // ConcreteHandler1!
        handle1.handle('Second') // ConcreteHandler2!
        handle1.handle('Third') // ConcreteHandler3!
    }
}
```

#### 협력 방법
사용자는 처리를 요청하고, 이 처리 요청을 실제로 그 요청을 받을 책임이 있는 `ConcreteHandler`객체를 만날 때까지 정의된 연결고리를 따라서 계속 전달됩니다.

### 결과
1. 객체 간의 행동적 결합도가 적어집니다.
2. 객체에게 책임을 할당하는 데 유연성을 높일 수 있습니다.
3. 메시지 수신이 보장되지는 않습니다.
   - 어떤 객체가 이 처리에 대한 수신을 담당한다는 것을 명시하지 않으므로 요청이 처리된다는 보장이 없습니다.

## 명령(Command)
### 의도
요청 자체를 캡슐화하는 것입니다. 이를 통해 요청이 서로 다른 사용자를 매개변수로 만들고, 요청을 대기 시키거나 로깅하여, 되돌릴 수 있는 연산을 지원합니다.

### 활용성
- 수행할 동작을 객체로 매개변수화하고자 할 때
  - 절차지향 프로그래밍에서는 이를 콜백 함수, 즉 어딘가 등록되었다가 나중에 호출되는 함수를 사용해서 이러한 매개변수화를 표현할 수 있다.
  - 명령 패턴은 콜백을 객체지향 방식으로 나타낸 것이다.
- 서로 다른 시간에 요청을 명시하고, 저장하며, 실행하고 싶을 때.
- 실행 취소 기능을 지원하고 싶을 때.
  - Command의 execute() 연산은 상태를 저장할 수 있는 데, 이를 이용해서 지금까지 얻은 결과를 바꿀 수 있다.
  - 이를 위해 unexecute() 연산을 Command 클래스의 인터페이스에 추가한다.
- 시스템이 고장 났을 때 재적용이 가능하도록 변경 과정에 대한 로깅을 지원하고 싶을 때.
  - Command 인터페이스를 확장해서 load()와 store()연산을 정의하면 상태의 변화를 지속적(persistant) 저장소에 저장해 둘 수 있다.
- 기본적인 연산의 조합으로 만든 상위 수준 연산을 써서 시스템을 구조화하고 싶을 때.
  - 일련의 과정을 통해 데이터를 변경하는 트랜젝션(transaction)의 모델링을 가능하게 한다.

### 구조 및 구현
```ts
interface Command {
    execute(): void
}

class ConcreteCommand implements Command {
    private receiver: Receiver
    constructor(receiver: Receiver) {
        this.receiver = receiver
    }
    execute(): void {
        console.log('ConcreteCommand Execute')
        this.receiver.action()
    }
}

class Receiver {
    action() {
        console.log('Receiver Action')
    }
}

class Invoker {
    private commands: Command[] = []

    storeCommand(command: Command) {
        this.commands.push(command)
        command.execute()
    }
}
```
#### 사용자측 코드
```ts
class Main {
    constructor() {
        const receiver = new Receiver()
        const command = new ConcreteCommand(receiver)
        const invoker = new Invoker()

        invoker.storeCommand(command)
        // ConcreteCommand Execute
        // Receiver Action
    }
}
```

## 해석자(Interpreter)
### 의도
간단한 언어의 문법을 정의하는 바업ㅂ과 그 언어로 문장을 구정하는 방법, 이들 문장을 해석하는 방법을 설명합니다.

### 활용성
- 정의할 언어의 문법이 간단할 때 적합하다.
- 문법이 복잡하다면 문법을 정의하는 클래스 계통이 복잡해지고 관리하기 힘들기 때문에 이럴 때는 파서 생성기가 적합하다.

### 해석자 정의 방법
1. 문법정의
   - expression: 시작 기호
   - literal: 터미널 기호
    ```
    expression ::= literal | alternation | sequence | repetition | '(' expression ')'
    alternation ::= expression '|' expression
    sequence ::= expression '&' expression
    repetition ::= expression '*'
    literal ::= 'a' | 'b' | 'c' ... {'a' | 'b' | 'c'...}*
    ```
2. 문법 오른편을 클래스로 정의
3. interpret(context) 연산을 구현하여 Context를 근거로 입력문자열에서 다음번 일치하는 문자열을 발견한다.
   - Context는 입력 문자열이 무엇이고 지금까지 어떻게 일치되는 내용을 찾아왔는지에 대한 상태 정보를 포함한다.
4. 결과적으로 interpret(context)에서 클래스들을 사용함으로 트리 형태로 만들어진다.

### 구조 및 구현
> 작성중

```ts
interface AbstractExpression {
    interpret(context: Context): void
}

class TerminalExpression implements AbstractExpression {
    interpret(context: Context) {}
}

class NonTerminalExpression implements AbstractExpression {
    interpret(context: Context) {}
}

class Context {}
```

## 반복자(Iterator)
### 의도
내부 표현부를 노출하지 않고 어떤 집합 객체에 속한 원소들을 순차적으로 접근할 수 있는 방법을 제공합니다.

### 활용성
- 객체 내부 표현 방식을 모르고도 집합 객체의 각 원소들에 접근하고 싶을 때
- 집합 객체를 순회하는 다양한 방법을 지원하고 싶을 때
  - 트리를 순회할 때 중위 순회 방식이나 전위 순회 방식등과 같은 순회 알고리즘을 바꿀 수 있다.
- 서로 다른 집합 객체 구조에 대해서도 동일한 방법으로 순회하고 싶을 때
 
### 구조 및 구현
> Cursor: Aggregate 클래스에서 순회 알고리즘을 정의하고, Iterator에는 순회의 상태만 저장할 수 있는 데,
> 이렇게 구현된 Iterator를 Cursor라고 가리킨다.

```ts
interface Aggregate {
    createIterator(): IteratorInterface
}

interface IteratorInterface {
    first(): void
    next(): void
    isDone(): boolean
    currentItem(): unknown
}

class ConcreteAggregate implements Aggregate {
    createIterator() {
        return new ConcreteInterator()
    }
}

class ConcreteInterator implements IteratorInterface {
    private state: number[]
    private index: number = 0
    constructor () {
        this.state = [0, 1, 2]
    }
    first() {
        this.index = 0
    }
    next() {
        this.index++
    }
    isDone() {
        return this.index > this.state.length
    }
    currentItem() {
        return this.state[this.index]
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const aggregate = new ConcreteAggregate()
        const iterator = aggregate.createIterator()

        while (!iterator.isDone()) {
            const item = iterator.currentItem()
            console.log(item)
            iterator.next()
        }
        // 0 1 2
    }
}
```

## 중재자(Mediator)
### 의도
한 집합에 속해 있는 객체의 상호작용을 캡슐화하는 객체를 정의합니다.
객체들이 서로를 직접 참조하지 않도록 하여 객체들간의 소결합(Loose Coupling)을 촉진시키며,
객체들 간의 상호작용을 독립적으로 다양화 시킬 수 있게 합니다.

### 활용성
- 한 객체가 다른 객체를 너무 많이 참고하고, 너무 많은 의사소통을 수행해서 그 객체를 재사용하기 힘들 때
- 여러 클래스에 분산된 행동들이 상속없이 상황에 맞게 수정되어야 할 때
- 여러 객체가 잘 정의된 형태이기는 하지만 복잡한 상호작용을 가질 때
- 객체간의 의존성이 구조화되지 않으며, 잘 이해하기 어려울 때

### 구조 및 구현
```ts
interface Mediator {
    colleague1: Colleague
    colleague2: Colleague
}

interface Colleague {
    mediator: Mediator
}

class ConcreteMediator implements Mediator {
    colleague1: ConcreteColleague1
    colleague2: ConcreteColleague2
    constructor() {
        this.colleague1 = new ConcreteColleague1(this)
        this.colleague2 = new ConcreteColleague2(this)
    }

    request() {
        const str = this.colleague1.request()
        const result = this.colleague2.request(str)
        
        console.log(result)
    }
}

class ConcreteColleague1 implements Colleague {
    mediator: Mediator
    constructor(mediator: Mediator) {
        this.mediator = mediator
    }
    request() {
        return 'Apple'
    }
}

class ConcreteColleague2 implements Colleague {
    mediator: Mediator
    constructor(mediator: Mediator) {
        this.mediator = mediator
    }
    request(str: string) {
        return str.toUpperCase()
    }
}
```

#### 협력방법
- Colleague는 Mediator에서 요청을 송수신합니다.
- Mediator는 필요한 Colleague 사이에 요청을 전달할 의무가 있습니다.

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const mediator = new ConcreteMediator()
        mediator.request() // APPLE
    }
}
```

## 메멘토(Memento)
### 의도
캡슐화를 위해하지 않은 체 어떤 객체의 내부 상태를 잡아내고 실체화 시켜둠으로써,
이후 해당 객체가 그 상태로 되돌아 올 수 있도록 합니다.

### 활용성
- 어떤 객체의 상태에 대한 스냅샷을 저장한 후 나중에 이 상태로 복구해야 할 때
- 상태를 얻는 데 필요한 직접적인 인터페이스를 두면 그 객체의 구현 세부사항에 드러날 수 밖에 없고,
이것으로 객체의 캡슐화가 깨질때

### 구조 및 구현
```ts
class Memento {
    private state: unknown
    constructor(state: unknown) {
        this.setState(state)
    }
    getState() {
        return this.state
    }
    setState(state: unknown) {
        this.state = state
    }
}

class Originator {
    private state: unknown
    constructor() {
        this.state = ''
    }
    createMemento() {
        return new Memento(this.state)
    }
    setMememto(memento: Memento) {
        this.state = memento.getState()
    }
}

class Caretaker {
    constructor(originator: Originator) {
        const memento = originator.createMemento()
        originator.setMememto(memento)
    }
}
```

## 감시자(Observer)
### 의도
객체 사이에 일대 다의 의존 관계를 정의해두어,
어떤 객체의 상태가 변할 때 그 객체에 의존성을 가진 다른 객체들이 그 변화를 통지 받고 자동으로 갱신될 수 있게 만듭니다.

### 활용성
- 어떤 추상 개념이 두 가지 양상을 갖고 하나가 다른 하나에 종속적일 때. 각 양상을 별도의 객체로 캡슐화하여 이들 각각을 재사용할 수 있습니다.
- 한 객체에 가해진 변경으로 다른 객체를 변경해야 하고, 프로그래머들은 얼마나 많은 객체들이 변경되어야 하는지 몰라도 될 때
- 어떤 객체가 다른 객체에 자신의 변화를 통보할 수 있는 데, 그 변화에 관심있어 하는 객체들이 누구인지에 대한 가정 없이도 그러한 통보가 될 때

### 구조 및 구현
```ts
interface Observer {
    update(): void
}

abstract class Subject {
    private observers: Set<Observer> = new Set()
    
    attach(observer: Observer) {
        this.observers.add(observer)
    }
    
    detach(observer: Observer) {
        this.observers.delete(observer)
    }
    
    notify() {
        this.observers.forEach(observer => {
            observer.update()
        })
    }
}

class ConcreteSubject extends Subject {
    private subjectState = ''
    getState() {
        return this.subjectState
    }
    setState(state: string) {
        this.subjectState = state
    }
}

class ConcreteObserver implements Observer {
    private subject: ConcreteSubject
    private observerState = ''
    
    constructor(subject: ConcreteSubject) {
        this.subject = subject
        this.subject.attach(this)
    }

    update() {
        console.log('updated')
        this.observerState = this.subject.getState()
    }
}
```

#### 사용자측 코드
```ts
class Main {
    constructor() {
        const subject = new ConcreteSubject()
        const observer1 = new ConcreteObserver(subject)
        const observer2 = new ConcreteObserver(subject)

        subject.notify() // updated updated
        subject.detach(observer1)
        subject.notify() // updated
    }
}
```

## 상태(State)
객체의 내부 상태에 따라 스스로 행동을 변경할 수 있게끔 허가하는 패턴으로, 이렇게 하면 객체를 마치 자신의 클래스를 바꾸는 것 처럼 보인다.

하나의 객체가 여러가지의 상태가 존재할 때 보통 if/switch 문으로 분기 후 결과를 처리한다.
신규 상태가 존재할 때 마다 if/switch 코드를 수정해야 함으로 객체의 상태를 클래스화해서 그것을 참조하게 한다.


```js
class Red {
  constructor () {}
  go(context) {
    console.log('Red --> for 1 minute')
    context.setState(new Green())
  }
}

class Green {
  constructor () {}
  go(context) {
    console.log('Green --> for 1 minute')
    context.setState(new Yellow())
  }
}

class Yellow {
  constructor () {}
  go(context) {
    console.log('Yellow --> for 10 seconds')
    context.setState(new Red())
  }
}

class TrafficLight {
  constructor () {
    this.currentState = new Red()
  }
  setState (state) {
    this.currentState = state
  }
  go () {
    this.currentState.go(this)
  }
}
```
```js
const trafficLight = new TrafficLight()
trafficLight.go() // Red
trafficLight.go() // Green
trafficLight.go() // Yellow
```

## 전략(Strategy)
동일 계열의 알고리즘군을 정의하고, 각각의 알고리즘을 캡슐화하여, 이들을 상호교환이 가능하도록 만드는 패턴이다.
알고리즘을 사용하는 사용자와 상관없이 독립적으로 알고리즘을 다양하게 변경할 수 있게 한다.


```js
class Shipping {
  constructor () {
    this.company = ''
  }
  setStrategy (company) {
    this.company = company
  }
  calculate (baggage) {
    return this.company.calculate(baggage)
  }
}

class UPS {
  constructor () {}
  calculate (baggage) {
    // calculations...
    return '$45.95'
  }
}

class USPS {
  constructor () {}
  calculate (baggage) {
    // calculations...
    return '$39.40'
  }
}

class Fedex {
  constructor () {}
  calculate (baggage) {
    // calculations...
    return '$43.20'
  }
}
```
```js
const baggage = { from: '76712', to: '10012', weigth: 'lkg' }

// the 3 strategies
const ups = new UPS()
const usps = new USPS()
const fedex = new Fedex()

const shipping = new Shipping()

shipping.setStrategy(ups)
console.log(`UPS Strategy: ${shipping.calculate(baggage)}`)

shipping.setStrategy(usps)
console.log(`USPS Strategy: ${shipping.calculate(baggage)}`)

shipping.setStrategy(fedex)
console.log(`Fedex Strategy: ${shipping.calculate(baggage)}`)
```


## 탬플릿 메소드(Template Method)
객체의 연산에는 알고리즘의 뼈대만을 정의하고 각 단계에서 수행할 구체적 처리는 서브클래스쪽으로 미루는 패턴이다. 알고리즘의 구조 자체는 그대로 놔둔 채 알고리즘 각 단계의 처리를 서브클래스에서 재정의할 수 있게 한다.


```js
class AbstractDataStore {
  process () {
    this.connect()
    this.select()
    this.disconnect()
    return true
  } 
}

class MySQL extends AbstractDataStore {
  connect () {
    console.log('MySQL: connect step')
  }
  select () {
    console.log('MySQL: select step')
  }
  disconnect () {
    console.log('MySQL: disconnect step')
  }
  process () {
    super.process()
  }
}
```
```js
const mySql = new MySQL()

mySql.process()
```

## 방문자(Visitor)
객체 구조를 이루는 원소에 대해 수행할 연산을 표현하는 패턴으로, 연산을 적용할 원소의 클래스를 변경하지 않고도 새로운 연산을 정의할 수 있게 한다.


```js
class Employee {
  constructor (name, salary, vacation) {
    this.name = name
    this.salary = salary
    this.vacation = vacation
  }
  accept (visitor) {
    visitor.visit(this)
  }
  getName () {
    return this.name
  }
  getSalary () {
    return this.salary
  }
  setSalary (sal) {
    this.salary = sal
  }
  getVacation () {
    return this.vacation
  }
  setVacation (vac) {
    this.vacation = vac
  }
}

class ExtraSalary {
  visit (emp) {
    emp.setSalary(emp.getSalary() * 1.1)
  }
}

class ExtraVacation {
  visit (emp) {
    emp.setVacation(emp.getVacation() + 2)
  }
}
```
```js
const employees = [
  new Employee('John', 10000, 10),
  new Employee('Mary', 20000, 21),
  new Employee('Boss', 250000, 51)
]

const visitorSalary = new ExtraSalary()
const visitorVacation = new ExtraVacation()

employees.forEach((emp) => {
  emp.accept(visitorSalary)
  emp.accept(visitorVacation)
  console.log(`${emp.getName()}: $${emp.getSalary()} and ${emp.getVacation()} vacation days`)
})
```
```
John: $11000 and 12 vacation days
Mary: $22000 and 23 vacation days
Boss: $275000 and 53 vacation days
```
