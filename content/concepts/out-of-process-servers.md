---
publish: true
title: Out-of-Process Servers
created: 2026-01-05T19:56:18.218Z
modified: 2026-08-24T15:29:07.526Z
tags:
  - programming-preliminaries
---

Despite the performance overhead for each call made, here are some advantages of an out-of-process server:

- **Process isolation**: The effects of one process are free from the other. Incase a call fails and causes a crash, a client in this case would not be affected if the server did, unlike in-process servers where a failed call could crash the client itself.
- **Better security**: Apparently the resources of each process can be guarded against unauthorized usage by "rogue" processes (?). ==The same level of security is not possible with an in-process server, because the client and server share the same process.==
- **The ability to maintain shared global resources**: Since EXEs maintain ownership of their own resources, an out-of-process server is easily capable of allocating global resources and sharing them among its various clients. The same is not true for DLLs, because an independent copy of the DLL, and thus an independent copy of the DLL's resources, is loaded into each client's address space (which only makes sense).

Unlike in-process servers, out-of-process servers exists outside the address space.

Lets create our first out-of-process server "UserInfoHandler".

Unlike the in-process server "UserInfo", we won't be needing dllmain.cpp here. The remaining files are the same required to create an out-of-process server.

The following are properties maintained and exposed by it:

| Property Name | Data Type     |
| ------------- | ------------- |
| Age           | short         |
| Name          | LPSTR         |
| Sex           | unsigned char |

The following are its interfaces with their respective functions:

| Interface Name | Function                                                                                                                                                                                                                                  | Purpose                                                                                      |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| ICopyInfo      | `CopyName(IUserInfo *pDest, IUserInfo *pSrc)`<br>`CopyAge(IUserInfo *pDest, IUserInfo *pSrc)`<br>`CopySex(IuserInfo *pDest, IUserInfo *pSrc)`<br>`CopyAll(IUserInfo *pDest, IUserInfo *pSrc)`                                             | The `ICopyInfo` interface is used to copy information from one `UserInfo` object to another. |
| IReverseInfo   | `ReverseName(IUserInfo *pIUserInfo)`<br>`ReverseAge(IUserInfo *pIUserInfo)`<br>`ReverseSex(IUserInfo *pIUserInfo)`<br>`ReverseAll(IUserInfo *pIUserInfo)`                                                                                 | The `IReverseInfo` interface is used to reverse a `UserInfo` object's information.           |
| ISwapInfo      | `SwapName(IUserInfo *pIUserInfo, IUserInfo *pIUserInfo)`<br>`SwapAge(IUserInfo *pIUserInfo, IUserInfo *pIUserInfo)`<br>`SwapSex(IUserInfo *pIUserInfo, IUserInfo *pIUserInfo)`<br>`SwapAll(IUserInfo *pIUserInfo, IUserInfo *pIUserInfo)` | The `ISwapInfo` interface is used to exchange information between two `UserInfo` objects.    |

Like in-process servers, out-of-process servers are also responsible for:

- Allocating GUIDs for each supported object and interface
- Defining the interfaces supported by each object.
- Implementing the functions defined by each interface.
- Implementing a class factory capable of creating each supported object.
- Registering class information for each supported object.
- Exposing a class factory for each supported object.
- Unloading (destroying) itself when appropriate.

## Registering the server

Unlike the in-process server from earlier, the `regsvr32.exe` utility cannot automatically register the exe. Since its an exe, it can register itself! We can run the server using an administrator prompt and pass it the appropriate arguments

```cmd
.\UserInfoHandler.exe /RegServer
```

The server can now be found in the registry.

## Marshaling

Since the `UserInfoHandler` server maintains a process space apart from its clients, we'll need to create a proxy/stub pair in order for the client and server to communicate across process boundaries.

Whenever a client invokes an interface function of an out-of-process server, the proxy - which runs in the client's address space - packages the required parameters and communicates them to the corresponding stub running in the server's address space. The stub unpackages the parameters and executes the function call, then packages any return values and transmits them back to the proxy. The proxy then unpackages the return values and forwards them to the actual client. When a proxy and stub are located on the same physical machine, they communicate using Local Procedure Calls (LPC). However, when they are located on separate machines, proxies and stubs communicate using Remote Procedure Calls (RPCs). The entire process, including the determination of when to use LPCs as opposed to RPCs, is totally transparent to the developer.

Because the proxy and stub are contained in the same module, we only have to build one proxy/stub pair. To build a proxy/stub pair for the `UserInfoHandler` server, we must first create a separate DLL project. Add into this project the following files: `UserInfoHandler_i.c`, `UserInfoHandler_i.h`, `UserInfoHandler_p.c` and `dlldata.c`. These files are generated by the MIDL compiler when it processed our `UserInfoHandler.idl`file. We'll shift our focus to `UserInfoHandler_p.c` and `dlldata.c`. `UserInfoHandler_p.c` actually contains all the code necessary to build a proxy and a stub for the interfaces defined in `UserInfoHandler.idl`. All we need to do now is add `dlldata.c` to the project.

[Building and Registering a Proxy DLL - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/com/building-and-registering-a-proxy-dll)

---

# Source

[Full source is provided here](https://github.com/laughtersec/com-servers-and-clients)

After a bit of searching, I found a sample project from Microsoft itself: https://github.com/microsoft/component-object-model-sample
