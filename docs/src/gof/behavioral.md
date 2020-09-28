---
sidebar: auto
title:  "GoF 디자인 패턴 | 행동패턴"
date:   2018-12-09
description: GoF 디자인 패턴 중 행동패턴을 정리합니다.
tags: [pattern]
category: 패턴
---
# GoF 디자인 패턴 | 행동패턴
## 이터레이터(Iterator)
내부 표현부를 노출하지 않고 어떤 객체 집합에 속한 원소들을 순차적으로 접근할 수 있는 방법을 제공하는 패턴이다.

* 객체 내부 표현 방식을 모르고도 집합 객체의 각 원소들에 접근하고 싶을 때
* 집합 객체를 순회하는 다양한 방법을 지원하고 싶을 때
* 서로 다른 집합 객체 구조에 대해서도 동일한 방법으로 순회하고 싶을 때

```js
class Iterator {
  constructor (items) {
    this.index = 0
    this.items = items
  }
  first () {
    this.index = 0
  }
  last () {
    this.index = this.count() - 1
  }
  count () {
    return this.items.length
  }
  next () {
    if (!this.hasNext()) {
      throw new Error(`Index is last item. use hasNext method.`)
    }
    this.index++
  }
  previous () {
    if (!this.hasPrevious()) {
      throw new Error(`Index is first item. use hasPrevious method.`)
    }
    this.index--
  }
  hasNext () {
    return this.index < this.count()
  }
  hasPrevious () {
    return this.index >= 0
  }
  currentItem () {
    return this.items[this.index]
  }
  forEach (callback=()=>{}) {
    for (this.first(); this.hasNext(); this.next()) {
      callback(this.currentItem())
    }
  }
  forEachRight (callback=()=>{}) {
    for (this.last(); this.hasPrevious(); this.previous()) {
      callback(this.currentItem())
    }
  }
}
```
```js
const items = ['one', 2, 'circle', true, "Applepie"];
const iter = new Iterator(items);

iter.forEach(item => console.log(item))
iter.forEachRight(item => console.log(item))
```
```
one
2
circle
true
Applepie
Applepie
true
circle
2
one
```

## 옵져버(Observer)
객체 사이에 일 대 다의 의존 관계를 정의해 두어, 어떤 객체의 상태가 변할 때 그 객체의 의존성을 가진 다른 객체들이 그 변화를 통지받고 자동으로 갱신될 수 있게 만드는 패턴이다. (Publish-Subscribe 관계)


- 어떤 추상 개념이 두 가지 양상을 갖고 하나가 다른 하나에 종속적일 때. 각 양상을 별도의 객체로 캡슐화하여 이들 각각을 재사용할 수 있습니다.
- 한 객체에 가해진 변경으로 다른 객체를 변경해야 하고, 프로그래머들은 얼마나 많은 객체들이 변경되어야 하는지 몰라도 될 때
- 어떤 객체가 다른 객체에 자신의 변화를 통보할 수 있는 데, 그 변화에 관심있어 하는 객체들이 누구인지에 대한 가정 없이도 그러한 통보가 될 때

```js
class Click {
  constructor () {
    this.handlers = new Set();  // observers
  }
  subscribe (fn) {
    this.handlers.add(fn);
  }
  unsubscribe (fn) {
    this.handlers.delete(fn)
  }
  fire (str) {
    this.handlers.forEach(fn => fn(str))
  }
}
```
```js
const clickHandler = (item) => console.log(`1 fired: ${item}`)
const clickHandler2 = (item) => console.log(`2 fired: ${item}`)

const click = new Click();

click.subscribe(clickHandler);
click.subscribe(clickHandler2);
click.fire('event #1');

click.unsubscribe(clickHandler);
click.fire('event #2');
```
```
1 fired: event #1
2 fired: event #1
2 fired: event #2
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

## 책임연쇄(Chain of responsibility)
메시지를 보내는 객체와 이를 받아 처리하는 객체들 간의 결합도를 없애기 위한 패턴입니다. 하나의 요청에 대한 처리가 반드시 
한 객체에서만 되지 않고, 여러 객체에서 그 처리 기회를 주려는 것입니다.


- 하나 이상의 객체가 요청을 처리해야 하고, 그 요청 처리자 중 어떤 것이 선행자 인지 모를 때. 처리자가 자동으로 확정되어야 합니다.
- 메시지를 받을 객체를 명시하지 않은 채 여러 객체 중 하나에게 처리를 요청하고 싶을 때
- 요청을 처리할 수 있는 객체 집합이 동적으로 정의되어야 할 때

### Switch
```js
switch (true) {
  case network() === 'online': break
  case network() === 'wifi': break
  case network() === 'offline': break
  case localCache(): break
  default: break
}
```

### Class
```js
class Calculator {
  constructor () {
    this.num = 0
  }
  print () {
    console.log(this.num)
    return this
  }
  add (n) {
    this.num += n
    return this
  }
  sub (n) {
    this.num -= n
    return this
  }
}
```
```js
const calculator = new Calculator()

calculator.add(100).sub(50).sub(20).print() //30
```

## 커멘드(Command)
요청 자체를 캡슐화하는 것입니다. 이를 통해 요청이 서로 다른 사용자를 매개변수로 만들고, 요청을 대기시키거나 로깅하며, 되돌릴 수 있는 연산을 지원합니다.(Action, Transaction)


- 수행할 동작을 객체로 매개변수화하고자 할 때
  - 절차지향 프로그램에서는 이를 콜백 함수, 즉 어딘가 등록되었다가 나중에 호출되는 함수를 사용해서 이러한 매개변수화를 표현할 수 있습니다. 명령 패턴은 콜백을 객체지향 방식으로 나타낸 것입니다.
- 서로 다른 시간에 요청을 명시하고, 저장하며, 실행하고 싶을 때
  - Command 객체는 원래의 요청과 다른 생명주기가 있습니다. 요청을 받아 처리하는 객체가 주소 지정 방식과는 독립적으로 표현될 수 있다면, Command 객체를 다른 프로세스에게 넘겨주고 거기서 해당 처리를 진행하게 할 수 있습니다.
- 실행 취소 기능을 지원하고 싶을 때
  - Command의 Execute() 연산은 상태를 저장할 수 있는 데, 이를 이용해서 지금까지 얻은 결과를 바꿀 수 있습니다. 이를 위한 Unexecute() 연산을 Command 클래스의 인터페이스에 추가합니다.
  실행된 명령어를 모두 기록해 두었다가 이 리스트를 역으로 탐색해서 다시 Unexecute()를 수행하게 됩니다. Execute()와 Unexecute() 연산의 반복 사용을 통해 수행과 취소를 무한 반복할 수 있습니다.
- 기본적인 연산의 조합으로 만든 상위 수준 연산을 써서 시스템을 구조화하고 싶을 때
  - 정보 시스템의 일반적인 특성은 트랜잭션을 처리해야 한다는 것입니다. 트랜잭션은 일련의 과정을 통해 데이터를 변경하는 것인데, Command 패턴은 이런 트랜잭션의 모델링을 가능하게 합니다.
  Command 클래스는 일관된 인터페이스를 정의하는 데, 이로써 모든 트랜잭션이 동일한 방식으로 호출됩니다. 새로운 트랜잭션을 만들면 상속으로 Command 클래스를 확장하면 되므로 시스템 확장도 어렵지 않습니다.
  
```js
const add = (x, y) => x + y
const sub = (x, y) => x - y
const mul = (x, y) => x * y
const div = (x, y) => x / y

class Command {
  constructor (execute, undo, value, name) {
    this.execute = execute
    this.undo = undo
    this.value = value
    this.name = name
  }
}

class AddCommand {
  constructor (value) {
    return new Command(add, sub, value, 'Add')
  }
}

class SubCommand {
  constructor (value) {
    return new Command(sub, add, value, 'Sub')
  }
}

class MulCommand {
  constructor (value) {
    return new Command(mul, div, value, 'Mul')
  }
}

class DivCommand {
  constructor (value) {
    return new Command(div, mul, value, 'Div')
  }
}

class Calculator {
  constructor () {
    this.current = 0
    this.commands = []
  }
  action (command) {
    const name = command.name
    return `${name.charAt(0).toUpperCase()}${name.slice(1)}`
  }
  execute (command) {
    this.current = command.execute(this.current, command.value)
    this.commands.push(command)
    console.log(`${this.action(command)}: ${command.value}`)
  }
  undo () {
    const command = this.commands.pop()
    this.current = command.undo(this.current, command.value)
    console.log(`Undo ${this.action(command)}: ${command.value}`)
  }
  getCurrentValue () {
    return this.current
  }
}
```
```js
const calculator = new Calculator()

calculator.execute(new AddCommand(100))
calculator.execute(new SubCommand(24))
calculator.execute(new MulCommand(6))
calculator.execute(new DivCommand(2))

calculator.undo()
calculator.undo()

console.log(`Value: ${calculator.getCurrentValue()}`)
```
```
Add: 100
Sub: 24
Mul: 6
Div: 2
Undo Div: 2
Undo Mul: 6
Value: 76
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

## 미디에이터(Mediator)
한 집합에 속해있는 객체들의 상호작용을 캡슐화하는 객체를 정의하는 패턴이다. 객체들이 직접 서로를 참조하지 않도록 함으로써
객체들 사이의 소결합(loose coupling)을 촉진시키며, 개발자가 객체들의 상호작용을 독립적으로 다양화시킬 수 있게 만든다.


```js
class Participant {
  constructor (name) {
    this.name = name
    this.chatroom = null
  }
  send (message, to) {
    this.chatroom.send(message, this, to)
  }
  receive (message, from) {
    console.log(`${from.name} to ${this.name}: ${message}`)
  }
}

class Chatroom {
  constructor () {
    this.participants = new Set()
  }
  register (participant) {
    participant.chatroom = this
    this.participants.add(participant)
  }
  send (message, from, to) {
    if (to) {                      // single message
      to.receive(message, from)
    } else {                       // broadcast message
      this.participants.forEach((participant) => {
        if (participant !== from) {
          participant.receive(message, from)
        }
      })
    }
  }
}
```
```js
const yoko = new Participant('Yoko')
const john = new Participant('John')
const paul = new Participant('Paul')
const ringo = new Participant('Ringo')

const chatroom = new Chatroom()

chatroom.register(yoko)
chatroom.register(john)
chatroom.register(paul)
chatroom.register(ringo)

yoko.send('All you need is love.')
yoko.send('I love you John.')
john.send('Hey, no need to broadcast', yoko)
paul.send('Ha, I heard that!')
ringo.send('Paul, what do you think?', paul)
```

## 메멘토(Memento)
캡슐화를 위배하지 않는 채 어떤 객체의 내부 상태를 잡아내고 신체화시켜, 이후에 해당 객체가 그 상태로 다시 되돌아올 수 있도록 하는 패턴이다.


```js
class Person {
  constructor (name, street, city, state) {
    Object.assign(this, {name, street, city, state})
  }
  hydrate () {
    return JSON.stringify(this)
  }
  dehydrate (memento) {
    const {name, street, city, state} = JSON.parse(memento)
    Object.assign(this, {name, street, city, state})
  }
}

class CareTaker {
  constructor () {
    this.mementos = new Map()
  }
  add (key, memento) {
    this.mementos.set(key, memento)
  }
  get (key) {
    return this.mementos.get(key)
  }
}
```
```js
const mike = new Person('Mike Foley', '1112 Main', 'Dallas', 'TX')
const john = new Person('John Wang', '48th Street', 'San Jose', 'CA')
const caretaker = new CareTaker()

// save state
caretaker.add(1, mike.hydrate())
caretaker.add(2, john.hydrate())

// mess up their names
mike.name = 'King Kong'
john.name = 'Superman'

// restore original state
mike.dehydrate(caretaker.get(1))
john.dehydrate(caretaker.get(2))

console.log(mike.name) // Mike Foley
console.log(john.name) // John Wang
```

## 인터프리터(Interpreter)
주어진 언어에 대해, 그 언어의 문법을 위한 표현 수단을 정의하고, 이와 어울러 그 표현 수단을 사용하여 해당 언어로 작성된 문장을 해석하는 해석기를 정의하는 패턴이다.


```js
class Context {
  constructor(input) {
    this.input = input
    this.output = 0
  }
  startsWith(str) {
    return this.input.startsWith(str)
  }
}

class Expression {
  constructor(name, one, four, five, nine, multiplier) {
    this.name = name
    this.one = one
    this.four = four
    this.five = five
    this.nine = nine
    this.multiplier = multiplier
  }
  interpret(context) {
    if (context.input.length === 0) {
      return
    }
    if (context.startsWith(this.nine)) {
      this.calculate(9, this.nine);
    } else if (context.startsWith(this.four)) {
      this.calculate(4, this.four);
    } else if (context.startsWith(this.five)) {
      this.calculate(5, this.five);
    }
    while (context.startsWith(this.one)) {
      this.calculate(1, this.one);
    }
  }
  calculate(num, str) {
    context.output += (num * this.multiplier)
    context.input = context.input.substr(str.length)
  }
}
```
```js
const interpreter = [
  new Expression('thousand', 'M', ' ', ' ', ' ', 1000),
  new Expression('hundred', 'C', 'CD', 'D', 'CM', 100),
  new Expression('ten', 'X', 'XL', 'L', 'XC', 10),
  new Expression('one', 'I', 'IV', 'V', 'IX', 1)
];

const roman = 'MCMXXVIII'
const context = new Context(roman)
interpreter.forEach(leaf => leaf.interpret(context))

console.log(`${roman} = ${context.output}`)
// MCMXXVIII = 1928
```
