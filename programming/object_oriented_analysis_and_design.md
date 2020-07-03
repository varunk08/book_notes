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
Hierarchy\
Inheriting from an Interface\
Polymorphism and dynamic binding

#### Genericity
Using placeholders for types.


### Elementary Design patterns
**Iterator** A pattern where an object is used to traverse a collection. Must support operations like next() and get()
**Singleton** Only a single instance. Private constructor. Use static getInstance()
````
class Singleton {
public:
static Singleton* GetInstance() {
  if (pInstance == nullptr) {
    pInstance = new Singleton();
  }
  return pInstance;
}
Singleton(const Singleton& other) = delete;
void operator=(const Singleton& other) = delete;

private:
Singleton* pInstance = nullptr;
Singleton() {}
};
````
