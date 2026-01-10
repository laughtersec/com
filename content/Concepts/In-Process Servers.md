---
{"publish":true,"created":"2025-12-25T11:40:34.889+05:30","modified":"2026-01-10T09:41:59.532+05:30","published":"2026-01-10T09:41:59.532+05:30","tags":["programming-preliminaries"],"cssclasses":""}
---

> Please get [[Concepts/COM Overview\|an overview of COM]] before proceeding. 

Lets create our first in-process server "UserInfo". ==This is a hypothetical COM object== which is used to maintain information about an individual person (name, dob, sex etc.). Here, each piece of information that is maintained is exposed as a property of the UserInfo COM object.

| Property Name | Data Type     |
| ------------- | ------------- |
| Name          | LPSTR         |
| Age           | unsigned int  |
| Sex           | unsigned char |

The process can be broken down into these simple steps:
- A GUID will be allocated for use as CLSIDs, IIDs and LIBIDs
- Interfaces that would be exposed will be defined
- Then they will be implemented.
- An object class factory will be implemented.
- Finally, the in-process server will be registered with the system registry.

After creating the in-process server, we would use it via a COM client.

The process can be broken down into these simple steps:
- Load an in-process server.
- Initialize the COM library.
- Obtain initial and subsequent interface pointers
- Manipulate a COM object

Lets begin.

## Allocating GUIDs

`CoCreateGUID` is used even by applications like GUIDGEN.EXE AND UUIDGEN.EXE internally.

Here's its working

```shell
**********************************************************************
** Visual Studio 2022 Developer Command Prompt v17.14.17
** Copyright (c) 2025 Microsoft Corporation
**********************************************************************

C:\Program Files\Microsoft Visual Studio\2022\Community>guidgen

C:\Program Files\Microsoft Visual Studio\2022\Community>uuidgen
d7f24778-c032-4c67-88cc-3ee55458855a

C:\Program Files\Microsoft Visual Studio\2022\Community>uuidgen -n3
6db492b9-cd03-4511-a6bb-3ece8b3377f2
22648112-e14f-463c-8ff5-f7e50dfdf5d0
cf79f686-dc42-4986-97c5-5f0d0bdcdfa8
```

## Defining the object's interfaces

> Please understand [[Concepts/IDL]] before proceeding.

The work of defining objects and interfaces is done using IDL. We can define COM objects, interfaces and type libraries using it.

Here's the typical syntax:

```shell
[attributes] elementname typename {memberdescriptions};
```

The `[attributes]` section is used to define the element's characteristics. `elementname` is a keyword that indicates the type of element being defined (coclass, interface, library etc.). `typename` assigns a name to the element. The `{memberdescriptions}` section contains definitions for one or more additional elements contained within the element being defined.

Example:

```cpp
import "unknwn.idl";

//IID_IUserInfo
//These are the attributes of the IUserInfo interface
[
	object,
	uuid(22648112-e14f-463c-8ff5-f7e50dfdf5d0),
	helpstring("IUserInfo Interface.")
]
//Declaration of the IUserInfo interface
interface IUserInfo : IUnknown
{
	//List of function definitions for each method supported
	//by the interface
	//
	//[attributes] returntype [calling convention]
	//funcname(params);
	//
	[propget, helpstring("Sets or returns the age of the user.")]
	HRESULT Age([out, retval] short *nRetAge);
	[propput, helpstring("Sets or returns the age of the user.")]
	HRESULT Age([in] short nAge);
	[propget, helpstring("Sets or returns the name of the user.")]
	HRESULT Name([out, retval] LPSTR *lpszRetName);
	[propput, helpstring("Sets or returns the name of the user.")]
	HRESULT Name([in] LPSTR lpszName);
	[propget, helpstring("Sets or returns the sex of the user.")]
	HRESULT Sex([out, retval] unsigned char *byRetSex);
	[propput, helpstring("Sets or returns the sex of the user.")]
	HRESULT Sex([in] unsigned char bySex);
}
```

[2 of the attributes](https://learn.microsoft.com/en-us/windows/win32/midl/interface-header-attributes) are interface header attributes, and [one of them](https://learn.microsoft.com/en-us/windows/win32/midl/type-library-attributes) is a type library attribute.

Now obviously we don't need to *define* the functions of an interface we are inheriting from, but we do need to *include* their definition, using the `import` statement. In our example, we did this by importing `unknwn.idl`.

Every interface is required to return a `HRESULT` so that the client can receive status information regarding the success or failure of an operation. To facilitate traditional function-specific return values, a function must declare a pointer to memory that will receive the return value, and have the function return information via the pointer. This pointer must include both the `out` and `retval` attributes, and should always be the last parameter in the list.

While these attributes identify the purpose of a parameter, each parameter still needs a data type. Like all languages, IDL has a set of intrinsic data types it supports.

Once the IUserInfo interface is defined, we can use it in the definition of the *UserInfo* object. It will be defined as an element of the UserInfo library element. The library element represents the type library as a whole, and is identified by a library identifier (LIBID). LIBIDs, like CLSIDs and IIDs, are also GUIDs. Following is the IDL definition of the UserInfo library and UserInfo object:

```cpp
//LIBID_UserInfo
//These are the attributes of the type library
[
	uuid(6db492b9-cd03-4511-a6bb-3ece8b3377f2),
	helpstring("UserInfo Type Library"),
	version(1.0)
]
//Definition of the UserInfo type library
library UserInfo
{
	//CLSID_UserInfo
	//Attributes of the UserInfo object
	[
		uuid(cf79f686-dc42-4986-97c5-5f0d0bdcdfa8),
		helpstring("UserInfo Object")
	]
	//Definition of the UserInfo object
	coclass UserInfo
	{
		//List all of the interfaces supported by the object
		[default] interface IUserInfo;
	}
}
```

Because a COM object may support many different interfaces, IDL supplies the default attribute to signal macro languages that IUserInfo is the interface to use for programmatic control. Had the UserInfo COM object supported additional interfaces, they would have all been listed as part of the coclass UserInfo definition. The coclass defines the various interfaces supported by a particular COM object, which ultimately defines the object itself.

```shell
C:\Users\laughtersec\Desktop\myComObject>midl.exe UserInfo.idl
Microsoft (R) 32b/64b MIDL Compiler Version 8.01.0628
Copyright (c) Microsoft Corporation. All rights reserved.
Processing .\UserInfo.idl
UserInfo.idl
Processing C:\Program Files (x86)\Windows Kits\10\\include\10.0.26100.0\\um\unknwn.idl
unknwn.idl
Processing C:\Program Files (x86)\Windows Kits\10\\include\10.0.26100.0\\shared\wtypes.idl
wtypes.idl
Processing C:\Program Files (x86)\Windows Kits\10\\include\10.0.26100.0\\shared\wtypesbase.idl
wtypesbase.idl
Processing C:\Program Files (x86)\Windows Kits\10\\include\10.0.26100.0\\shared\basetsd.h
basetsd.h
Processing C:\Program Files (x86)\Windows Kits\10\\include\10.0.26100.0\\shared\guiddef.h
guiddef.h
C:\Users\laughtersec\Desktop\myComObject>dir
<...>
 Directory of C:\Users\laughtersec\Desktop\myComObject

21-11-2025  23:30    <DIR>          .
22-11-2025  11:30    <DIR>          ..
21-11-2025  22:36               805 dlldata.c
21-11-2025  22:36             6,766 UserInfo.h
21-11-2025  22:35             1,620 UserInfo.idl
21-11-2025  22:36             3,104 UserInfo.tlb
21-11-2025  22:36             1,992 UserInfo_i.c
21-11-2025  22:36            16,206 UserInfo_p.c
               6 File(s)         30,493 bytes
               2 Dir(s)   9,826,025,472 bytes free

C:\Users\laughtersec\Desktop\myComObject>
```

We can see the following generated files

| File Name    | Purpose                                                                                                                                                                                 | Important to note  |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| UserInfo_p.c | Contains code that can be used to generate proxy/stub pair for marshaling and unmarshaling `UserInfo` function calls.                                                                   |                    |
| dlldata.c    | Same as above. And in the case of in-process servers which are located in                                                                                                               |                    |
| UserInfo.h   | Contains the C and C++ definitions for the `IUserInfo` interface is declared as a C++ abstract class, which means that at least one of its member functions is a pure virtual function. |                    |
| UserInfo_i.c | Contains the definitions of the human-readable names that are used to refer to the `IUserInfo` interface, the type library, and the `UserInfo` object class.                            |                    |
| UserInfo.tlb | Actual compiled type library, essentially a language-independent header file.                                                                                                           | Easier to analyze. |

So far, we just defined the skeleton, now we can start working on the implementation (the procedures that the COM object can carry out).

## Implementing Interface Functions

While all of the interfaces have been defined, they are defined as C++ abstract base classes, which means that the C++ classes that define them cannot provide an implementation for them. ==Implementation is provided in the C++ classes that subsequently inherit from the abstract base class==. Seems counter-intuitive right? That's the academic dogma giving you a stroke. [Here you go](https://stackoverflow.com/questions/2697783/what-does-program-to-interfaces-not-implementations-mean). 

We can create a C++ class which will inherit from our interface `IUserInfo` abstract base class. And in this class we will then provide the implementation of it.

```cpp
class CUserInfo : IUserInfo
{
	private:
		ULONG m_cRef;
		short m_nAge;
		LPSTR m_lpszName;
		BYTE m_bySex;
	public:
		//IUnknown
		STDMETHODIMP QueryInterface(REFIID iid, LPVOID *ppv);
		STDMETHODIMP _(ULONG)AddRef(void);
		STDMETHODIMP _(ULONG)Release(void);
		//IUserInfo
		STDMETHODIMP get_Age(short *nRetAge);
		STDMETHODIMP put_Age(short nAge);
		STDMETHODIMP get_Name(LPSTR *lpszName);
		STDMETHODIMP put_Name(LPSTR lpszName);
		STDMETHODIMP get_Sex(BYTE *byRetSex);
		STDMETHODIMP put_Sex(BYTE byteSex);
		//Constructor
		CUserInfo();
		//Destructor
		~CUserInfo();
};  //CUserInfo COM object
```

`STDMETHODIMP` is just a macro that expands to `HRESULT __stdcall`.

Now, all that remains is providing an implementation for each function prototype/member function.

The first COM-related function is `QueryInterface` which is used for interface navigation. If a client calls `QueryInterface` with the IID of a supported interface, the object should respond by returning a pointer to that particular interface. ==In the case of the `UserInfo` COM object, if a client calls `QueryInterface` with the IID for either `IUnknown` or `IUserInfo`, `UserInfo` should return a pointer to that interface.== In the following example, we can see how the CUserInfo::QueryInterface object casts itself into either a `IUserInfo` pointer or an [[Concepts/IUnknown]] pointer, depending on the requested IID. ==If the requested interface is supported, `QueryInterface` calls `AddRef` to increase the reference count in accordance with COM's reference counting rules.== If all goes well, `QueryInterface` reports `NOERROR` to the client. ==If a client requests an unsupported interface, `E_NOINTERFACE` is returned to notify the client that the requested interface is not supported.== 

```cpp
STDMETHODIMP CUserInfo::QueryInterface(REFIID iid, LPVOID *ppv)
{
	*ppv = NULL;
	if (IID_IUnknown == iid)
		*ppv = (LPVOID)(IUnknown *)this;
	else if (IID_IUserInffo == iid)
		*ppv = (LPVOID)(IUserInfo *)this;
	else
		return E_NOINTERFACE;  //  Interface not supported
	//  Perform reference count through the returned interface
	((IUnknown *)*ppv)->AddRef();
	return NOERROR;
}  //  QueryInterface
```

The next COM-specific function that we investigate is the `AddRef` function. Whenever the number of outstanding references to an object is increased, `AddRef` is called to increase the object's internal reference counter:

```cpp
STDMETHODIMP_(ULONG)CUserInfo::AddRef(void)
{
	return ++m_cRef;
}  //  AddRef
```

Now the `Release` function is the complete opposite of the `AddRef` function. 

```cpp
STDMETHODIMP_(ULONG)CUserInfo::Release(void)
{
	m_cRef--;
	if (0 == m_cRef)
	{
		delete this;
		//  Decrement the global object count
		g_cObjects--;
		//  See if it's alright to unload the server
		if (::ServerCanUnloadNow())
			::UnloadServer();
		return 0;
	}
	return m_cRef;
}  //  Release
```

Whenever the number of outstanding references to an object is decreased, `Release` is called to decrease the object's internal reference counter. When there are no outstanding references, the object deletes itself. Once the object has deleted itself, the global object counter `g_cObjects` is decremented to reflect the fact that there is now one less object in existence. After all of this, the `ServerCanUnloadNow` function is consulted to determine if it is all right for the entire COM server to unload itself from memory, a feat that would be accomplished by the `UnloadModule` function. But, because DLLs aren't responsible for unloading themselves, the `UnloadModule` function simply returns. ==However, if the UserInfo COM object were being implemented in an EXE server,== `UnloadModule`==would actually unload the EXE server from memory (this is different from the current discussion of in-process servers).==

> This bit might naturally make you wonder the significance of counting references (unless you don't care). Since COM is a very integral part of the working of the Windows operating system, it is good practice to keep tabs on created objects that constantly occupy memory. Why do we keep tabs? So that cleanup/garbage collection is organized and automated, leaving the programmer to do things that "really matter". But we all know that automation is an 80-20 concept that can't solve **all** problems. It is a programmer's responsibility to compensate for the "20", if not singlehandedly the "80".

## Implementing a Class Factory

This step deals with the instantiation of the classes rather than defining it. A "factory" is something that produces a certain unit in large numbers. Same principal applied here, but the unit that the class factory produces is instantiations of our COM object.

To do this, we will define another COM object whose sole purpose is to act as an instantiator, or class factory, of our COM object.

Typically, class factories implement the `IClassFactory` interface and is ultimately responsible for creating other COM objects, as discussed before.

```cpp
class CUserInfoFactory : IClassFactory
{
	STDMETHODIMP LockServer(BOOL block);
	STDMETHODIMP CreateInstance(IUnknown* pUnknownOuter, REFIID iid, LPVOID *ppv);
}
```

The `IClassFactory` interface has only two methods:
- `LockServer`: called during COM server unloads.

```cpp
STDMETHODIMP CUserInfoFactory::LockServer(BOOL block)
{
	if (block)
		g_cLocks++;
	else
	{
		g_cLocks--;
		//  Check if server can be unloaded
		if(::ServerCanUnloadNow())
			::UnloadServer();
	}
	return NOERROR;
}  //  LockServer
```

Because an in-process server cannot unload itself, the operating system must ask the server if it can be unloaded by calling the `DllCanUnloadNow` function. This means that every in-process server is responsible for implementing and exposing the `DllCanUnloadNow` function. This is a relatively simply function that returns `S_OK` if there are no existing instances of objects and no outstanding locks; otherwise, the function returns `S_FALSE`.

- `CreateInstance`: called whenever a client wants to instantiate an instance of a particular COM object; through a call to `CoCreateInstance` for example. The `CUserInfoFactory`, in our example, is derived from `IClassFactory` responsible for creating `UserInfo` objects.

```cpp
STDMETHODIMP CUserInfoFactory::CreateInstance(IUnknown* pUnknownOuter, REFIID iid, LPVOID *ppv)
{
	HRESULT hr;
	CUserInfo *pCUserInfo = NULL;
	*ppv = NULL;
	//  This object doesn't support aggregation
	if (NULL != pUnknownOuter)
		return CLASS_E_NOAGGREGATION;
	//  Create the CUserInfo object
	pCUserInfo = new CUserInfo();
	if (NULL == pCUserInfo)
		return E_OUTOFMEMORY;
	//  Retrieve the requested interface
	hr = pCUserInfo->QueryInterface(iid, ppv);  //  Instantiation here
	if(FAILED(hr))
	{
		delete pCUserInfo;
		pCUserInfo = NULL;  //  stop Use-After-Free from emerging
		return hr;
	}
	//  Increment the global object counter
	g_cObjects++;
	
	return NOERROR;
}  //  CreateInstance
```

Clients using `CreateInstance` create an instance of a COM object and receive a pointer to a specific interface all in one call. ==Since an object has an initial reference count of zero upon instantiation, the call to== `QueryInterface` ==also serves to increment the object's reference count to one.== If all goes well, `CreateInstance` also increments a global object counter, which is used to keep track of the total number of objects being served. ==Since in-process servers cannot maintain their own global memory, the global object counter is really the total number of objects being used by a particular client.==

## Registering Class Information

In order for COM's implementation locator services to locate, load and launch your server, you must add each COM object's class information to the system registry, which is typically done once, as part of the installation process. Therefore, every server must provide a mechanism through which it can be notified to either register or unregister class information. COM requires that in-process servers provide this capability through `DllRegisterServer` and `DllUnregisterServer` functions, which are defined in `<olectl.h>`. Applications like REGSVR32.EXE that are used to register and unregister in-process COM servers simply call the Win32 API function `LoadLibrary` to load a particular server, and then call the Win32 API function `GetProcAddress` to obtain a pointer to either the `DllRegisterServer` or `DllUnregisterServer` function.

When a `UserInfo` client invokes the `DllRegisterServer` function, the server must add the `UserInfo` CLSID as a subkey under the `HKEY_CLASSES_ROOT\CLSID` key and optionally provide a textual description of the `UserInfo` COM object. Here's what it would look like in a registry:

```
HKEY_CLASSES_ROOT
	CLSID
		{some-GUID} = Description
		InprocServer32 = C:\UserInfo\UserInfo.dll
```

Since its part of a registry, the registry keys need to be modified, using the Windows API.

Here's a custom function to do so

```cpp
BOOL SetRegKeyValue(LPTSTR lpszKey, LPTSTR lpszSubKey, LPTSTR lpszValue)
{
	BOOL bOk = FALSE;
	long lErrorCode;
	HKEY hKey;
	_TCHAR szKey[MAX_STRING_LENGTH + 1];
	
	_tcscpy(szKey, lpszKey);
	if (NULL != lpszSubKey)
	{
		_tcscat(szKey, _TEXT("\\"));
		_tcscat(szKey, lpszSubKey);
	}
	lErrorCode = RegCreateKeyEx(
		HKEY_CLASSES_ROOT, 
		szKey, 
		0,
		NULL,
		&hKey,
		NULL
	);
	if (ERROR_SUCCESS == lErrorCode)
	{
		lErrorCode = RegSetValueEx(
			hKey, 
			NULL, 
			0, 
			REG_SZ,
			(BYTE *) lpszValue,
			sizeof(lpszValue) / sizeof(_TCHAR)
		);
		if (ERROR_SUCCESS == lErrorCode)
			bOk = TRUE;
		RegCloseKey(hKey);
	}
	return bOK;
}  //  SetRegKeyValue
```

## Exposing the Class Factory

We already implemented a class factory, whose job is to instantiate our COM object. The call to `CoGetClassObject` is made to retrieve a pointer to the `IClassFactory` interface, which is responsible for creating objects of the desired CLSID, which is supplied as the first parameter to `CoCreateInstance`. To obtain the appropriate `IClassFactory` pointer **from an in-process server**, COM checks the registry to retrieve the pathname of the appropriate server. After obtaining the server's pathname, COM calls the COM API function `CoLoadLibrary` to load the server into memory. After that, ==COM calls the Win32 API function ==`GetProcAddress` ==to request the address of the== `DllGetClassObject` ==function.== COM requires that every in-process server implement and expose a `DllGetClassObject` function. Based on a CLSID that is passed to it, `DllGetClassObject` is responsible for creating the appropriate class factory. After receiving the appropriate `IClassFactory` interface pointer, the COM system services call `IClassFactory::CreateInstance` to instantiate the desired COM object.

The `DllGetClassObject` function begins by validating that the object identified by the CLSID parameter can be created by the server; if it can't, `DllGetClassObject` returns `CLASS_E_CLASSNOTAVAILABLE`. However, if the server is capable of creating the object, `DllGetClassObject` creates an instance of the class factory responsible for creating the desired object. ==Once the class factory is instantiated,== `DllGetClassObject` ==calls== `QueryInterface`==, passing it the client-specified IID, which in most cases is== `IClassFactory`. The call to `QueryInterface` also serves to increment the class factory object's reference count from zero to one.

```cpp
STDAPI DllGetClassObject(REFCLSID rclsid, REFIID riid, LPVOID *ppv)
{
	CUserInfoFactory *pCUserInfoFactory = NULL;
	HRESULT hr = NOERROR;
	
	if (CLSID_UserInfo == rclsid)
	{
		//  Create the UserInfo classFactory
		pCUserInfoFactory = new CUserInfoFactory();
		//  Check for out of memory error
		if (NULL == pCUserInfoFactory)
			return E_OUTOFMEMORY;
		//  Get the requested interface
		hr = pCUserInfoFactory->QueryInterface(riid, ppv);
		if (FAILED(hr))
		{
			delete pCUserInfoFactory;
			pCUserInfoFactory = NULL;
			return hr;
		}
	}
	else
		//  Object not supported
		hr = CLASS_E_CLASSNOTAVAILABLE;
		
	return hr;
}  //  DllGetClassObject
```

## Server Unloading

Typically, when the last reference on the last object being served by a server is released, the server is unloaded. However, if the server is responsible for creating objects that typically have a short life span, it may be desirable to keep the server loaded in memory even when it isn't serving any objects, thus eliminating the overhead associated with loading and unloading the server. To accommodate this functionality, the `IClassFactory` interface supports the `LockServer` function, as seen in [[Concepts/In-Process Servers#Implementing a Class Factory]]. `LockServer` takes a single boolean parameter that determines whether or not the server should be "locked" in memory. If the value is true, the global reference counter is incremented. If the value is false, the global reference counter is decremented. ==Only when there are no locks and no instantiated objects can the server unload==.

## Putting it all together

All that is needed now to build the `UserInfo` server is a module definition file (.def). The module definition file is used to export the `DllRegisterServer`, `DllUnregisterserver`, `DllGetClassObject`, and `DllCanUnloadNow` functions from the DLL. The `.def` file looks like this:

```def
LIBRARY		UserInfo
DESCRIPTION	"UserInfo In-Process Server."
EXPORTS
	DllRegisterServer	@1	PRIVATE
	DllUnregisterServer	@2	PRIVATE
	DllGetClassObject	@3	PRIVATE
	DllCanUnloadNow		@4	PRIVATE
```

Here is `dllMain.cpp`

```cpp
#include "UserInfo.h"  //  Goes without saying
#define MAX_STRING_LENGTH 255
#define GUID_SIZE 128

#include<objbase.h>
#include<olectl.h>   //  for DLLRegisterServer and DLLUnregisterServer
#include<tchar.h>

//
//Forward declarations
//
BOOL SetRegKeyValue(LPTSTR lpszKey, LPTSTR lpszSubKey, LPTSTR lpszValue);
BOOL ServerCanUnloadNow(void);
void UnloadServer(void);
//
//Global variables
//
HMODULE g_hModule = NULL;
ULONG g_cObjects = 0;
ULONG g_cLocks = 0;

//
//DllRegisterServer
//
STDAPI DllRegisterServer(void)
{
	BOOL bOK;
	_TCHAR szModulePath[MAX_PATH + 1];
	_TCHAR szCLSID[GUID_SIZE + 1];
	_TCHAR szCLSIDKey[MAX_STRING_LENGTH + 1];
	wchar_t wszGUID[GUID_SIZE + 1];

	//Obtain the path to server's executable file for later use
	GetModuleFileName(g_hModule, szModulePath, sizeof(szModulePath) / sizeof(_TCHAR));
	//Convert the CLSID to the format {00000000-0000-0000-0000-000000000000}
	StringFromGUID2(CLSID_UserInfo, wszGUID, sizeof(wszGUID) / sizeof(wchar_t));
	#ifdef _UNICODE
		//UNICODE
		_tcscpy(szCLSID, wszGUID);
	#else
		//SBCS and MBCS
		//Convert from the wide character set to the multibyte character set
		WideCharToMultiByte(CP_ACP, 0, wszGUID, -1, szCLSID, sizeof(szCLSID) / sizeof(_TCHAR), NULL, NULL);
	#endif
		//HKEY_CLASSES_ROOT\CLSID\{00000000-0000-0000-0000-000000000000}
		_tcscpy(szCLSIDKey, _TEXT("CLSID\\"));
		_tcscat(szCLSIDKey, szCLSID);
		bOK = SetRegKeyValue(szCLSIDKey, NULL, _TEXT("DCOM Enterprise Apps - UserInfo Object."));
		if (bOK)
			bOK = SetRegKeyValue(szCLSIDKey, _TEXT("InProcServer32"), szModulePath);
		if (bOK)
			return NOERROR;
		else
			return SELFREG_E_CLASS;
}  //  DLLRegisterServer

//
//DllUnregisterServer
//
STDAPI DllUnregisterServer(void)
{
	long lErrorCode;
	_TCHAR szCLSID[GUID_SIZE + 1];
	_TCHAR szCLSIDKey[MAX_STRING_LENGTH + 1];
	_TCHAR szInProcServer32Key[MAX_STRING_LENGTH + 1];
	wchar_t wszGUID[GUID_SIZE + 1];
	
	//Convert the CLSID to the format
	//{00000000-0000-0000-0000-000000000000}
	StringFromGUID2(CLSID_UserInfo, wszGUID, sizeof(wszGUID) / sizeof(wchar_t));
	#ifdef _UNICODE
		//UNICODE
		_tcscpy(szCLSID, wszGUID);
	#else
		//SBCS and MBCS
		//Convert from the wide character set to the multibyte character set
		WideCharToMultiByte(CP_ACP, 0, wszGUID, -1, szCLSID, sizeof(szCLSID) / sizeof(_TCHAR), NULL, NULL);
	#endif
	//HKEY_CLASSES_ROOT\CLSID\{00000000-0000-0000-0000-000000000000}
	_tcscpy(szCLSIDKey, _TEXT("CLSID\\"));
	_tcscat(szCLSIDKey, szCLSID);
	_tcscpy(szInProcServer32Key, szCLSIDKey);
	_tcscat(szInProcServer32Key, _TEXT("\\InProcServer32"));
	
	//Delete sub-keys first
	lErrorCode = RegDeleteKey(HKEY_CLASSES_ROOT, szInProcServer32Key);

	//Delete the entry under CLSID
	if (ERROR_SUCCESS == lErrorCode)
		lErrorCode = RegDeleteKey(HKEY_CLASSES_ROOT, szCLSIDKey);

	if (ERROR_SUCCESS == lErrorCode)
		return NOERROR;
	else
		return SELFREG_E_CLASS;
}  //  DllUnregisterServer

//
//DllGetClassObject
//
STDAPI DllGetClassObject(REFCLSID rclsid, REFIID riid, LPVOID* ppv)
{
	CUserInfoFactory* pCUserInfoFactory = NULL;
	HRESULT hr = NOERROR;

	if (CLSID_UserInfo == rclsid)
	{
		//Create the UserInfo classFactory
		pCUserInfoFactory = new CUserInfoFactory();
		//Check for out of memory error
		if (NULL == pCUserInfoFactory)
			return E_OUTOFMEMORY;
		//Get the requested interface
		hr = pCUserInfoFactory->QueryInterface(riid, ppv);
		if (FAILED(hr))
		{
			delete pCUserInfoFactory;
			pCUserInfoFactory = NULL;
			return hr;
		}
	}
	else
		hr = CLASS_E_CLASSNOTAVAILABLE;
	return hr;
} //  DllGetClassObject

//
//DllCanUnloadNow
//
STDAPI DllCanUnloadNow(void)
{
	if (ServerCanUnloadNow())
		return S_OK;
	else
		return S_FALSE;
}  //  DllCanUnloadNow

//
//SetRegKeyValue
//
BOOL SetRegKeyValue(LPTSTR lpszKey, LPTSTR lpszSubKey, LPTSTR lpszValue)
{
	BOOL bOk = FALSE;
	long lErrorCode;
	HKEY hKey;
	_TCHAR szKey[MAX_STRING_LENGTH + 1];
	
	_tcscpy(szKey, lpszKey);
	if (NULL != lpszSubKey)
	{
		_tcscat(szKey, _TEXT("\\"));
		_tcscat(szKey, lpszSubKey);
	}
	lErrorCode = RegCreateKeyEx(HKEY_CLASSES_ROOT, szKey, 0, NULL, REG_OPTION_NON_VOLATILE, KEY_ALL_ACCESS, NULL, &hKey, NULL);
	if (ERROR_SUCCESS == lErrorCode)
	{
		lErrorCode = RegSetValueEx(hKey, NULL, 0, REG_SZ, (BYTE*)lpszValue, sizeof(lpszValue) / sizeof(_TCHAR));
		if (ERROR_SUCCESS == lErrorCode)
			bOk = TRUE;
		RegCloseKey(hKey);
	}
	return bOk;
}  //  SetRegKeyValue

//
//ServerCanUnloadNow
//
BOOL ServerCanUnloadNow(void)
{
	//  The server can unload if there are no outstanding objects or class factory locks
	if (0 == g_cObjects && 0 == g_cLocks)
		return TRUE;
	else
		return FALSE;
}  //  ServerCanUnloadNow

//
//UnloadServer
//
void UnloadServer(void)
{
	//  Since DLLs aren't responsible for unloading themselves, simply return
	return;
}  //  UnloadServer

// DLL Entry Point
//STDAPI macro cannot be used because it contains HRESULT, which returned by a COM interface and not a DLL's entry point.
//APIENTRY is really just WINAPI which expands to __stdcall calling convention
extern "C" BOOL WINAPI DllMain(HMODULE hModule, DWORD dwReason, LPVOID lpReserved)  //  Check if this works with the rest of the implementation
{
	//  Save the dll module handle for later use
	if (DLL_PROCESS_ATTACH == dwReason)
		g_hModule = hModule;
}
```

[Full source is provided here](https://github.com/laughtersec/com-servers-and-clients)


