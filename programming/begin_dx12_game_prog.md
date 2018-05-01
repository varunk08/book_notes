Book notes for  
### Beginning DirectX12 Game programming
#### by Frank D. Luna


#### Chapter 4 Direct 3D initialization
D3d12 designed to significantly reduce CPU overload.  

*Component object model* (COM) allows directX to be programming language independent. COM objects are reference counted. We `Release` it rather than `delete`. `ComPtr` is like a smart pointer for COM objects. Three main ComPtr interfaces: `Get` (returns the underlying pointer), `GetAddressOf` (returns the address of the underlying pointer) and `Reset` (sets the ComPtr to nullptr). COM interfaces are prefixd with `I`.  

Swapping the roles of front and back buffer is called *presenting*. The front and back buffer form a *swap chain*. *Depth buffering* provides ability of order independent drawing.  
*Resource descriptors* add a level of indirection. It is so that GPU resources can remain generic chunks of memory and can be accessed from different parts of the pipeline. A *view* is synonymous with a *resource descriptor*.  

Types of descriptors:
1. CBV/SRV/UAV: const buffer, shader resource, unordered access
2. Sampler descriptors
3. RTV: render target
4. DSV: depth stencil
