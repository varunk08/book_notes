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

#### Item 1

In a scenario like this,

````
template<typename T>
void f(ParamType param);

f(expr)
````

the type `T` deduced is dependent on both `expr` and the form of `ParamType`.  
A *univeral reference*'s declared type is `T&&`  

compile time array size func using template function which takes in a reference to an array:
````
template <typename T>
constexpr size_t arraySize(T(&)[N]) noexcept
{
    return N;
}
````