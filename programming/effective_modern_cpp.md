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
