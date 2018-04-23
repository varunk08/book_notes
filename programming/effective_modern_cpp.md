### Effective Modern C++ by Scott Meyers

There have been four official versions of C++, each named after the year in which the corresponding ISO Standard was adopted: C++98, C++03, C++11 and C++14.  

Foundation of `Move semantics` is distinguishing expressions that are `rvalues` and `lvalues`.  
`rvalues` indicate objects eligible for move operations, while `lvalues` generally don't.  
`rvalues` correspond to temporary objects returned from functions, while `lvalues` correspond to objects you can refer to, either by name or by following a pointer  or `lvalue` reference.  


Distinction between arguments and parameters of a function is important, because parameters are `lvalues` but the arguments with which they are initialized may be `rvalues` or `lvalues`.  

*Exception safety basic guarantee*: assuring callers that even if an exception is thrown, program invariants remain intact (no data structures are corrupted).  
*strong guarantee*: state of the program remains as it was prior to the call.  

Function objects created through lambda expressions are known as *closures*  

*smart pointers* overload the pointer-dereferencing operators (`operator-> and operator*`).  

#### 1: Template type deduction

In a scenario like this,

````
template<typename T>
void f(ParamType param);

f(expr)
````

_key_: the type `T` deduced is dependent on both `expr` and the form of `ParamType`.  
A *univeral reference*'s declared type is `T&&`  

compile time array size func using template function which takes in a reference to an array:
````
template <typename T>
constexpr size_t arraySize(T(&)[N]) noexcept
{
    return N;
}
````
````
template<typename T>
void func(xxx param);

expr = xxx;

func(expr);

````

#### 2: `auto` type deduction  
Each of the auto-declarations are treated in the same manner as a template type dedcution, as though the auto declared variable is passed to a template function.  
`auto` can deduce initializer list types by assuming it to be `std::initializer_list<T>` but templates cannot.  
````
auto x = 27;
const auto cx = x;
const auto& rx = x;
````
is treated as:  
````
template <typename T>
void func_for_x(T param);

func_for_x(27);

template <typename T>
void func_for_cx(const T param);

func_for_cx(cx);

template <typename T>
void func_for_rx(const T& param);

func_for_rx(rx);
````

#### 3: `decltype`:
given an expression `decltype` returns the type.  
`decltype` almost always yields the type of an object without modifications.  
primary use is declaring function templates where the return type of the function depends on its parameter types.  
To make sure a return type of `auto` does not use template type deduction, `decltype(auto)` is used to indicate that `decltype`  rules should be used for type deduction instead.  

#### 4: Know how to view `decltype`(s):
Compile time Trick: Use an un defined template class and declare variables with it so the compiler will show an error along with the actual types of the variables declared.  
Runtime: use `typeid(variable).name()` in a `printf`  
`std::type_info::name` from IDEs is not reliable. The better alternate is Boost's TypeIndex library.  
`boost::typeindex::type_id_with_cvr` takes the type as a template argument and does not remove const, volatile or reference qualifiers.  

#### 5: Prefer `auto` to explicit type declarations:  
apparently dereferenced iterators type is known only to the compiler. so use auto to dereference an iterator and use by value.  
`std::function` can be any callable object in c++11. It generalizes the function pointer.  
The return type of `vector.size()` is `std::vector<int>::size_type` which is a unsigned integer.  

#### 6: Use the explicitly typed initializer idiom when auto deduces undesired types.  
C++ forbids reference to bits.  
````
std::vector<bool> features(const Widget& w);
auto highPriority = features(w)[5];
````
Here High priority is a `std::vector<bool>::reference` and not `bool`.  
`auto` incorrectly deduces the type since `operator[]` cannot return a reference to bool.
The value deduced is dependent on how `std::vector<bool>::reference` is implemented. `std::vector<bool>::reference` is an example of a proxy class. The std library's smart pointers are also proxy classes. Some classes in cpp employ *expression templates*. Avoid code of this form: `auto someVar = expression of "invisible" proxy class type;`  
`static_cast<type>` would have to be used to explicitly assign such expressions to `auto` declared variables.  
*invisible proxies*: those proxy classes that are returned for expressions in place of whay you would actually expect is returned!  

#### 7: Distinguishing between `()`  and `{}` for object creation.
confusion occurs with initialization of the form `type var = {};`. does assignment happen here?  
*braced initialization* is universal and strict type checking is enforced.  
cpp has *most vexing parse* problem where a call to a constuctor with no arguments is deduced as a function declaration. Using braces solves this problem. `Widget w1()` vs `Widget w1{}`. The former is a func declaration while the latter is a call to the constructor.    
`std::atomics` are uncopyable.  
Initialization using `=` doesn't check for narrowing conversions.  
If the parameters within the `std::initializer_list` which is used as the only argument for the constructor cannot be matched by the parameters, then the `non std::initializer_list` constructors are used.  
Empty braces `{}` used in construction results in default constructor being called and not the constuctor that takes `std::initializer_list` with an empty list.  
  This matters in `std::vector<>`. It takes an `std::initializer_list` in one constructor and in another constructor there are two arguments, first one taking num elements and second one taking initialization value for each element.  
  This is also a problem when creating variadic templates that take multiple variable number of arguments.  

#### 8: Prefer `nullptr` to `0` and `NULL`
The type of `NULL` is implementation defined. If `NULL` is defined as a long, then conversion of `long` to `int`, `bool` and `void*` are equally good. Guideline is to avoid function overloading on pointer and integral types. `nullptr` doesn't have an integral or pointer type. The type of `nullptr` is `std::nullptr_t` and it can convert to all pointer types.

#### 9: Prefer alias declarations to typedefs
C++11 supports `alias` which should be used instead of `typedef`
````
using TypeName = type;
````
`alias` declarations can be templatized. Example, a templated list with a custom allocator:
````
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;

MyAllocList<Widget> lw;
````
Names of dependent types must be preceded with *typename*  

#### 10: Prefer scoped `enums` to unscoped `enums`
Forward declaring enums, using `enum class` prevents the need for recompilation if it's changed.  
To save memory, compilers choose the smallest underlying type for an unscoped enum. Like choosing `char` as enum type if there are only a few enums.  
The default type for scoped enums is `int`. This can be specified.  
Use `noexcept` when the function will never throw an exception.

#### 11: Prefer `deleted` functions to private undefined ones.
C++ defines default functions for you. The C++98 approach was to declare them private and not define them. The copy constructor and copy assignment operator are handled this way. Use `=delete` to mark the copy constructor and copy assignment operator as *deleted* functions in C++11. Using these functions will cause compilation to fail, whereas in the previous method of declaring them private would only be caught at link-time. Non-member functions can also be *deleted*. This prevents default overloaded functions from being used. Template functions can also be deleted to prevent unwanted compiler provided overloads. Function templates inside a class cannot have a different access level compared to the main template.

#### 12: Declare overriding functions `override`
virtual function overriding is what makes it possible to invoke a derived class function through a base class interface. In c++11 the base function and derived function must have similar *reference qualifiers*.
````
void doWork() &; // applies only when *this is an lvalue
void doWork() &&; // applies only when *this is an rvalue
````
Compiler warnings vary in overriding function specification. There are many things that can go wrong `const`-ness, lvalue or rvalue type, incorrect parameter type etc. The keyword `override` is a keyword only if found in end of function.
`rvalues` are temporary objects.

#### 13: Prefer `const_iterator`s to `iterator`s.
In c++11 container functions can return positions as `const_iterator`s. In c++98 it was not so convenient to use `const_iterator`s but  in c++11 and c++14, it is made practical and so should be used.

#### 14: Declare functions `noexcept` if they wont emit exceptions
`noexcept` functions are more optimizable. Funtions with *wide contracts* don't rely on preconditions and are generally `noexcept`. Functions with *narrow contracts* are dependent on pre-conditions. Conditional `noexcept`s exist. ex: `noexcept(false)`. If a functions shouldnt' throw exceptions, then return codes for errors will have to be implemented - larger runtime penalty due to branching.

#### 15: Use `constexpr` whenever possible
constexpr is a compile time const. It need not be, but it helps the compiler optimize things. For example, for embedded applications, the compiler can put constants in read-only-memory.
`constexpr` functions act like normal functions if its called with one or more values that are not known during compilation.
`std::pow` is not a constexpr function. it can't be used say to init an array.
In C++11 `constexpr` functions may contain no more than a single executable statement: a `return`.
`constexpr` functions are limited to taking and returning *literal* types. i.e. values known during compile-time.
User-defined data types can also have `constexpr` constructors and member functions. `constexpr` declarations of such types will be constructed at compile time using the `constexpr` constructors.
warning: If you declare an object or function `constexpr` clients may use it in such contexts, but later when you decide it was a mistake and remove it, client code might not compile.

#### 16: Make const member functions thread safe
use `mutable` for declaring variables inside `const` functions to be able to modify them.
but having such variables inside `const` member functions might lead to different threads trying to modify them at the same time leading to a race condition.
adding a mutex to a class makes it unable to be copied or moved, since `std::mutex` cannot be copied or moved.
for simply counting `std::atomic` is a better light weight fit than `std::mutex`.
`std::atomic` could be expensive though, since multiple thread would read the same value, perform calculations and try to set it.  
for setting single variables `std::atomic` is correct. For multiple variables `std::mutex` is unavoidable.  

#### 17: Understand special member function generation
 C++98 generates four functions: default constructor, destructor, copy constructor and  copy assignment operator.  
 default constructor is generated only if the class declares no constructors at all.  
 generated special functions are public and inline and non-virtual unless the function is a destructor in a derived class inheriting from a base class with a virtual destructor. in that case the compiler generated destructor if also virtual.  
C++11 adds two more: the move constructor and the move assignment operator.
````
class Widget
{
  Widget (Widget&& rhs); // move constructor
  Widget& operator=(Widget&& rhs); // move assignment operator
}
````

most c++98 legacy classes don't support move construction and will be copy constructed in the background. move operation is used only if it supports, else copy operation is used.
generation of move functions is not independent. move construction and move assignment are tied together. declaring one by the user will prevent the other from being auto-generated.
move operations won't be generated for any class that explicitly declares a copy operation. vice versa is also true. declaring a move operation prevents generation of copy operations.
move operations are generated for classes only if these are true:
1. No copy operations are declared
2. no move operations are declared
3. no destructor is declared in the class.

If you want to declare a custom destructor but can get away with using the default copy and assign behaviour, use the keyword `=default`. polymorphic base classes are classes that define interfaces through which derived base class objects re manipulated. they normally have virtual destructors.
copying a `std::map` is orders of magnitude slower than moving it.
adding `final` to a virtual function prevents the function from being overridden in derived classes. `final` if used in a class will prevent it from being derived.


#### Smart pointers
Things wrong with raw pointers:
1. declaration doesn't reveal whether it's an array or something else
2. declaration doesn't reveal whether the object should be destroyed, whether the pointer owns the objects
3. doesn't reveal mechanism of deletion. `delete` or other mechanism?
4. if `delete` can be used, doesn't reveal whether `delete` or `delete[]` should be used.
5. difficult to ensure that destruction is performed only once in code path.
6. no way to tell if pointer dangles

*Smart pointers* are wrappers around raw pointers. There are four smart pointers in c++11: `std::auto_ptr`, `std::unique_ptr`, `std::shared_ptr` and `std::weak_ptr`.  
`std::auto_ptr` is a deprecated leftover from c++98. `std::unique_ptr` does everything `std::auto_ptr` does plus more.

#### 18: Use `std::unique_ptr` for exclusive ownership resource management
It embodies *exclusive ownership* semantics. It always owns what it points to. Copying isn't allowed. It is a move-only type. Upon destruction, it destroys its resource.
during constcution `std::unique_ptr` objects can be configured to use *custom deleters*: arbitrary functions or function objects, including those from lambda expressions to be invoked when it's time for their resources to be destroyed.  
a raw pointer cannot be assigned to a smart points. `reset` is used to assign a raw ptr to a smart pointer.  
Using custom deleters as lambda expressions means size increment of `std::unique_ptr` is from  one word to two. these lamdba expressions must be stateless, captureless.  
`std::unique_ptr`  comes in two forms, one for individual objects `std::unique_ptr<T>` and one for arrays `std::unique_ptr<T[]>`  
`std::unique_ptr` can be easily converted into a `std::shared_ptr`.  

#### 19: Use `std::shared_ptr` for shared-ownership resource management
`std::shared_ptr` tries to implement something similar to garbage collection but with predictable timing of destructors.  
The last `std::shared_ptr` that was pointing to an object destroys the object when it is destroyed or when it points to something else. It refers the objects reference count.  `std::shared_ptr` is twice the size of a raw pointer. Memory for the reference count must be dynamically allocated. Increments and decrements of the reference count must be atomic. The type of the deleter for `std::shared_ptr` is not part of the type of the pointer itself. The reference count is part of a larger data structure known as the *control block*. There is a control block for each object managed by `std::shared_ptr`.  
A control block is created under the following conditions:
1. a `std::make_shared` always creates a control block.
2. whenever a `std::shared_ptr` is constructed from a unique-ownership pointer like `std::unique_ptr` and `std::auto_ptr`
3. when `std::shared_ptr` constructor is called witha a raw pointer.

constructing two `std::shared_ptr`s from a single raw pointer leads to undefined behaviour.
`std::enable_shared_from_this` if you want a class managed by `std::shared_ptr`s to be able to safely create a `std::shared_ptr` from a `this` pointer.
*Curiously named recurring template pattern*: when a derived class inherits from a base class that is templatized on the type of the derived class!

#### 20: Use `std::weak_ptr` for `std::shared_ptr` like pointers that can dangle.
`std::shared_ptr` cannot know if the object it points to has been destroyed. In that case, the pointer dangles. `std::weak_ptr` can track when it dangles. It can't be dereferenced or tested for nullness. It is an augmentation of `std::shared_ptr`.  
`std::weak_ptr` does not affect the reference count. `std::shared_ptr` is used to initialize a `std::weak_ptr`.  
`std::shared_ptr` can be set to nullptr. To check if the `std::weak_ptr` points to a destroyed object, check using `std::weak_ptr.expired()`  
In the time between the check to `expired()` and trying to dereference the pointer, some other thread may change the object using the `std::shared_ptr`. To prevent this race condition, use `std::weak_ptr.lock()`, which returns an `std::shared_ptr`. It is null if the object has expired.
If a `std::weak_ptr` which points to an expired object is used to initialize a `std::shared_ptr` an exception will be thrown.  
An example usage of `std::weak_ptr` is a factory function which returns `std::shared_ptr` for objects and uses `std::weak_ptr` for it's internal cache. when the caller is done using the returned object, it can be destroyed and the weak pointers used in the internal cache can detect when they dangle. `std::weak_ptr`s can detect when they dangle only when the object is also managed by a `std::shared_ptr`.  
Another usage scenario: *Observer Design Pattern* has two components: subjects and observers. In most implementations subjects hold a pointer to the observer to issue state change notifications. The subjects can hold a list of `std::weak_ptr`s for observers so that they can detect if the observers are destroyed if the pointers dangle.
`std::shared_ptr` cycles: When two objects have a shared smart pointer to each other but none of the other program data structures have a pointer to them, these objects cannot be accessed or their memory reclaimed because of the non-zero reference count!  
`std::weak_ptr`s can be used to break this cycle.
Other uses cases are in non-hierarchical data structures. (In hierarchical data structures, parent can point to child using `std::unique_ptr` and child to parent using raw pointers, since when the parent is destroyed the child will have to be destroyed too and the child will never have a dangling parent pointer).
`std::weak_ptr`s manipluate a second reference count that doesn't affect the actual reference count of the object pointed to by `std::shared_ptr`.  

#### 21 Prefer `std::make_unique` and `std::make_shared` to direct use of `new`

it was introduced in c++14

basic version for c++11 use:
````
template <typename T, typename... Ts>

std::unique_ptr<T> make_unique(Ts&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
````

`std::make_unique` and `std::make_shared` and `std::allocate_shared` are the three make functions. They return a smart pointer after calling the objects constructor.  
Code duplication increases compile times, leads to bloated object code and makes code base difficult to work with.  
`std::make_shared` allocates one chunk of memory for both the object constructed and the control block.  
There are circumstances where the make functions should not be used: (1) when using a custom deleter, (2) the perfect-forwarding construction in the make functions uses parantheses (calling the non-`std::initializer_list` constructor). Braced initializer cannot be used, as they can't be perfect-forwarded.  
For `std::shared_ptr`'s make function, additional limitations are: custom operator new and operator delete cannot be used since they handle memory of size exactly `sizeof(object)`. The make function `std::allocate_shared` allocates additional mem for control block.  
A *weak count* is maintained in the control block in addition to the *reference count*. Even if the *reference count* is zero, if there are any `std::weak_ptr`s pointing to the block, the control block is not deallocated. Here the *weak count* is non-zero as long as one `std::weak_ptr` exists.  
Using functions within the constuction call, causes the compiler to generate out-of-order code that can cause memory to be leaked. It's better to create the smart pointer in a separate line and then pass in to a function as a parameter, rather than constructing the smart pointer in the function call. The arg is passed as an rvalue (move) before and now the arg is passed as an lvalue (copy). `std::move` can be used to pass the smart pointer to the function though.  
````
std::shared_ptr<Widget> spw(new Widget, customDeleter);
processWidget(std::move(spw), computePriority();
````
#### Item 22: When using the pimpl idiom, define special member functions in the implementation file.
Pimpl idiom is where the members of a class are moved to a separate implementation class and the main class simply has a pointer to this implementation class. This speeds up  compilation since the clients of the main class don't have to include header files that the implementation is dependent on and the impl can change without the clients have to make changes.  
The allocation and deallocation of data members (the impl class) goes in the imple file of the main class.
````
Widget::Widget()
:
pImpl(new Impl) {}
````
Instead of using raw pointers, smart pointers can be used to handle impl allocation and deallocation.
````
Widget::Widget()
: pImpl(std::make_unique<Impl>()) {}
````
The destructor is no longer needed. There is a problem with the default deleter. If the compiler sees that it is using the default deleter on a raw pointer to an incomplete type, then it will `static_assert`. So a destructor must be declared before the impl type has been defined.   

#### Chapter 5 Rvalue reference, move semantics and perfect forwarding

_Move semantics_: Gives option to replace expensive copy operations with less expensive move operations.  
_Perfect forwarding_: makes it possible to write function templates that take arbitrary arguments.  

#### 23 Understand `std::move` and `std::forward`
Neither of them do anything at runtime, they both don't generate any executable code. They produce casts.  
`std::move` unconditionally casts to an *rvalue*
```
template <typename T>
typename remove_reference<T>::type&& move (T&& param)
{
    using ReturnType = typename remove_reference<T>::type&&;

    return static_cast<ReturnType>(param);
}

```

`std::move` returns an rvalue reference.
Don't declare objects const if you want to move from them. The copy constuctor will be called for const types, and not the non-const rvalue move constructor.  
`std::forward` is a conditional cast.
function parameters are lvalues. Whether an argument was initialized as an lvalue or an rvalue is encoded in the template type T and `std::forward` can read that encoded information to know to conditionally cast to the required lvalue or rvalue type.

#### 24: Distinguish universal references from rvalue references.
To declare an rvalue reference to some type `T`, we write `T&&`.  
It doesn't always work that way!
rvalue references bind only to rvalues.  
`T&&` are actually *universal references*, can behave as rvalue references or lvalue references `T&`.  
They can bind to const or non-const objects, to volatile or non-volatile objects and even to non-const and volatile objects.  
If `T&&` appears without type deduction scenario, then it is a rvalue reference.  
```
void f(Widget&& param);     // no type deduction
auto&& var2 = var1;         // type deduction
Widget&& var1 = Widget();   // no type deduction. var1 is a rvalue reference
```
If the initializer of a universal reference is an rvalue then it is an rvalue ref, if it is an lvalue, then it is an lvalue reference.  

#### 25: Use std::move on rvalue references, std::forward on universal references
Rvalue references bind only to objects that are candidates for moving.
```
class Widget
{
    Widget(Widget&& rhs); // rhs refers to an object eligible for moving.
};
```
A *universal reference* on the other hand might be bound to an rvalue or lvalue reference.
Rvalue references should be unconditionally cast to rvalue when forwarding them to other functions. Universal references should be conditionally cast to rvalues when forwarding them.
*universal  references* should be cast to rvalues only if they were initialized with an rvalue.
Passing a variable assuming it will be copied while internally the function does a `std::move` can lead to the passed in variable ending up with an unspecified value.
Replacing func that takes universal reference with two overloaded funcs that take lvalue and rvalue references incur runtime costs in some cases.  
Poor scalability: funcs that take arg pack will need o(2^n) overloads.
```
Matrix operator+ (Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return std::move(lhs);
}

```
is more efficient than
```
Matrix operator+ (Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return lhs;
}

```
since in the first the move constructor will be used but in the second the value will be copied.
*Return value optimization (RVO)*: local variable is constructed in the memory of the return value to avoid copying.
RVO can be used only if the return value is a local variable.  
Never apply `std::move` or `std::Forward` for local values that are returned by value, since they are eligible for RVO.

#### 26: Avoid  overloading on universal references.

In function arguments, lvalues passed are copied.
In func overloading, an exact match beats a match that requires a promotion, such as a short to an  int.
The interaction among perfect-forwarding constructors and compiler generated copy and move operations is tricky.  
Overloading on universal reference parameters should be avoided.  

#### 27: Familiarize yourself with alternatives to overloading and universal references

Abandon overloading: use different func names for overloaded funcs. won't work for constructors  
Pass by `const T&`: lvalue ref to const. It's inefficient (no compiler optimizations?, no move?)  
Pass by Value: simple and explicit  
Use Tag dispatch: if a universal reference parameter is part of a param list containing other types, then the match for other types takes priority. These tags are extra params which serve no runtime purpose. The compiler should optimize them away. See *template metaprogramming*.  

`std::enable_if` forces compilers to behave as if a particular template didn't exist.
```
class Person
{
    public:
    template < typename T,
               typename = typename std::enable_if< !std::is_base_of< Person,
                                                                     typename std::decay<T>::type
                                                                   >::value
                                                  >::type
              >
    explicit Person(T&& n);

}

```
Things to remember:
1. Alternatives to the combination of universal references and overloading include the use of distinct function names, passing parameters by lvalue-reference-to-const, passing parameters by value, and using tag dispatch.
2. Constraining templates via std::enable_if permits the use of universal references and overloading together, but it controls the conditions under which compilers may use the universal reference overloads.
3. Universal reference parameters often have efficiency advantages, but they typically have usability disadvantages.

#### 28: Understand reference collapsing
For this template func, the deduced parameter T will encode whether the argument passed to param was an lvalue or an rvalue.
```
template <typename T>
void func(T&& param);

```
References to references are illegal in C++
`auto& & rx = x;` won't compile.  
```
template <typename T>
void func(T&& param);
func(w);
```
becomes
```
void func(Widget& && param);
```
and ultimately becomes
```
void func(Widget& param);
```
due to reference collapsing by the compiler.  
Four possible reference-reference combinations with lvalue and rvalue references.  
Rule: *If either reference is an lvalue reference, the result is an lvalue reference. Otherwise (i.e., if both are rvalue references) the result is an rvalue reference.*
Example `Widget&` is an lvalue and `Widget`is an rvalue. `std::forward` converts lvalue to lvalue ref and rvalue to rvalue reference.

Reference collapsing occurs in four contexts:
1. Template Initialization
2. Type Generation for auto variables
3. Typedefs and alias declarations
4. decltype

#### 29: Assume move operations are not present, not cheap and not used.
`std::array` stores the data directly in the object itself. Move operation is not merely just copying the pointer to the heap memory for the data like in say `std::vector`. Each element of the array is individually copied or moved.  
`std::string` offers constant time moves and linear time copies. Unless *small string optimization* is used for strings with smaller than 15 characters. Here the string is stored in a buffer within the object.   
Move no good:
1. No move operations - copying is used.
2. Not faster.
3. Not usable - move operations require `noexcept`
4. Source not lvalue - only rvalues maybe used for move sources.


#### 30: Familiarize yourself with perfect forwarding failure cases.
*forwarding* is when one func passes what it was passed exactly to the next func. This rules out by-value parameters because they're copies of what the original caller passed on. Pointer parameters are also ruled out. This only deals with parameters that are references.  
*Perfect forwarding* means we also forward some salient characteristics: types, whether lvalue or rvalue, whether const or volatile.  
Only universal refs encode info about the lvalueness or rvalueness of the args.  

A good forwarding template func should be a *variadic* template - that takes any number of arguments.  
Kinds of arguments that can be perfect-forwarded:
1. Braced initializers
2. 0 or NULL as null pointers
3. Declaration-only integral static const and constexpr data members
4. Overloaded function names and template Names
5. Bitfields

### Chapter 6 Lambda Expressions
*lambda expresssion* is just the code. A *closure* is the runtime object created by a lambda. A *closure class* is a class from which a closure is instantiated. Each lambda causes compilers to generate a unique closure class.

#### 31: Avoid default capture modes.
two default capture modes: by-ref and by-value  
default capture mode is defined within the `[]`. `[&]` capture by ref. `[=]` capture by value.  
compilers replace any non-static member variable referred to with a version using *this* pointer.  
static variables can't be captured.  

#### 32: Use init capture to move objects into closures.
C++11 offers no way to capture a _move-only object_ int a closure.  
C++14 offers direct support for moving objects into closures.  
*Init capture* is a mechanism to perform any kind of captures, except a default capture mode and allows us to specify:
1. name of a data member in the closure class
2. expression initializing the data member

`[closureMember = initExpr]`. Scope of "closureMember" is that of the closure class. Scope of "initExpr" is that of where the lamda was defined.  
Another name is *generalized lambda capture*  
Use `std::bind` to emulate this init capture in c++11.

"A lambda is just a way to cause a class and a class instance to be generated"  
```
auto func = std::bind(lambda_expr, std::move(data));
```
`std::bind` produces function objects. All data members in the closure is const.  

#### 33: use `decltype` on `auto&&` parameters to `std::forward` them.  
if an lvalue is passed in `decltype()` gives an lvalue ref, an rvalue ref in case rvalue is passed in.  
```
auto f=[auto&&... xs]
{
    return normalize(std::forward<decltype(xs)>(xs)...);
};
```

#### 34: Prefer lambdas to `std::bind`
lamdas are more readable
c++14 standard suffixes for time: seconds(s), milliseconds (ms), hour (h) found in `std::literals` namespace  
template type argument for the standard operator templates can be omitted (`std::plus<>()`)   
compilers are less likely to inline func calls through func pointers  
all objects passed to bind objects are passed by reference
std::bind is justified in two constrained situations:
1. move capture
2. polymorphic function objects


### Chapter 7. The Concurrency API

#### 35: Prefer task-based programming to thread-based.  
pass func to `std::async`  
no straight forward way to get return value in thread-based approach.  
types of threads:
1. hardware threads
2. software threads (os or system threads) - os schedules these on hardware threads
3. std::threads: objects in a c++ process that act as handles to underlying software threads.  

`std::system_error` is thrown when you try to create more than the system can provide in software threads, even if the func is `noexcept`.  
`std::async` shifts the thread management responsibility to the implementer of the c++ standard library.  
`std::sync` permits the scheduler to arrange for the specified function to be run on the thread requesting the func's result.   
`std::launch::async` launch policy will ensure that the func will run on a different thread.  
state of the art schedulers use thread pools and work-stealing algorithms  
cases where threads might have to be used directly:
1. need access to the API of the underlying thread implementation
2. need to optimize thread usage - app execution profile is known and will be the only process on the machine with fixed hw characteristics.
3. threading tech beyond c++ Concurrency API is needed - eg. thread pools.

#### 36: Specify `std::launch::async` if asynchronicity is essential.
using `std::async` is just requesting a async *launch policy*
two policies in the enum:
1. std::launch::async
2. std::launch::deferred
issues with default launch policy:
1. not possible to predict whether f will run concurrently with t
2. not possible to predict whether f runs on a thread different from the thread invoking get or wait or fut
3. it may not be possible to predict whether f runs at all.

passing `std::launch::async` guarantees async execution.
```
auto fut = std::async(std::launch::async, func_to_be_executed);
```

#### 37: Make `std::threads` joinable on all paths.

two states of thread object: *joinable* or *unjoinable*  
a thread that's finished, blocked or waiting is *joinable*  

*unjoinable* threads:
1. default constructed `std::threads`
2. `std::thread` objects that have been moved from
3. `std::thread`s that have been *joined*
4. `std::thread`s that have been *detached*

It is on us to ensure that if a `std::thread` object is used, it's made unjoinnable on every path out of the scope in which it's defined.

RAII objects: put common termination actions in the destructor.  

there is no support in c++11 for *interruptible threads*  

1. Make `std::thread`s unjoinable on all paths.
2. join-on-destruction can lead to difficult-to-debug performance anomalies
3. detach-on-destruction can lead to difficult to debug undefined behavior
4. declare `std::thread` objects last in lists of data members.

#### 38: Be aware of varying thread handle destructor behavior
both `std::thread` objects and future object can be thought of as *handles* to system threads.
the callee (usually running asynchronously) writes the result of its computation into the communications channel (typically via a `std::promise`) and the caller reads that result using a `future`  
the result of the callee is actually stored in the *shared state*.
