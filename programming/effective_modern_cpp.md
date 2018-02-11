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

### Chapter 1 Deducing types
`auto` and `decltype`  

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