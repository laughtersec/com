---
publish: true
title: COM Overview
created: 2025-11-09T04:48:20.814Z
modified: 2026-08-24T17:22:29.756Z
tags:
  - theory
---

# Introduction

- Basic protocol for object communication
- OLE is just one of the services logically placed above COM and that uses its facilities.
- COM, and therefore OLE, is centered around the idea of an "interface", where an interface represents a contract between an object and its user. It is the only way to interact with a COM object.
  - A COM object is defined in terms of the individual _interfaces_ that it supports.
  - Interfaces, in programmatic terms, are arrays of pointers to functions, which is C-speak for a list of entry points to defined routines.
  - Each interface is identified by a unique identifier called an interface identifier (IID), similar to the way in which each COM object is identified by a unique CLSID.
  - Like CLSIDs, IIDs are also GUIDs, which means that they are created like any other GUID using the COM API `CoCreateGuid`.
  -
- In its role as the glue between objects, OLE implements a wide variety of interfaces and defines a whole lot more.
- Implementation of an interface means that OLE supplies code that performs the actions expected of each of the member functions in the interface.

![[../excalidraw/com_objects_and_interfaces]]

In this example, GameEnvironment is your typical COM object, with its underlying interfaces. But in the following example, you have your typical public object, not using COM.

![[../excalidraw/normal_objects]]

Every interface inside a COM object is also uniquely identifiable. And a separate pointer is required to access each interface. Once an interface is exposed for client usage, it _must never be changed_ (they are **immutable**).

## Interface navigation

To support interface navigation, every interface must implement a special function named `QueryInterface` which is part of the `IUnknown` interface (this interface is implemented in every COM object).

[IUnknown::QueryInterface(REFIID,void) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/unknwn/nf-unknwn-iunknown-queryinterface\(refiid_void\))

It is the standard way to make initial contact with a COM object to later allow interaction with the rest of the interfaces of the COM object.

Typically done like this:

```cpp
HRESULT hr;
IUnknown *pIUnknown = NULL;
IDesiredInterface *pIDesiredInterface = NULL;
//  <...> Initialize the COM library

hr = CoCreateInstance(CLSID_DesiredComClass, ..., ..., IID_IUnknown, (LPVOID*)&pIUnknown); 
//  Some error handling in between
hr = pIUnknown->QueryInterface(IID_IDesiredInterface, (LPVOID*)&pIDesiredInterface);
//  Now we can use the functions of that interface using the obtained pointer to the specific interface we wanted
pIDesiredInterface->desiredFunction();
//  Whatever comes after
```

==However, the documentation does not state anywhere that it is mandatory to use the== `IUnknown` ==interface when calling== `CoCreateInstance` ==although it is a conventional practice.== Because every interface in turn calls the `IUnknown` interface (?).

## Lifetime management

Its in the name! Lifetime being the active instance of an object. Everything that lives has to die at some point. The management of the cycle of creation and destruction of objects is lifetime management. How is it done in COM? Using reference counters. It is incremented when an object is created, and decremented when it is destroyed.

The `IUnknown` interface has two functions `AddRef` and `Release` for this purpose. As we know, every COM object requires implementing `IUnknown`. So naturally, when we follow the steps to [[#Interface navigation|navigate interfaces]], the reference count will be incremented.

```cpp
class CSomeObject : IUnknown
{
	private:
		ULONG m_cRef;  //  Reference counting variable
};
```

> A better example can be seen [[in-process-servers#Implementing Interface Functions|here]].

> A better example can be seen in [in-process-servers](concepts/in-process-servers.md)

## Object Versioning and Evolution

## COM Servers

Anything that provides accessibility to, or is a medium to various COM objects.

### In-process servers

- The class for each COM object is implemented in a binary code module (DLL or EXE) called a COM server. The DLL implementations are loaded directly into the client process' address space, and are referred to as in-process servers.

> I'm sure you know what a DLL is and how it works with a process. If you don't, please stop reading.

- Since in-process servers don't own their own resources, they cannot maintain global resources that are accessible by multiple clients.

### Out-of-process servers (DLLHost.exe)

- COM servers can be created as standalone EXEs, in which case they maintain an address space of their own.
- COM servers created as EXEs are commonly referred to as out-of-process servers.
- Since EXEs maintain their own address space, out-of-process servers are also capable of owning their own resources, which may be shared among their clients.
- In most cases its hosted locally, so its also called a local server.

Any COM server running in or out of process other than the client's machine is referred to as a remote server, and is said to serve the client(s) remote objects. In the case where a remote server is an in-process server, COM automatically creates a separate surrogate process and loads the in-process server into its address space.

The context switching between processes is what creates a demand of resources and out-of-process and remote processes are responsible for this.

## COM System Services

Also referred to as the COM library. But its called "COM System Services" because of its integration with Service Control Manager, which is responsible for its activation and helps locating COM objects and their respective interfaces and type libraries. It is also responsible for facilitating communication between COM clients and servers. [Here's more information on how it facilitates it](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/98c08086-d94a-443c-b7c6-167e82652885).

[CoInitialize function (objbase.h) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/objbase/nf-objbase-coinitialize)

### Implementation Locator Service

- "Implementation" implying the COM server.
- "Locator" implying the mechanism to search for COM servers via GUIDs (in the registry, GUIDs are mapped to their respective COM servers).
- "Service" implying the "Locator" runs as a service on Windows.

A system-wide database called the _system registry_ is maintained by COM. Essentially, it is a lookup table mapping CLSIDs to COM server filenames.
CLSIDs, in a registry, are stored under `HKEY_CLASSES_ROOT\CLSID`. Under this, you will find the GUIDs for every class.

The implementation locator service is implemented in the form of **Service Control Manager (SCM)**.

- It locates the appropriate server for a COM object identified by a client supplied CLSID.
- Launches the COM server.

## LPC and RPC Mechanism

This level of "indirection" is all that is required for COM to transparently provide the programmer with a single, location-independent programming model.

> When you obtain an interface pointer, you are actually receiving a pointer to a pointer that is pointing to a VTBL of function pointers. An interface is actually a pointer to a VTBL of function pointers.

![interfaces\_and\_vtables](excalidraw/interfaces_and_vtables.md)

Obviously, the pointers will only be able to access information within a single process space. If a server is out-of-process, client interface pointers are not allowed to access information in the server's process space (again, obviously). To solve this problem, COM relies on a special piece of in-process software called a proxy. In the case where you ever receive an interface pointer to an out-of-process object, you really are just receiving a pointer to a proxy. The proxy exists to take the place of the object and to forward any client requests to another special piece of software called a _stub_.

Naturally, the client's interface pointer can access the proxy and it is the _said_ object _for_ the client. But how does it forward and receive requests to the original? This is done via _marshaling_.

Like the proxy, the stub is also in-process; however, the stub is located in the server's process space. The stub receives requests from the proxy and _unmarshals_ any parameters before actually invoking the method of the interface. To the object, the stub _is_ the client. The object (let's say its on a remote computer) passes any return data to the stub, which forwards it to the proxy (via RPC if remotely), which passes it to the client. Middlemen after middlemen :)

> Important to remember:
> The first point of contact, originating from a COM object's proxy on its remote/local COM object via RPC/LPC, is a stub.
> The stub, would obviously have the respective pointer to the interface, for which it is unmarshaling the parameters for its internal methods received via RPC.
> The proxy, would obviously know which stub (located remotely or locally) to communicate with via RPC/LPC.

When the client is accessing a local server, and the proxy and stub are located on the same machine, they communicate via Local Procedure Calls (LPCs). LPC is a form of inter-process communication specifically designed for one process to invoke the methods of a different process. LPCs work fine as long as the proxy and stub are located on the same computer.

> RPC is the basis of DCOM.

![[../Excalidraw/com_proxies_and_stubs|1000x800]]

## Type Libraries

^90083c

A type library is essentially a language-neutral description of COM elements. Type libraries are typically used during **automation** to perform parameter type checking. Automation is the process of manipulating an application's COM objects from outside the application using special automation interfaces.
