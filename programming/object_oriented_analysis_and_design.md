## Object oriented analysis and design notes
*Brahma Dathan. Sarnath Ramnath. 2015*

#### Part 1
Foundational concepts.
#### Part 2
Useful design patterns.
Case study.
#### Part 3
Advanced topics.\
Inheritance. \
*Liskov Substituition Principle and Open-closed principle*\
Case studies.

---
### Introduction
Process was based on functional modules.  
Limited reusability on change of process logic.  
Process is broken down into objects which can be independent of process changes.  

### Relationship between Classes
(in order of restrictiveness)
1. Association
2. Inheritance
3. Genericity (type parameterization)

#### Association
Arity (one-one, one-many)  
Aggretation/ composition (triangle composed of vertices)

#### Inheritance
Hierarchy  
Inheriting from an Interface  
Polymorphism and dynamic binding

#### Genericity
Using placeholders for types.


### Elementary Design patterns
**Iterator** A pattern where an object is used to traverse a collection. Must support operations like next() and get()  
**Singleton** Only a single instance. Private constructor. Use static getInstance()
````
class Singleton {
public:
  static Singleton& GetInstance() {
      static Singleton instance;
      return instance;
  }
  Singleton(const Singleton& other) = delete;
  void operator=(const Singleton& other) = delete;
  
private:
  Singleton(){}
};
````
Singleton's can be inherited from to create Singleton subclasses.  
**Adapter** 
Satisfy an interface requirement using an incompatible implementation through an adapter class.


### Analysing a system
#### 1. Gather the requirements.
Functional requirements - interaction b/w system and users
Non-functional requirements - response time, usability, accuracy.
#### 2. Precisely document the functionality required of the system.
- Use case analysis
#### 3. Develop a conceptual model of the system, listing conceptual classes and their relationships.
one method: look at the use cases and pick out all the nouns in the description of the requirements  
Use knowledge of the domain

### Design and implementation
- what platforms will the system run on?
- languages and programming paradigms used
- user interfaces provided by the system
- failure handling (data loss and corruption prevention)
- distributed?
- data storage

### Advanced OOP design topics
Major goal of inheritance is reuse.  
Inheritance - subclassing or implementing interfaces
#### Applications of inheritance:
**Restricting behaviours**: Inheritance can be used to restrict. `Square` is a restricted version of `Rectangle`. `BlueLabel` is a restricted version of `Label` widget.  
**Abstract superclass**: Force extract common functionality from potential future sub-classes. `Account` is a general concept, while `CheckingAccount` and `SavingsAccount` are concrete.  
**Adding Features**: `DataStream` is the base and `ReReadableDataStream` is with added feature of `reReadability`.  
**Hiding features of the superclass**: `Queue` is a restricted version of `List` that allows only add/remove from one end.  
**Combining structural and type inheritance**: `BinarySearchTree` can be seen as a class that extends `BinaryTree` (structural) and implements `OrderedList` interface (implementation inheritance).  
**Limitations**:  
- deep hierarchies
- may need to hide super class members
- derived class's type may not be a true subType of the super class's type.
**Address limitations**:  
- inherit from abstract types than from concrete types
- favor composition over inheritance

### Modeling with Finite State Machines
GUI systems  
Unlike say a `Library` system, a `microwave` system doesn't have any standard business processes for use-case modeling.  
#### Finite state machines
Transition defined by 4-tuple `(si, sf, I, O)`  
si = initial state, sf = final state, I = input, O = output if any  
Mealy machine - output depends on the event and the current state  
Moore machine - output depends only on the current state  
Table of `(state, event)` pairs  
There are algorithms for state minimisation  
**Solution**  
1. Identify conceptual classes (construct a list of nouns)
2. Identify software classes
