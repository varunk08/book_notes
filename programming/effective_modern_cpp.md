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
