---
publish: true
title: IUnknown
created: 2026-08-31T04:42:47.242Z
---

The most fundamental of interfaces is IUnknown, the root of all COM interfaces.

| Pointers to functions | Purpose                                                                    |
| --------------------- | -------------------------------------------------------------------------- |
| AddRef                | Reference counting which determines the lifetime of the related COM object |
| Release               | Reference counting which determines the lifetime of the related COM object |
| QueryInterface        | Get a new interface pointer on an object                                   |

`IUnknown`==is the only COM interface that does not derive from another COM interface. Every other legal COM interface must derive from==`IUnknown`==directly or from one other legal COM interface, which itself must derive either from== `IUnknown`==directly or from one other legal COM interface.== This means that at the binary level all COM interfaces are pointers to vtbls that begin with the three entries stated in the table above. Any interface-specific methods will have vtbl entries that appear after these common three entries.
