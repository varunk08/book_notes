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

A *descriptor heap* is an array of descriptors. You will need a separate descriptor heap for each type of descriptor.
*Super sampling* makes the back buffer and depth buffer 4x bigger than the screen resolution and then resolving/ down sampling. *multisampling* is cheaper. It computes the color at the pixel center of the sub-pixel and resolves the output pixel.

DXGI. *DirectX graphics infrastructure* is an API used along with D3D. It is a common API for multiple graphics APIs that require common functionality like swapchains, display adapter enumeration, formats etc. Provides,  
`IDXGIAdapter` for display adapters (software and hardware adapters).  
`IDXGIOutput` for display output (monitors).  

More info on DXGI:
[**DXGI overview:**](http://msdn.microsoft.com/en-us/library/windows/desktop/bb205075(v=vs.85).aspx)  
[**DXGI best practices:**](http://msdn.microsoft.com/en-us/library/windows/desktop/ee417025(v=vs.85).aspx)
[**DXGI 1.4 improvements:**]( http://msdn.microsoft.com/en-us/library/windows/desktop/mt427784%28v=vs.85%29.aspx)  

#### CPU/GPU interaction
