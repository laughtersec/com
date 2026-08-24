---
publish: true
title: IDL
created: 2025-12-06T05:19:12.649Z
modified: 2026-08-24T15:28:38.780Z
tags:
  - theory
---

# The glue between RPC and programming-language-barriers

COM IDL (Interface Description Language) is based on the Open Software Foundation Distributed Computing Environment Remote Procedure Call (OSF DCE RPC) IDL.

DCE IDL allows **remote procedure calls** to be described in a language-neutral manner that also enables an IDL compiler to generate networking code that transparently remotes the described operations over a variety of network transports. COM IDL simply adds a few COM-specific extensions to DCE IDL to support the object-oriented nature of COM (e.g. inheritance, polymorphism).

Not coincidentally, when COM objects are accessed across execution context or machine boundaries, all client-object communications use MS-RPC (an implementation of DCE RPC that is part of Windows NT and Windows 95) as the underlying transport.

The Win32 SDK includes an IDL compiler called MIDL.exe that parses COM IDL files and generates several artifacts.

![[../Excalidraw/IDL_generator|1000x800]]
MIDL.exe generates C/C++ compatible header files that contain the abstract base class definitions that correspond to the interfaces that are defined in the original IDL file. The fact that MIDL automatically generates the C/C++ header file implies that no COM interfaces should be defined manually in C++.  Having a single point of definition avoids having multiple incompatible versions of an interface definition that can fall out of sync over time. MIDL also generates source code that allows the interface to be used across thread, process, and host boundaries. Finally, MIDL can also generate a binary file that allows other COM-aware environments to produce language mappings for the interfaces defined in the original IDL file. This binary file is called a [[com-overview#^90083c|type library]] and contains tokenized IDL in an efficiently parsed form.

Type libraries are typically distributed as part of the implementation's executable and allow languages like Visual Basic, Java or Object Pascal to use the interfaces that are exposed by the implementation.

## How an "Interface" should be seen

There are two perspectives to looking at an interface:

- Logical: Discussions of an interface's methods and the operations they perform focus on the logical aspect of an interface.
- Physical: Discussions of memory, stack frames, network packets, or other runtime phenomena usually refer to the physical aspect of an interface.

Some physical aspects of an interface are directly derivable from a logical description (e.g., vtbl ordering, order of parameters on the stack (???calling conventions???)). Other physical aspects (e.g., array boundaries, network representations of complex data types) require additional qualification.

## Example syntax

IDL allows interface designers to work mainly in the logical realm using C-style syntax. However, IDL also allows interface designers to precisely specify any aspects of an interface that cannot be derived directly from its C-style logical description using annotations that are formally called attributes.

```c
[
	v1_enum, helpstring("This is a property!")
]
enum PROPERTY { SQFT, CEILING_HEIGHT, ROOM_COUNT };
```

The v1\_enum attribute applies to the enumeration definition of PROPERTY. This particular attribute informs the IDL compiler that the network representation for PROPERTY should be 32 bits, not 16 bits, which is the default. The helpstring attribute also applies to PROPERTY and injects the string "This is a property!" into the generated type library as documentation for the enumeration. If one ignores the attributes in an IDL file, the syntax is simply that of C. IDL supports structures, unions, arrays, enumerations, and typedefs with a syntax identical to that of their C counterparts.

When defining COM methods in IDL, one needs to indicate explicitly whether the caller or the callee will be writing or reading each method parameter. This is accomplished using the parameter attributes `[in]` and `[out]`:

```c
void Method1(
	[in] long arg1,
	[out] long *parg2,
	[in, out] long *parg3
)
```

For this IDL fragment, the caller is expected to pass a value to the object in `arg1` and in the location referred to by `parg3`. Upon completion, the object is expected to pass values back to the caller through the locations referred to by `parg2` and `parg3`. This means that for the following caller sequence:

```c
long arg2 = 20, arg3 = 30;
p->Method(10, &arg2, &arg3);
```

the object cannot count on receiving the actual value of 20 via parg2. If the object is running in the same execution context as the caller and both parties are implemented in C++, then `*parg2` will in fact contain the value of 20 on method entry. However, if the object is accessed from a different execution context or one part is implemented in a language that optimizes away the initialization of the out-only parameter, then the caller's initialization will be lost.

## How this affects Methods (implementation) and Results (their return)

Method results are one aspect of COM where the [[#How an "Interface" should be seen|logical and physical]] worlds diverge. ==Almost all COM methods physically return an error number of type HRESULT.== ==The use of a uniform result type allows COM's remoting architecture to overload the result of a method and also indicate communications errors simply by reserving a range of values for RPC errors.== HRESULTs are 32-bit integers that provide information to the caller's runtime environment about what type of error may have occurred (e.g., network errors, server failures).

For many COM-compatible implementation languages (e.g., Visual Basic, Java), these ==HRESULTs are intercepted by a supporting runtime or virtual machine and mapped to programmatic exceptions==.

## Intrinsic Data Types Supported by IDL

| Data Type           | Description                                                                                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| boolean             | Data item with either a TRUE or FALSE value                                                                                                                       |
| char                | 8-bit, signed data item                                                                                                                                           |
| double              | 64-bit IEEE floating-point number                                                                                                                                 |
| int                 | System-dependent signed integer                                                                                                                                   |
| float               | 32-bit IEEE floating-point number                                                                                                                                 |
| long                | 32-bit signed integer                                                                                                                                             |
| short               | 16-bit signed integer                                                                                                                                             |
| wchar\_t             | Unicode character accepted only for 32-bit type libraries                                                                                                         |
| BSTR                | Length-prefixed string                                                                                                                                            |
| CURRENCY            | 8-byte, fixed-point number                                                                                                                                        |
| DATE                | 64-bit, floating-point fractional number of days since December 30, 1899                                                                                          |
| DECIMAL             | 98-bit, unsigned binary integer scaled by a power of 10. Provides size and scale for a number (as in coordinates)                                                 |
| SCODE               | Built-in error type that corresponds to VT\_ERROR. An SCODE (used on 16-bit systems only) does not contain the additional error information provided by an HRESULT |
| VARIANT             | One of the variant data types (discussed in Automation objects)                                                                                                   |
| IDispatch \*         | Pointer to an `IDispatch` interface                                                                                                                               |
| IUnknown \*          | Pointer to an `IUnknown` interface                                                                                                                                |
| SAFEARRAY(TypeName) | TypeName is any of the above types. An array of these types.                                                                                                      |
| TypeName \*          | TypeName is any of the above types. A pointer to a type.                                                                                                          |
| void                | Allowed only as a function return type or in a parameter list to indicate no arguments                                                                            |
| HRESULT             | Return type used for reporting error information in interfaces.                                                                                                   |
| LPWSTR              | Unicode string accepted only for 32-bit type libraries                                                                                                            |
| LPSTR               | Zero-terminated string                                                                                                                                            |

## HRESULT Return Value Constants

| Constant      | Meaning                                       |
| ------------- | --------------------------------------------- |
| S\_OK          | Function completed and the result is TRUE     |
| S\_FALSE       | Function completed and the result is FALSE    |
| NOERROR       | Function completed with no return value       |
| E\_UNEXPECTED  | An unexpected error has occured               |
| E\_INVALIDARG  | One of the user-supplied arguments is invalid |
| E\_OUTOFMEMORY | Sufficient memory could not be allocated      |
| E\_NOINTERFACE | The requested interface is not supported      |

## HRESULT Macros

| Syntax                                                      | Description                                                                       |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------- |
| SUCCEEDED(HRESULT status)                                   | If the severity field of the HRESULT is 0, returns TRUE; otherwise, returns FALSE |
| FAILED(HRESULT status)                                      | If the severity field of the HRESULT is 1, returns TRUE; otherwise, returns FALSE |
| HRESULT\_CODE(HRESULT hr)                                    | Returns the error code field of the HRESULT                                       |
| HRESULT\_FACILITY(HRESULT hr)                                | Returns the facility field of the HRESULT                                         |
| HRESULT\_SEVERITY(HRESULT hr)                                | Returns the severity field of the HRESULT                                         |
| HRESULT MAKE\_HRESULT(SEVERITY sev, FACILITY fac, CODE code) | Creates a new HRESULT given a severity, a facility, and a status code             |

User-defined error codes should have a code value between 0x0200 and 0xFFFF, as values 0x0000 and 0x01FF are used by the COM-defined FACILITY\_ITF codes.

## Full Reference

[Microsoft Interface Definition Language - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/midl/midl-start-page)
