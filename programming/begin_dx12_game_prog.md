Book notes for  
### Beginning DirectX12 Game programming
#### by Frank D. Luna

#### Windows programming (aside)

Uses native win32 API. `WNDCLASS` structure used to create a window class.  
`CreateWindow` creates window. `ShowWindow` displays it.  
Ref book: *Programming Windows by Charles Petzold*  
Windows applications do not have direct access to the hardware. Cannot write to video memory directly.   
*Event driven programming model*: an app waits for something to happen. Windows sends a *message* to the app on event by adding it to the *message queue*. The app dispatches the message to the *window procedure* for a particular window that message is for.  
`DefWindowProc` is the default window procedure provided by windows.
*Unicode*: uses 16-bit encoding. For unicode in c++ we use *wide character* type `wchar_t`. C++ std library also provides `std::wstring`.   

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

`__uuidof(**(ppType))` evaluates to the COM interface ID of `(**(ppType))`  
Associated with a command list is a memory backing class called a  `ID3D12CommandAllocator`. As commands are recorded to the command list, they will actually be stored in the associated command allocator.  

You can create multiple command lists associated with the same allocator, but you cannot record at the same time. All command lists must be closed except the one whose commands we are going to record.  
Resetting the command list does not affect the commands in the command allocator because the associated command allocator still has the commands in memory that the command queue references.  
To reuse the command allocator memory for the next frame, `ID3D12CommandAllocator::Reset` is called. We must however make sure that a command allocator is not reset until we are sure that the **GPU has finished executing all the commands in the allocator**.  

A **fence** is used to ensure that the GPU has finished executing commands up to a point.  

**Resource Transitions**. Resources are in a default state when created. *Barriers* are used to prevent *resource hazards*.

**Multi-threading**. Multiple command lists can be built in parallel. Command lists, command allocators are *not free threaded* - each thread will get it's own command list and allocator. Command queue is free threaded. The maximum number of command lists that the app will use must be specified at initialization.  

**Initializing Direct3D**:
1. Create the `ID3D12Device` using the `D3D12CreateDevice`
2. create a fence object and query descriptor sizes
3. check 4x MSAA quality level support
4. create command queue, command list allocator and main command list
5. describe and create the swap chain
6. create the descriptor heaps the application requires
7. resize the back buffer and create a render target view to the back buffer
8. create the depth/stencil buffer and its associated depth/stencil view
9. set the viewport and scissor rectangles

A WARP (windows advanced rasterization platform) device is a software adapter.  
Command list needs to be in closed state before resetting.  

A handle to the first descriptor is obtained with the `ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart` method.
In-order to bind a resource to a pipeline, a *view* to a resource is created and it is bound to the pipeline stage.  

Pixels outside the scissor rectangle are culled - not rasterized to the back buffer. (example, not rendering in areas covered by UI elements). Scissor rects need to be reset whenever the command list is reset. Multiple scissor rects can't be specified for a single render target.  

**Timing and animation**
Performance timer: win32 provides `QueryPerformanceCounter` - gives time in counts
`QueryPerformanceFrequency` - returns counts per second  

`GameTimer` class is implemented by the author  

**The Demo Application Framework**
Clients are to derive from `D3DApp` class. All examples in the book derive from this common util class. It has "framework" methods that handle the common misc stuff like message handling and "non-framework" virtual methods that each example must implement.  
`ThrowIfFailed` is a macro that can print the code line where an error occured.  
