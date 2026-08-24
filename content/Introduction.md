COM rejects the religious academic dogma of object-oriented programming.

Traditionally, inheritance is emphasized as the primary vehicle for reuse. The designers of COM disagree with this, and proposed that there were actually two kinds of inheritance, rather than just implementation inheritance.

Implementation inheritance assumes that actual implementation (behavior) is inherited. It is simply one mechanism for reusing an existing implementation.
However, **interface** inheritance assumes that only the specification of behavior is inherited. It is the latter form of inheritance that enables polymorphism. This type of inheritance is fully supported by COM.

Implementation inheritance has it's own issues apparently, where it can lead to undue coupling between a base class and a derived class. Because implementation inheritance often causes details of a base class' implementation to "leak", violating the encapsulation of the class, COM's designers felt that the use of implementation inheritance should be restricted to programming within components. While COM does not support implementation inheritance _across_ components, it does support it within components. It fully supports, and relies on, interface inheritance.

The fundamental concept used to model reuse of COM is encapsulation, not inheritance. Instead, COM uses inheritance to model type relationships between objects that share similar functionality. By building COM's reuse model on encapsulation, the designers were encouraging a form of _black-box_ reuse suitable for the anticipated component marketplace. The idea is that clients should treat objects as opaque components with respect to what is inside them and how they are implemented.

Object-Oriented Programming = Polymorphism + (Some) Late Binding + (Some) Encapsulation + Inheritance

Component-Oriented Programming = Polymorphism + (Really) Late Binding + (Real, Enforced) Encapsulation + Interface Inheritance + Binary Reuse
