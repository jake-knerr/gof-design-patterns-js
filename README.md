<!-- omit in toc -->

# GoF Design Patterns

This document summarizes the design patterns contained within the seminal work _Design Patterns: Elements of Reusable Object-Oriented Software_ using JavaScript. The authors are often referred to as the Gang of Four (GoF).

Feel free to create an issue or open a pull request if you have something to add or believe something is incorrect.

Also, please star this repository if you find it useful. Thanks!

<!-- omit in toc -->

## Copyright

© Jake Knerr at Blotli.

<!-- omit in toc -->

## Table of Contents

- [GoF Design Patterns](#gof-design-patterns)
  - [Copyright](#copyright)
  - [Table of Contents](#table-of-contents)
  - [Creational Patterns](#creational-patterns)
    - [Factory Method](#factory-method)
    - [Abstract Factory](#abstract-factory)
    - [Builder](#builder)
    - [Prototype](#prototype)
    - [Singleton](#singleton)
  - [Structural Patterns](#structural-patterns)
    - [Adapter](#adapter)
    - [Bridge](#bridge)
    - [Composite](#composite)
    - [Decorator](#decorator)
    - [Facade](#facade)
    - [Flyweight](#flyweight)
    - [Proxy](#proxy)
  - [Behavioral Patterns](#behavioral-patterns)
    - [Chain of Responsibility](#chain-of-responsibility)
    - [Command](#command)
    - [Interpreter](#interpreter)
    - [Iterator](#iterator)
    - [Mediator](#mediator)
    - [Memento](#memento)
    - [Observer](#observer)
    - [State](#state)
    - [Strategy](#strategy)
    - [Template Method](#template-method)
    - [Visitor](#visitor)

---

## Creational Patterns

The following patterns relate to creating objects.

**[⬆ Table of Contents](#table-of-contents)**

---

### Factory Method

The factory method pattern is an factory object (creator) that has a single method that creates and returns objects (products).

- _Creator_ — the factory.
  - factoryMethod(): Product — creates and returns a product.
- _Product_ — objects created and returned by the creator.

_Example:_

```javascript
// Product constructor
class BigWidget {}

// Creator constructor
class WidgetFactory {
  // factoryMethod()
  build() {
    return new BigWidget();
  }
}

// client code
const factory = new WidgetFactory();

const bigWidget = factory.build();
```

_Example - using arguments to specify what type of product to create:_

```javascript
// Product constructor
class BigWidget {}

// Product constructor
class SmallWidget {}

// Creator constructor
class WidgetFactory {
  // factoryMethod()
  build(type) {
    if (type === "big") return new BigWidget();
    if (type === "small") return new SmallWidget();
  }
}

// client code
const factory = new WidgetFactory();

const bigWidget = factory.build("big");
const smallWidget = factory.build("small");
```

#### Why Use This Pattern

- Client code does not need to know how a specific product is created. This results in looser coupling between products and clients.
- Products can be requested without necessarily creating them. Perhaps they are recycled.

#### Consequences

- This pattern can create unjustified complexity.
- Factories must be kept simple to avoid violating the single-responsibility principle.

**[⬆ Table of Contents](#table-of-contents)**

---

### Abstract Factory

The abstract factory pattern is an factory object (factory) that constructs a family of related objects (products) using multiple factory methods (see above). Each specific product type is constructed by a single factory method.

- _Factory_ — the factory.
  - factoryMethodA(): ProductA — creates and returns a product of type ProductA.
  - factoryMethodB(): ProductB — creates and returns a product of type ProductB.
  - factoryMethodC(): ProductC — and so on.
- _Product_ — family of related objects created and returned by the factory.

_Example:_

```javascript
// Product A constructor
class BigWidget {}

// Product B constructor
class SmallWidget {}

// Factory constructor
class WidgetFactory {
  // factoryMethod()
  createBiggerWidget() {
    return new BigWidget();
  }

  // factoryMethod()
  createSmallerWidget() {
    return new SmallWidget();
  }
}

// client code
const factory = new WidgetFactory();

const smallWidget = factory.createSmallerWidget();
const bigWidget = factory.createBiggerWidget();
```

_Example - using prototypes:_

```javascript
// Factory constructor
function WidgetFactory() {
  // Product A prototype
  const bigWidgetProto = {
    big: { propA: 1, propB: 2 },
  };

  // Product B prototype
  const smallWidgetProto = {
    small: { propC: 1, propD: 2 },
  };

  // factoryMethod()
  this.createBiggerWidget = function () {
    return Object.create(bigWidgetProto);
  };

  // factoryMethod()
  this.createSmallerWidget = function () {
    return Object.create(smallWidgetProto);
  };
}

// client code
const factory = new WidgetFactory();

const smallWidget = factory.createSmallerWidget();
const bigWidget = factory.createBiggerWidget();
```

#### Why Use This Pattern

- The pattern provides clarity concerning which products are related. It also promotes consistency among products.
- It is straightforward to swap in a different factory because you only need to change the factory once — where it is created/instantiated.

**[⬆ Table of Contents](#table-of-contents)**

---

### Builder

The builder pattern is a factory (builder and director) that creates objects (products) that require multiple construction steps. The multiple steps are the primary difference between this pattern and the other creational patterns.

The construction process is split between two objects: the builder and the director. Client code only calls methods of the director, and the director calls the builder's multistep construction methods. Note, the GoF pattern has client code call methods on both the director and the builder, but I don't see the point in the additional complexity. Therefore, I am writing this pattern in such a way that client code only calls methods on the director. I don't think anything important is lost because of this change.

- _Builder_ — assembles products. Builder methods are not directly called by client code.
  - step1() — the first step to create a product.
  - step2() — the second step to create a product.
  - step3() — the third step to create a product.
  - step4() — and so on.
- _Director_ — creates products using the multistep interface of a particular builder.
  - construct(builder: Builder): Product — retrieve a product from the passed builder.
- _Product_ — the object created by a particular builder and returned by the director.

_Example:_

```javascript
// Builder constructor
class DrillBuilder {
  step1() {
    this.drill = new Drill();
  }

  step2() {
    this.drill.addParts();
  }

  step3() {
    this.drill.polish();
  }

  // implementation detail; gets products back to the director
  getItem() {
    return this.drill;
  }
}

// Director constructor
class MachineShopDirector {
  construct(builder) {
    builder.step1();
    builder.step2();
    builder.step3();

    return builder.getItem();
  }
}

// Product constructor
class Drill {
  addParts() {
    this.bits = 4;
  }

  polish() {}

  spin() {}
}

// client code
const shop = new MachineShopDirector();
const drillBuilder = new DrillBuilder();

const drill = shop.construct(drillBuilder);
drill.spin();
```

#### Example Notes

- In the example above, `DrillBuilder#getItem()` is an implementation detail that returns freshly created products to the director. Getting products back to the director could be accomplished in other ways.

#### Why Use This Pattern

- The builder pattern can provide consistency for objects with complex multistep construction processes.

#### Consequences

- This pattern results in very tight coupling between the director and builders.
- It creates additional complexity by breaking a factory into multiple objects.

**[⬆ Table of Contents](#table-of-contents)**

---

### Prototype

The prototype pattern is a technique to create objects by cloning an existent object (prototype). In other words, this pattern uses an object as a breeder for creating other objects.

Note, JavaScript handles prototypes differently than this pattern. Instead of cloning prototypes to create new objects, objects share prototypes via the prototype chain and delegation.

- _Prototype_ — the template object.
  - clone(): Prototype — clones itself and returns the clone (shallow clone).

_Example - GoF implementation of the prototype pattern:_

```javascript
// Prototype
const carPrototype = {
  drive() {},

  clone() {
    // create a shallow copy
    const cloned = {};

    cloned.drive = this.drive;

    return cloned;
  },
};

// client code
const car = carPrototype.clone();

car.drive();
```

_Example - using class to create objects that share a prototype:_

```javascript
// Prototype
class Car {
  drive() {}
}

// client code
const car = new Car();
car.drive();
```

_Example - using Object.create() to create objects that share a prototype:_

```javascript
// Prototype
class Car {
  drive() {
    console.log("driving...");
  }
}

const corvette = Object.create(Car.prototype, {
  driveFast: {
    value: () => console.log("driving fast!"),
  },
});

// client code
corvette.drive();
corvette.driveFast();
```

_Example - prototypal inheritance using the extends keyword:_

```javascript
class Car {
  drive() {
    console.log("driving...");
  }
}

class Corvette extends Car {
  driveFast() {
    console.log("driving fast!");
  }
}

// client code
const corvette = new Corvette();

corvette.drive();
corvette.driveFast();
```

#### Why Use This Pattern

- Prototypes are built into JavaScript.

#### Consequences

- Mutating prototypes at runtime should be done with extreme caution because such mutations affect all objects that share the prototype.

**[⬆ Table of Contents](#table-of-contents)**

---

### Singleton

The singleton pattern involves limiting the number of instances of a particular class (singleton) to just one. The singleton must guard that it is only instantiated once and that this instance can only be accessed via a static getInstance() method. Client code cannot be allowed to create new instances.

- _Singleton_ — class that creates a single instance.
  - static getInstance(): Singleton — returns the single instance for client use.

_Example:_

```javascript
let instanceCreated;
let instance;

// Singleton
export class Singleton {
  static getInstance() {
    if (!instance) {
      instanceCreated = true;
      instance = new Singleton();
    }

    return instance;
  }

  // check that a singleton instance is only created once and created by
  // getInstance(); not by client code
  constructor() {
    if (instance || !instanceCreated)
      throw new Error("Singleton cannot be created this way.");
  }

  behavior() {}

  moreBehavior() {}
}

// client code
import Singleton from "singleton/Singleton.js";

const singleton = Singleton.getInstance();
singleton.behavior();

// attempts to create an instance explicitly will throw an error
const anotherSingleton = new Singleton(); // Error
```

#### Example Notes

- The example above assumes that a client module will import the exported binding `Singleton`. Thus, `instanceCreated` and `instance` are private to the `Singleton` module and not accessible to clients.

#### Why Use This Pattern

- For languages that require a class to create objects, a singleton can be used instead of global variables to limit namespace pollution and the risk of global name collisions.

#### Consequences

- Since Javascript allows for the direct creation of objects, sharing a frozen object literal is much easier than the class-based singleton pattern.

**[⬆ Table of Contents](#table-of-contents)**

---

## Structural Patterns

The following patterns relate to how objects are combined to form more complex structures.

**[⬆ Table of Contents](#table-of-contents)**

---

### Adapter

The adapter pattern involves an object (adapter) that adapts the incompatible interface of another object (adaptee), thereby making the adaptee available for client use. In other words, a client makes a request to the adapter, which translates the request so that the adaptee can understand it.

Typically, this pattern is used after a program is designed and implemented. Thus, this pattern typically accommodates unplanned changes. Also, it may be used in defensive programming to shield client code from the changing interfaces of third-party dependencies.

- _Adaptee_ — the object being adapted; the interface that is incompatible with what the client expects.
  - specificRequest() — a method that the adapter adapts.
- _Adapter_ — adapts the adaptee; presents an interface that client code expects.
  - request() — a method that client code expects.

_Example:_

```javascript
// neither adaptee nor adapter; an old interface that the client code expects
class OldCar {
  startByHandCranking() {}

  pumpTheBrakes() {}
}

// Adaptee constructor
class NewCar {
  startWithKey() {}

  pressAntiLockBrakes() {}
}

// Adapter constructor
class CarAdapter {
  constructor() {
    const car = new NewCar();

    // add the old interface to the returned object
    this.startByHandCranking = () => {
      return car.startWithKey();
    };

    this.pumpTheBrakes = () => {
      return car.pressAntiLockBrakes();
    };
  }
}

// client code
const modernCarAdapter = new CarAdapter();

modernCarAdapter.startByHandCranking();
modernCarAdapter.pumpTheBrakes();
```

#### Example Notes

- The modern car can still be operated by hand cranking and pumping the brakes even though modern cars have electric starter motors and anti-lock brakes.

#### Why Use This Pattern

- A dependency's interface has changed, and client code cannot be updated to use the new interface.
- The program needs to swap in new implementations.

#### Consequences

- This pattern can get verbose when large numbers of methods need to be adapted. It may be time to refactor in this case.
- It looks like runtime monkey patching, but it isn't because the adapter is not dynamically replacing properties.

**[⬆ Table of Contents](#table-of-contents)**

---

### Bridge

The bridge pattern is similar to the adapter pattern, but instead of hard-coding the adaptee, the adaptee (implementor) is passed as a dependency to the adapter (abstraction). This decouples the abstraction from the implementor so you can change them independently. This pattern is sometimes called the double adapter pattern.

Unlike the adapter pattern, which is typically used to accommodate unplanned changes down the road, the bridge pattern is typically a part of the design phase. The classic example is support for different drivers, in which the implementor varies with each type of vendor (MySQL, MongoDB, etc.).

- _Abstraction_ — presents the interface that the client code expects.
  - setImpl(imp: Implementor) — pass in an implementor as the implementation of the interface.
  - operation() — a method that is implemented by the implementor.
- _Implementor_ — defines an implementation of the abstraction's interface.
  - operationImpl() — implements operation() for the abstraction.

_Example:_

```javascript
// Abstraction constructor
class Database {
  setImpl(imp) {
    this.imp = imp;
  }

  read(id) {
    return this.imp.select(id);
  }

  write(id, data) {
    return this.imp.insert(id, data);
  }
}

// Implementor constructor
class MySQL {
  select(id) {}

  insert(id, data) {}
}

// Implementor constructor
class MongoDB {
  select(id) {}

  insert(id, data) {}
}

// client code
const mysql = new MySQL();
const database = new Database();

// choosing the MySQL implementor
database.setImpl(mysql);
database.read(10);
database.write(10, "I am data.");
```

#### Why Use This Pattern

- You can extend the abstraction and implementor independently.

#### Consequences

- The greater complexity may not be worth it if simple and shallow object hierarchies would work.
- The coupling between the abstraction and the implementor creates the need to manage two interfaces. In the example above, since I am already programming MySQL and MongoDB to an interface, it may not be worth it to push them to a new interface, the abstraction's interface.

**[⬆ Table of Contents](#table-of-contents)**

---

### Composite

The composite pattern facilitates creating nested data structures — composite objects (composite) parent other component objects (leaf or composite). A tree data structure is a classic use case.

- _Component_ — defines the shared interface and default behavior used by composites and leafs.
  - operation() — example of default behavior.
  - add(cmp: Component) — not implemented.
  - remove(cmp: Component) — not implemented.
  - getChild(index: int): Component — not implemented.
  - parent(): Component — (optional) retrieve the parent component.
- _Composite_ — implements component, including the child-related methods because composites can parent child-components.
  - add(cmp: Component) — add a child component.
  - remove(cmp: Component) — remove a child component.
  - getChild(index: int): Component — retrieve a child component by index.
- _Leaf_ — implements part of component but does not implement child-related methods because leafs do not parent child-components.
  - leafOperation() — example of behavior only used by a leaf.

_Example:_

```javascript
// Component constructor
class Node {
  constructor(name, parent) {
    this.name = name;
    this.parent = parent;
  }

  // parent()
  parent() {
    return this.parent;
  }

  // operation()
  rename(name) {
    this.name = name;
  }

  // not implemented
  add(node) {}

  // not implemented
  remove(node) {}

  // not implemented
  getChild(node) {}
}

// Leaf constructor
class File extends Node {
  // leafOperation()
  openFile() {}
}

// Composite constructor
class Folder extends Node {
  constructor(name, parent) {
    super(name, parent);

    this.children = [];
  }

  add(node) {
    node.parent = this;
    this.children.push(node);
  }

  remove(node) {
    node.parent = null;
    this.children.splice(this.children.indexof(node), 1);
  }

  getChild(index) {
    return this.children[index];
  }
}

// client code
const root = new Folder("root");
const desktop = new Folder("desktop");
const file1 = new File("main.exe");
const file2 = new File("bootstrap.exe");

root.add(file1);
root.add(desktop);
desktop.add(file2);
```

#### Implementation Notes

- Notice that leafs have unimplemented methods for child-related behavior. This seems odd, but unless I am mistaken, this is what the GoF composite pattern strictly requires.

#### Why Use This Pattern

- The component interface provides an obvious way to interact with the data structure.
- Behavior can cascade through an enormous data structure via a single function call.

#### Consequences

- Seemingly innocuous calls on a composite could cause deeply executing code and performance issues.
- Consider caching results for massive trees.

**[⬆ Table of Contents](#table-of-contents)**

---

### Decorator

The decorator pattern involves wrapping an object (component) at runtime with another object (decorator) and adding additional behavior. The decorator should support the entire interface of the component. Decorators provide an alternative to inheritance.

Since JavaScript supports delegation and dynamic objects, it is already straightforward to add properties to objects at runtime without the need for a wrapping object. As a result, this pattern is not as useful for JavaScript as it is for statically typed languages like Java, C#, etc.

- _Component_ — the object to be decorated.
  - operation() - example of behavior.
- _Decorator_ — wraps a component and supports its interface, typically by forwarding, and adds additional behavior.
  - operation() — supports this behavior from component.
  - additionalOperation() — adds additional behavior.

_Example:_

```javascript
// Component constructor
class Car {
  startGasEngine() {}
}

// Decorator constructor
class DragsterDecorator {
  constructor(car) {
    this.car = car;
  }

  // re-implement decorated object interface
  startGasEngine() {
    return this.car.startGasEngine();
  }

  // add new behavior
  igniteRocketEngine() {}
}

// Decorator (enhanced) constructor
class WarpDriveDecorator extends DragsterDecorator {
  engageHyperdrive() {}
}

// client code
const car = new Car();

const dragster = new DragsterDecorator(car);
dragster.igniteRocketEngine();

const warpDrive = new WarpDriveDecorator(car);
warpDrive.igniteRocketEngine();
warpDrive.engageHyperdrive();
```

#### Example Notes

- The technique from the example above would be a silly way to implement decorators in JavaScript. See the example below for a more idiomatic approach.

_Example - since JavaScript supports delegation and dynamic objects, adding behavior is very easy:_

```javascript
// Component constructor
class Car {
  startGasEngine() {}
}

// Decorator
function dragsterDecorator() {
  this.igniteRocketEngine = () => {};
}

// client code
let car = new Car();

// turn car into a dragster
dragsterDecorator.call(car);

car.igniteRocketEngine();
```

#### Example Notes

- This example isn't strictly an illustration of the decorator pattern because another object doesn't wrap `car`, but the example still has the hallmarks of runtime decorating.

#### Why Use This Pattern

- Inheritance is not feasible.
- To avoid deep inheritance chains by creating many decorators and using them to add the required behavior instead of using different classes.
- To implement aspect-oriented programming. Decorators could be used as aspects to manage cross-cutting concerns.

#### Consequences

- Decorators are slower than inheritance. Instead of reusing prototypes, decorators redeclare methods each time an object is decorated.
- Idiomatic mixins in JavaScript are similar to decorators.

**[⬆ Table of Contents](#table-of-contents)**

---

### Facade

The facade pattern is a single object (facade) that presents a simplified interface to access a set of different objects (subsystems). The facade makes complex subsystems easier to use by only requiring clients to interact with the facade.

- _Facade_ — aggregates and knows about the subsystems' interfaces and forwards client requests to the appropriate subsystem.
- _Subsystems_ — each implements subsystem functionality and has no knowledge of the facade. There can be an unlimited number of subsystems.

_Example:_

```javascript
// Subsystem
function swingBat() {}

// Subsystem
function pitchBall() {}

// Subsystem
function catchBall() {}

// Facade constructor
class Baseball {
  swing() {
    return swingBat();
  }

  pitch() {
    return pitchBall();
  }

  catch() {
    return catchBall();
  }
}

// client code
const baseball = new Baseball();

baseball.swing();
baseball.pitch();
baseball.catch();
```

#### Why Use This Pattern

- When you have many subsystems, and subsystem complexity is high.

#### Consequences

- This pattern can be beneficial in layered architecture to present a simplified interface to a layer. A good example is a facade for the interface at the boundary of the presentation and domain layers.

**[⬆ Table of Contents](#table-of-contents)**

---

### Flyweight

The flyweight pattern is an object (flyweight) that stores shared data for other objects to conserve memory and possibly improve performance. Since flyweights store shared data, they must be immutable or treated as immutable. They are named after the boxing weight class for fighters with weights between 108 to 112 pounds.

This pattern is similar to data normalization techniques that strive to eliminate redundancy.

Note, the GoF make a distinction between intrinsic and extrinsic state. Intrinsic state is internal and not dependent on context. Intrinsic state can be shared. Extrinsic state is dependant on context and can not be stored in a flyweight. However, clients can pass extrinsic state to flyweight methods to retrieve shared intrinsic state.

- _Flyweight_ — defines an interface to store and retrieve shared data. Not created or requested directly by client code.
  - operation(extrinsicState) — extrinsic state can be passed as an argument to retrieve specific intrinsic state data.
- _FlyweightFactory_ — creates and manages flyweights. Client code requests flyweights from this object.
  - getFlyweight(key) — called by client code; retrieves an existent flyweight or creates a new flyweight.

_Example:_

```javascript
// Flyweight constructor
class Flyweight {
  // type and manufacturer are properties of intrinsic state; treat them as
  // immutable
  constructor(type, manufacturer) {
    // intrinsic state
    this.type = type;

    // intrinsic state
    this.manufacturer = manufacturer;
  }

  // operation(x: extrinsicState)
  fly(planeUID) {}
}

// FlyweightFactory constructor
class FlyweightFactory {
  constructor() {
    this.flyweights = {};
  }

  // getFlyweight(key)
  getFlyweight(type, manufacturer) {
    // using type & manufacturer as the key to retrieve flyweights
    let flyweight = this.flyweights[type + manufacturer];

    // if the flyweight doesn't exist, create a new one
    if (!flyweight) {
      this.flyweights[type + manufacturer] = new Flyweight(type, manufacturer);
      flyweight = this.flyweights[type + manufacturer];
    }

    return flyweight;
  }
}

class Aircraft {
  constructor(aircraftType, manufacturer, planeUID) {
    // request a flyweight from the flyweight factory
    this.flyweight = flyweightFactory.getFlyweight(aircraftType, manufacturer);
    this.planeUID = planeUID;
  }

  get type() {
    return this.flyweight.type;
  }

  get manufacturer() {
    return this.flyweight.manufacturer;
  }

  fly() {
    // call a flyweight method and pass extrinsic state
    return this.flyweight.fly(this.planeUID);
  }
}

// client code
const flyweightFactory = new FlyweightFactory();

const aircraft1 = new Aircraft("fixedWing", "boeing", 22);
const aircraft2 = new Aircraft("fixedWing", "boeing", 12);
const aircraft3 = new Aircraft("rotary", "sikorsky", 55);
const aircraft4 = new Aircraft("rotary", "sikorsky", 100);

// you now have two flyweights storing ["fixedWing", "boeing"] and
// ["rotary", "sikorsky"]

aircraft4.fly();
```

#### Example Notes

- Notice that the four aircraft are sharing two flyweights.
- The intrinsic data fields `type` and `manufacturer` (on the flyweights) should be immutable. For the sake of brevity, the example assumes that client code treats the intrinsic data as immutable.

#### Why Use This Pattern

- The program contains objects with large amounts of redundant data.
- One is obsessive with normalization.

#### Consequences

- Flyweights can decrease performance. Lookup times for flyweights can be lengthy because of a vast number of objects, or the lookup method is poorly optimized.

**[⬆ Table of Contents](#table-of-contents)**

---

### Proxy

The proxy pattern uses an object (proxy) to wrap another object (subject) and control access to the wrapped object. The proxy should have an interface that is identical to the subject. The proxy should not violate Liskov's Substitution Principle, and client code should not need to be aware it is working with a proxy instead of the subject.

The proxy pattern is typically used for three purposes:

1. _Remote proxies_ — encapsulate a remote request for subject data.
1. _Virtual proxies_ — an actual subject is only instantiated when the client first requests access to the subject via the proxy; this is an efficient technique.
1. _Protection proxies_ — implementing access control in the proxy to limit access to the subject.

- _Proxy_ — composes the subject and provides an interface that is identical to subject's interface.
- _Subject_ — the object that the proxy manages.

_Example:_

```javascript
// Subject constructor
class BankAccount {
  getMoney() {}
}

// Proxy constructor
class BankAccountProxy {
  constructor() {
    this.dbStarted = false;

    // store a reference to the subject; could be performed on demand
    this.bankAccount = new BankAccount();
  }

  getMoney() {
    // make sure that the database has started before allowing access to
    // the subject
    if (!this.dbStarted) throw new Error("Start the database!");

    return this.bankAccount.getMoney();
  }
}

// client code
const bankAccount = new BankAccountProxy();

bankAccount.getMoney();
```

#### Why Use This Pattern

- This pattern is useful for aspect-oriented programming. Proxies can handle cross-cutting concerns, and client code does not need to be aware of the proxy's existence.

**[⬆ Table of Contents](#table-of-contents)**

---

## Behavioral Patterns

The following patterns concern the responsibilities between objects. For example, many-to-many object relationships create complicated control flow behavior; these patterns help deal with such complexity.

**[⬆ Table of Contents](#table-of-contents)**

---

### Chain of Responsibility

The chain of responsibility pattern is a request passed to a chain of coupled objects (handlers). One — or none — of the handlers can handle the request.

This pattern consists of three parts: (1) _sender object_, (2) _handler objects_, and (3) _request object_. The sender makes the request. The handlers are a chain of one or more objects that choose whether to handle the request or pass it on to another handler. If a handler handles the request, then it is also known as the receiver object. Finally, the request object encapsulates all the internal state of the request.

An example is event bubbling in the DOM. Events propagate through nested HTML elements until one element — or none — handles the request.

- _Handler_ — can handle a request or pass it to another handler. There can be a chain of these objects.
  - handleRequest(request) — assumes responsibility for a request or sends the request to another handler.
  - successor() — (optional) retrieve the next handler in the chain.

_Example - implements a straightforward event bubbling system. Event objects are created and dispatched to buttons when the window is clicked, and the clicked button can handle the event or pass it on to its parent:_

```javascript
// Handler constructor
class Button {
  constructor(title, parent, handleType) {
    this.title = title;
    this.handleType = handleType;

    // successor(); field to retrieve the successor handler
    this.parent = parent;
  }

  // handleRequest()
  click(event) {
    // assume responsibility for the request
    if ((event.type = this.handleType)) {
      console.log("I was clicked!");

      // or pass the request to the successor
    } else {
      if (this.parent) this.parent.click(event);
    }
  }
}

// client code; create the receiver objects
const buttonOne = new Button("Button One", undefined, "click");
const buttonTwo = new Button("Button Two", buttonOne, "wheelclick");
const buttonThree = new Button("Button Three", buttonTwo, "doubleclick");

// sender; set up a listener for clicks
window.addEventListener("click", (event) => {
  // request is the event object
  buttonThree.click(event);

  // buttonOne will log "I was clicked"
});
```

#### Why Use This Pattern

- It reduces coupling between the sender and the receiver.

#### Consequences

- There is no guarantee that the request will be fulfilled.

**[⬆ Table of Contents](#table-of-contents)**

---

### Command

The command pattern is a technique to represent a request as an object.

Requests are encapsulated by objects (commands) that have an execute method that calls behavior on a separate object (receivers). The receiver satisfies the request. Objects that invoke behavior on commands (invokers) do not need to know anything about the implementation of the commands or the receivers. Invokers do not directly interact with receivers.

Commands are also known as object-oriented callbacks.

- _Command_ — has a reference to the receiver.
  - execute() — calls behavior on the receiver.
- _Receiver_ — carries out a request. Not called directly by client code.
  - actionOne() — behavior that can be called by a command.
  - actionTwo() — behavior that can be called by a command.
  - actionThree() — and so on.
- _Invoker_ — object that calls execute() on a command. It invokes the command.

_Example:_

```javascript
// Receiver constructor
class Car {
  turnLeft() {}

  turnRight() {}
}

// Command constructor
class TurnLeftCommand {
  constructor(car) {
    // has knowledge of the receiver
    this.car = car;
  }

  execute() {
    // calls behavior on the receiver
    this.car.turnLeft();
  }
}

// Command constructor
class TurnRightCommand {
  constructor(car) {
    // has knowledge of the receiver
    this.car = car;
  }

  execute() {
    // calls behavior on the receiver
    this.car.turnRight();
  }
}

// client code
const car = new Car();
const turnLeftCommand = new TurnLeftCommand(car);
const turnRightCommand = new TurnRightCommand(car);

// Invoker
const steeringWheel = {
  left: turnLeftCommand.execute.bind(turnLeftCommand),
  right: turnRightCommand.execute.bind(turnRightCommand),
};

steeringWheel.left();
steeringWheel.right();
```

#### Why Use This Pattern

- This pattern promotes loose coupling by decoupling the invoking object from the object that performs the behavior.
- It makes it easy to swap out new command implementations.

#### Consequences

- Callbacks and delegation are typically a more straightforward solution in JavaScript.

**[⬆ Table of Contents](#table-of-contents)**

---

### Interpreter

The interpreter pattern describes how to define a grammar for a language and create an interpreter that is used to parse sentences. In other words, this pattern enables one to create a programming language! Note that this pattern requires that the interpreter be handed an abstract syntax tree (AST) that represents the sentences. This pattern doesn't address how to create the AST. Creating an AST is left up to the developer and can be very hard for non-trivial grammars.

Each grammar rule is a type that implements the interpret method. This way, the root node of an AST can call its interpret method and interpret is then called recursively on all the child nodes, ultimately interpreting the entire sentence.

Steve Yegge whimsically stated that the GoF wrote a book that contains 22 patterns and a practical joke (interpreter).

- _TerminalExpression_ — represents terminal symbols in the grammar. Terminal symbols are the characters of the alphabet that appear in the strings generated by the grammar.
  - interpret() — interpret the expression specifically with regards to the type of expression.
- _NonterminalExpression_ — represents nonterminal symbols in the grammar. Nonterminal symbols are representations of sequences of terminal symbols. In other words, they are placeholders for terminal symbols. Not used in the example below.
  - interpret() — interpret the expression specifically with regards to the type of expression.
- _Context_ — contains information that is global to the interpreter.

_Example - https://en.wikipedia.org/wiki/Interpreter_pattern - ported from Java to JavaScript and simplified:_

```javascript
// TerminalExpression constructor
class Number {
  constructor(num) {
    this.number = num;
  }

  interpret(variables) {
    return this.number;
  }
}

// TerminalExpression constructor
class Plus {
  constructor(left, right) {
    this.leftOperand = left;
    this.rightOperand = right;
  }

  interpret(variables) {
    return (
      this.leftOperand.interpret(variables) +
      this.rightOperand.interpret(variables)
    );
  }
}

// TerminalExpression constructor
class Variable {
  constructor(name) {
    this.name = name;
  }

  interpret(variables) {
    if (!variables[this.name]) return;

    return variables[this.name].interpret(variables);
  }
}

// implementation detail for the pattern; takes a sentence and creates an
// abstract syntax tree composed of TerminalExpression and
// Non-TerminalExpression objects
class Parser {
  constructor(expression) {
    const expressionStack = [];
    const tokens = expression.split(" ");

    tokens.forEach((token) => {
      if (token === "+") {
        const subExpression = new Plus(
          expressionStack.pop(),
          expressionStack.pop()
        );
        expressionStack.push(subExpression);
      } else {
        expressionStack.push(new Variable(token));
      }
    });
    this.syntaxTree = expressionStack.pop();
  }

  interpret(context) {
    return this.syntaxTree.interpret(context);
  }
}

// client code

// replace the variables with values and evaluate the sentence
const expression = "w x +";
const sentence = new Parser(expression);

// Context
const context = {};
context["w"] = new Number(5);
context["x"] = new Number(10);

const result = sentence.interpret(context);

// outputs 15
console.log(result);
```

#### Example Notes

This example is extremely simple. It only supports the "plus" operator, and there are no nonterminal expressions.

#### Why Use This Pattern

- You have a language with a simple grammar that needs to be parsed, and performance is not a concern.

#### Consequences

- It is easy to change or extend the grammar because grammar rules are types; therefore, one can use inheritance to extend the grammar.

**[⬆ Table of Contents](#table-of-contents)**

---

### Iterator

The iterator pattern is a technique to access the elements of a collection object without exposing the collection's implementation.

The collection (aggregate) has a method to create an iterator object, often called the cursor. The iterator should encapsulate traversing the aggregate.

- _Aggregate_ — the collection.
  - createIterator() : Iterator — create and return the iterator.
- _Iterator_ — used to access the data in an aggregate without exposing its implementation.
  - first() : Element — example of a typical iterator method; returns the first element in the aggregate.
  - next() : Element — example of a typical iterator method; returns the next element in the aggregate.
  - currentItem() : Element — example of a typical iterator method; returns the current element in the aggregate.

_Example:_

```javascript
// Aggregate constructor
class Collection {
  // implementation detail; using an array as the backing store; client code
  // does not need to know this
  constructor(elements) {
    this.elements = elements;
  }

  // createIterator(): Iterator
  getCursor() {
    return new Cursor(this);
  }
}

// Iterator constructor
class Cursor {
  constructor(collection) {
    this.collection = collection.elements;
    this.index = 0;
  }

  first() {
    this.index = 0;

    return this.collection[this.index];
  }

  next() {
    this.index++;

    return this.collection[this.index];
  }

  currentItem() {
    return this.collection[this.index];
  }
}

// client code
const collection = new Collection(["a", "b", "c"]);
const cursor = collection.getCursor();

console.log(cursor.first());
console.log(cursor.next());
console.log(cursor.currentItem());

// the following is logged to the console: "a b b"
```

#### Why Use This Pattern

- This pattern can simplify the interface of a complex collection.
- It can limit the use of leaky abstractions present in collections.
- It allows for swapping out iterator implementations.

#### Consequences

- This pattern may create useless complexity.

**[⬆ Table of Contents](#table-of-contents)**

---

### Mediator

The mediator pattern involves objects (colleagues) that do not communicate directly with each other. Instead, a colleague interacts with an object (mediator), and the mediator performs the interactions with the other colleagues.

Think of an air traffic controller; aircraft do not communicate with one another. Instead, they communicate with the air traffic controller (mediator), who sends messages to the appropriate airplanes (colleagues).

This pattern promotes loose coupling by allowing objects' interactions to vary independently of the objects themselves.

- _Mediator_ — defines an interface to invoke behavior on colleagues.
- _Colleagues_ — each has a reference to the mediator. Colleagues communicate with the mediator instead of each other.

_Example:_

```javascript
// Mediator constructor
class Chatroom {
  constructor() {
    this.people = [];
  }

  join(person) {
    this.people.push(person);
  }

  leave(person) {
    this.people.splice(this.people.indexOf(person), 1);
  }

  postMessage(person, msg) {
    // send the message to everyone except the person who posted it
    for (let i = 0; i < this.people.length; i++) {
      if (this.people[i] !== person) this.people[i].receiveNewMessage(msg);
    }
  }
}

// Colleague constructor
class Person {
  constructor(chatroom) {
    // store a reference to the mediator
    this.chatroom = chatroom;
    this.chatroom.join(this);
  }

  leaveRoom() {
    this.chatroom.leave(this);
  }

  addNewMessage(msg) {
    // communicate with the mediator directly instead of colleagues
    this.chatroom.postMessage(this, msg);
    console.log(`I posted a new message -> ${msg}`);
  }

  receiveNewMessage(msg) {
    console.log(`someone posted a new message -> ${msg}`);
  }
}

// client code; pass in the mediator to each colleague
const chatroom = new Chatroom();

const bob = new Person(chatroom);
const sally = new Person(chatroom);

bob.addNewMessage("Hi Everyone");
sally.addNewMessage("Hi, it was nice of you to say hello.");
sally.addNewMessage("Goodbye.");
sally.leaveRoom();
bob.addNewMessage("Hello?");
```

#### Why Use This Pattern

- There is a need to detangle coupled objects and reduce complexity.
- This pattern centralizes control, which makes control flow expressive and clear.
- It makes it easier to alter many-to-many interactions because only the mediator needs to be changed instead of the colleagues.

#### Consequences

- Reuse is difficult because of the numerous objects that may interact.

**[⬆ Table of Contents](#table-of-contents)**

---

### Memento

The memento pattern involves storing a snapshot of the state of an object (originator) in another object (memento) so that the originator's state can be restored after changes are made. In short, a technique to add undo functionality.

- _Memento_ — stores the internal state of the originator at a moment in time.
- _Originator_ — the object with the state to capture.
  - setMemento(m: Memento) — restore the originator's state to the state represented by the passed memento.
  - createMemento(): Memento — create and return a memento.
- _Caretaker_ — the object that calls createMemento() and setMemento(m) on the originator. The caretaker does not directly use or examine mementos.

_Example:_

```javascript
// Memento constructor
class Memento {
  constructor(txt) {
    this.txt = txt;
  }
}

// Originator constructor
class Sentence {
  updateText(txt) {
    this.txt = txt;
  }

  // createMemento(): Memento
  createMemento() {
    return new Memento(this.txt);
  }

  // setMemento(m: Memento)
  setMemento(memento) {
    // restore state based on the memento
    this.txt = memento.txt;
  }
}

// Caretaker constructor
class WordProcessor {
  constructor() {
    this.mementos = [];
  }

  addSentence(txt) {
    this.sentence = new Sentence();
    this.sentence.updateText(txt);
    this.mementos.push(this.sentence.createMemento());
  }

  updateSentence(txt) {
    this.sentence.updateText(txt);
    this.mementos.push(this.sentence.createMemento());
  }

  undo() {
    // get older memento
    let memento = this.mementos[this.mementos.length - 2];
    this.sentence.setMemento(memento);
  }
}

// client code
const words = new WordProcessor();
words.addSentence("It was the best of times, it was the worst of times.");
words.updateSentence("Stuff was happening...");

// I liked the old sentence more
words.undo();
```

#### Why Use This Pattern

- There is a need for undo functionality.
- Obtaining state directly from the originator would expose its implementation and break encapsulation.

#### Consequences

- This pattern can simplify originators.
- Mementos could consume large amounts of memory if describing state requires large amounts of data.

**[⬆ Table of Contents](#table-of-contents)**

---

### Observer

The observer pattern facilitates one-to-many relationships. The pattern involves one object (subject) that is observed by a group of other objects (observers). When the subject changes state, it informs the observers. Observers are not aware of each other.

This pattern is the backbone of event-driven systems.

- _Subject_ — the subject of observation. It has references to its observers.
  - attach(o: Observer) — add the passed observer to the list of objects that are observing the subject.
  - detach(o: Observer) — remove the passed observer from the list of objects that are observing the subject.
  - notify() — notifies all the observers that the subject has changed.
- _Observer_ — observes the subject. Observers are notified by the subject when the subject's state changes. Observers can also explicitly change the state of the subject. There can be an unlimited number of observers for each subject.
  - update() - called by the subject when its state changes.

_Example:_

```javascript
// Subject constructor
class Subject {
  constructor() {
    this.observers = [];

    // state variable
    this.myState = "happy";
  }

  // attach(o: Observer)
  attach(observer) {
    this.observers.push(observer);
  }

  // detach(o: Observer)
  detach(observer) {
    this.observers.splice(this.observers.indexof(observer), 1);
  }

  changeState() {
    // change state
    this.myState = "sad";

    // notify all the observers of the state change
    this.notify();
  }

  // notify()
  notify() {
    for (let i = 0; i < this.observers.length; i++) {
      this.observers[i].update();
    }
  }
}

// Observer constructor
class Observer {
  constructor(subject) {
    this.subject = subject;

    subject.attach(this);
  }

  // update()
  update() {
    console.log(`The subject was updated. It's now ${this.subject.myState}.`);
  }
}

// client code
const subject = new Subject();
const observer1 = new Observer(subject);
const observer2 = new Observer(subject);
const observer3 = new Observer(subject);

subject.changeState();

// "The subject was updated. It's now sad." would log 3 times
```

#### Why Use This Pattern

- When the dependent objects (observers) are unknown in number or type.

#### Consequences

- Trivial changes to the subject could cause massive refactoring due to a large number of observer types.

**[⬆ Table of Contents](#table-of-contents)**

---

### State

The state pattern involves an object (context) that can alter its behavior by using other objects (states). To client code, the context appears to change type.

- _Context_ — the state machine. It maintains a reference to the current state object. Client code makes direct requests to the context.
  - request() — client request. It will be forwarded/delegated to the current state object.
- _State_ — base type for all the state objects. It includes default behavior.
- _State subtypes_ — specific states. Are not called directly by client code.
  - handle() — called by the context. It handles indirect client requests.

_Example:_

```javascript
// Context constructor
class StateMachine {
  constructor() {
    // maintain a reference to the current state object; sets default state
    // object to a StateOne instance
    this.currentState = new StateOne(this);
  }

  // request()
  request() {
    // delegate the client request to the current state object
    return this.currentState.handle.call(this);
  }

  // implementation detail; allows state objects to change context's current
  // state object
  setState(state) {
    this.currentState = state;
  }
}

// State constructor; no default behavior for this example
class State {
  constructor() {}
}

// State subtype constructor
class StateOne extends State {
  handle() {
    console.log("My state is one.");

    // change the state of the context object
    this.setState(new StateTwo());
  }
}

// State subtype constructor
class StateTwo extends State {
  handle() {
    console.log("My state is two.");

    // change the state of the context object
    this.setState(new StateThree());
  }
}

// State subtype constructor
class StateThree extends State {
  handle() {
    console.log("My state is three.");
  }
}

// client code
const stateMachine = new StateMachine();

// the following will log: "My state is one."
stateMachine.request();

// the following will log: "My state is two."
stateMachine.request();

// the following will log "My state is three." twice
stateMachine.request();
stateMachine.request();
```

#### Example Notes

- Note that when `StateMachine#request` calls `State#handle`, the context is passed via delegation : `this.currentState.handle.call(this)`. Passing the context as an implicit argument is convenient.

#### Why Use This Pattern

- It is a clean way to delineate the possible states of an object. Also, it makes state changes explicit, which is very expressive and clear.

#### Consequences

- State objects can now be shared.

**[⬆ Table of Contents](#table-of-contents)**

---

### Strategy

The strategy pattern involves a family of algorithms encapsulated as objects (strategies) and allows them to vary independently from the objects (contexts) that use them.

- _Context_ — uses a strategy and maintains a reference to it. May define an interface for the strategies to access context data.
- _Strategies_ — used by the context. Each strategy defines an algorithm, and each strategy has the same interface.

_Example:_

```javascript
// Context constructor
class Travel {
  setStrategy(strategy) {
    this.strategy = strategy;
  }

  getTravelTime() {
    return this.strategy.time();
  }

  getCost() {
    return this.strategy.cost();
  }
}

// Strategy constructor
class Chevy {
  time() {
    const time = 1000;

    return time;
  }

  cost() {
    const cost = 10;

    return cost;
  }
}

// Strategy constructor
class SR71 {
  time() {
    const time = 10;

    return time;
  }

  cost() {
    const cost = 1000000;

    return cost;
  }
}

// client code

// create two strategies
const car = new Chevy();
const plane = new SR71();

// create the context
const travel = new Travel();

travel.setStrategy(car);

// logs "time -> 1000, cost -> 10"
console.log(`time -> ${travel.getTravelTime()}, cost -> ${travel.getCost()}`);

travel.setStrategy(plane);

// logs "time -> 10, cost -> 1000000"
console.log(`time -> ${travel.getTravelTime()}, cost -> ${travel.getCost()}`);
```

#### Why Use This Pattern

- There is a need to change algorithms at runtime.
- It can be useful when many objects vary only slightly. This pattern allows for configuration instead of bloated type hierarchies.
- This pattern can move complex conditional logic out of the context object and into a strategy to promote clean code and cohesion.

#### Consequences

- Clients must be aware of the strategies, which can cause abstractions to leak.
- Since all strategies share the same interface, some may not use the entire interface; this violates the interface segregation principle.
- The pattern increases the number of objects in the application.

**[⬆ Table of Contents](#table-of-contents)**

---

### Template Method

The template method pattern involves defining an abstract class (AbstractClass) that outlines the steps of an algorithm in one method and delegates the implementation of some or all of the algorithm's steps to subclasses (ConcreteClasses). This technique allows the subclasses to redefine the steps of the algorithm without changing the algorithm's overall structure.

- _AbstractClass_ — the base class that (1) defines all of the steps (methods) of an algorithm, (2) defines a method that calls all of the steps, and (3) can optionally implement some shared steps.
  - templateMethod() - calls each of the steps that define the algorithm. Called by client code.
  - stepOne() — example of a step called by templateMethod().
  - stepTwo() — example of a step called by templateMethod().
  - stepThree() — and so on.
- _ConcreteClass_ — each specific algorithm inherits from AbstractClass and overrides the steps needed to add its particular implementation.

_Example:_

```javascript
// AbstractClass
class Pet {
  // shared step; implemented in AbstractClass to reduce duplicate
  // implementations in ConcreteClass instances
  eat() {
    console.log("How cute!");
  }

  // all steps should have a placeholder even if they are unimplemented
  play() {}
  fetch() {}
  purr() {}

  // templateMethod(); calls all the steps
  love() {
    this.eat();
    this.play();
    this.fetch();
    this.purr();
  }
}

// ConcreteClass
class Dog extends Pet {
  // specific implementation of the fetch step
  fetch() {
    console.log("woof!");
  }
}

// ConcreteClass
class Cat extends Pet {
  // specific implementation of the purr step
  purr() {
    console.log("meow!");
  }
}

// client code
const dog = new Dog();
dog.love(); // logs "How cute! woof"

const cat = new Cat();
cat.love(); // logs "How cute! meow"
```

#### Why Use This Pattern

- This pattern can encourage code reuse among different algorithms.

#### Consequences

- This pattern is an example of the Hollywood principle (don't call us, we'll call you), where a parent class calls methods on derived classes.

**[⬆ Table of Contents](#table-of-contents)**

---

### Visitor

The visitor pattern involves adding behavior to an object (element) by using other objects (visitors). Visitors are passed to elements, which then invoke a visit method on the visitors; then, using the element's interface, the visitor adds behavior to the element.

Note, this pattern, as described by the GoF, uses double dispatch. However, since JavaScript is a single dispatch language, type checking the argument will be used instead.

- _Element_ — the object to which a visitor adds behavior. It only requires a single method: accept.
  - accept(visitor: Visitor) { visitor.visit(this); } — always the same function signature and body.
- _Visitor_ — adds behavior to the element. In a double dispatch language, one would implement a visit(element) method for each element type. Since JavaScript is single dispatch, use type checking instead.
  - visit(element: Element) — called by element. Is passed the element that made the call. Uses the element's interface to add behavior to element.

_Example:_

```javascript
// Element constructor
class Dog {
  accept(visitor) {
    visitor.visit(this);
  }

  setName(name) {
    this.name = name;
  }

  sayMyName() {
    console.log(`I am type Dog. My breed is ${this.name}`);
  }
}

// Element constructor
class Cat {
  accept(visitor) {
    visitor.visit(this);
  }

  setName(name) {
    this.name = name;
  }

  sayMyName() {
    console.log(`I am type Cat. My breed is ${this.name}`);
  }
}

// Visitor constructor
class Mexican {
  visit(element) {
    // no double dispatch; thus must use type checking
    if (element instanceof Dog) {
      element.setName("Chihuahua");
    } else if (element instanceof Cat) {
      element.setName("Aztec");
    }
  }
}

// Visitor constructor
class Canadian {
  visit(element) {
    // no double dispatch; thus must use type checking
    if (element instanceof Dog) {
      element.setName("Newfoundland");
    } else if (element instanceof Cat) {
      element.setName("Foldex");
    }
  }
}

// client code

// elements
const dog = new Dog();
const cat = new Cat();

// visitors
const mexican = new Mexican();
const canadian = new Canadian();

// mexican
dog.accept(mexican);
cat.accept(mexican);
dog.sayMyName(); // logs "I am type Dog. My breed is Chihuahua";
cat.sayMyName(); // logs "I am type Cat. My breed is Aztec";

// canadian
dog.accept(canadian);
cat.accept(canadian);
dog.sayMyName(); // logs "I am type Dog. My breed is Newfoundland";
cat.sayMyName(); // logs "I am type Cat. My breed is Foldex";
```

#### Why Use This Pattern

- For objects with many methods, the complexity can be broken up into visitors to increase cohesion.
- One is using a programming language — not JavaScript — that supports double dispatch.

#### Consequences

- Since visitors rely on element's interface, it is tempting for element to allow visitors to access private state, thereby breaking encapsulation.
- Since the added behavior doesn't change element's structure, this pattern doesn't violate the open/closed principle.
- Element does not need to change if a visitor changes. Thus, this pattern does not violate the single-responsibility principle.

**[⬆ Table of Contents](#table-of-contents)**
