### Effective Modern C++ by Scott Meyers

There have been four official versions of C++, each named after the year in which the corresponding ISO Standard was adopted: C++98, C++03, C++11 and C++14.  

Foundation of `Move semantics` is distinguishing expressions that are `rvalues` and `lvalues`.  
`rvalues` indicate objects eligible for move operations, while `lvalues` generally don't.  
`rvalues` correspond to temporary objects returned from functions, while `lvalues` correspond to objects you can refer to, either by name or by following a pointer  or `lvalue` reference.  


Distinction between arguments and parameters of a function is important, because parameters are `lvalues` but the arguments with which they are initialized may be `rvalues` or `lvalues`.  

*Exception safety basic guarantee*: assuring callers that even if an exception is thrown, program invariants remain intact (no data structures are corrupted).  
*strong guarantee*: state of the program remains as it was prior to the call.  

Function objects created through lambda expressions are known as *closures*  
