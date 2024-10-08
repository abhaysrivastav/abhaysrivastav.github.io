# Design Patterns from GoF

## Adapter Design Patterns (Structural)

### Intent
Convert the interface of a class into another interface that client expect. Adapter lets classes work together that coult not otherwise because of incompatible interface.

### Participants 
**Target** : Interface that is visible to the client.

**Adapter** : It implements the Target interface and translate the client request to the "Adaptee" in a way it can understand. 

**Adaptee** : Define an existing interface that is incompatible with target interface. The Adapter uses this class and translate its interface so that it can work with target. 

**Client** : Collaborates with objects that implement the Target interface. 

![Adapter Pattern : Source Head First ](assests/adapter.JPG) Source : Head First 

## Composite Design Patterns (Structural)
### Intent
Compose the object into the tree like structures to represent the part-whole hierarchies. Composite lets clients treat individual object and composition of the object in the same way.

### Participants 

**Component** : Declares the interface for objects in composition, which can be treated uniformly.

**Leaf** : Represent the leaf object in composition. It implements the Component Interface.

**Composite** :  Container of child component, and it also implements Component Interface. 

**Client** : It interact with the composite object using Component interface. Client need not to know whether its working with leaf object or composite object.

![Composite Pattern : Source Head First ](assests/Composite.JPG) Source : Head First 


## Decorator Pattern  (Structural)

### Intent 

Attach additional responsibilities to an object dynamically Decorators provide a flexible alternative to subclassing for extending functionality. 

### Participants 

**Component**  : Objects that can have responsibilites added to them dynamically. 

**ConcreteComponent** : Implements the Component interface and define the core behavior. 

**Decorator** : Implement the Component interface and contains the reference to a Component Object.

**Concrete Decorator** : Extende the decorator class to add responsibilities. 

![Composite Pattern : Source Head First ](assests/Decorator.JPG) Source : Head First 


## Facade Design Pattern

Provide a unified interface to a set of interfaces in a subsystem. It defines a higher level interface that makes the subsystems earier to use. 

### Participants 

**Facade** : A simple interface to complex subsystem. It delegates the client request to approprate subsystem. 

**Subsystem Classes** : These classes implement the subsystem's functionality. Subsystem classes are complex and handle the actual work but it dont directly interact with the client. 

![Composite Pattern : Source Head First ](assests/facade.JPG) Source : Head First 

## State Design Pattern

### Intent 
Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

### Participants 

**Context**:
 - Context holds a reference to  a state object, which represent the current state of the context.
 - It delegates state-specific behavior to current state object.
 - It can also transition to another state by changing the reference to a different State object.

**State** (Abstract State):
 - The State interface declares the methods that all concrete state must implement . These methods encapsulate the behavior associated with perticular state of context.

**ConcreteState**: 
 - Each ConcreteState corresponds to a specific state of the context and define its specific behavior. These classes handle requests from Context and also decide the transition the Context to a different state.  

![Composite Pattern : Source Head First ](assests/state.JPG) Source : Head First 

 
## Command Design Patterns 

### Intent 
Encapsulate request as an object, thereby letting you parameterize clients with different requests, queue or log requests and support undoable operation. 

**It Means** : Wrap the request in an object, so that we can give it to a different part of the program. 

### Participants

### Command (Interface)
 -  Its an interface for executing an operation. It includes an **execute()** method, which is implemented by concrete command classes to perform specific actions. 

### ConcreteCommand
 - Implements the **Command** interface and define the binding between receiver object and an action. It calls methods on the Receiver to perform an specific an action. 

### Receiver 
 - It knows actually how to perform an operation for any request. 

### Invoker:
 - It asks the command to carry out the request. It stores a command object and and invokes Command's trigger() method. 

### Client
 - Creates a **ConcreteCommand** and set its **Receiver**.
 - Configure the **Invoker** with the **ConcreteCommand** object. 

![Composite Pattern : Source Head First ](assests/command.JPG) Source : Head First 

## Proxy Pattern

### Intent 
Provide a surrogate or placeholder for another object to control access to it.

### Participants

#### Subject
First we have a Subject, which provides an interface for the RealSubject and the Proxy. By implementing the same interface, the Proxy can be substituted for the RealSubject anywhere it occurs.

#### Proxy
The Proxy holds a reference to the RealSubject. In some cases, the Proxy may be responsible for creating and destroying the RealSubject. Clients interact with the RealSubject through the Proxy. Because the Proxy and RealSubject implement the same interface (Subject), the Proxy can be substituted anywhere the subject can be used. The Proxy also controls access to the RealSubject; this control may be needed if the Subject is running on a remote machine, if the Subject is expensive to create in some way or if access to the subject needs to be protected in some way.

#### RealSubject:
The RealSubject is the object that does the real work. It’s the object that the Proxy represents and controls access to.

![Proxy Pattern : Source Head First ](assests/proxy.JPG) Source : Head First 

## Strategy Design Pattern

### Intent 

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Participants

#### Context

Responsibility: The Context maintains a reference to a Strategy object. It is the main object that interacts with the strategy and delegates the work to the strategy. The Context doesn't implement the algorithm itself but relies on the strategy to do so.
Example: A SortingContext class might use a SortStrategy object to sort data. The Context is responsible for accepting the data and deciding when to call the strategy, but it doesn't know or care how the sorting is done.
Key Role: Encapsulates and uses a Strategy. The context typically provides an interface for clients to interact with, and behind the scenes, it delegates the task to the selected strategy.

#### Strategy

Responsibility: This declares the interface for all the algorithms (strategies) that can be used interchangeably. Each concrete strategy will implement this interface.
Example: A SortStrategy interface might define a method like sort(), which concrete sorting strategies (e.g., QuickSort, MergeSort) will implement.
Key Role: The common interface that allows the client to interact with different strategies in a uniform way.

#### ConcreteStrategy

Responsibility: These are the classes that implement the Strategy interface. Each class provides a specific algorithm or behavior.
Example: A QuickSort class that implements the SortStrategy interface and provides the logic for quicksort, or a MergeSort class that implements merge sort.
Key Role: Each concrete strategy encapsulates a specific algorithm or behavior and can be swapped with other strategies that implement the same interface.

![Strategy Pattern : Educative ](assests/strategy.JPG) Source : Educative 






