#SOLID (object-oriented design)


Wikipedia : 
"In computer programming, SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion) is a mnemonic acronym introduced by Michael Feathers for the "first five principles" named by Robert C. Martin[1][2] in the early 2000s[3] that stands for five basic principles of object-oriented programming and design. The intention is that these principles, when applied together, will make it more likely that a programmer will create a system that is easy to maintain and extend over time.[3] The principles of SOLID are guidelines that can be applied while working on software to remove code smells by causing the programmer to refactor the software's source code until it is both legible and extensible. It is part of an overall strategy of agile and Adaptive Software Development.[3]"

##S=Single responsibility principle
A class should have only a single responsibility (i.e. only one potential change in the software's specification should be able to affect the specification of the class).

This does not mean that a class should only have one public method.

A class is likely to have more than one public method. But these methods must all target one responsibility. 

What constitutes a responsibility is a judgement call.

##O=Open/closed principle
Software entities … should be open for extension, but closed for modification.

The aim here is that you do not need to go back and edit existing class methods as the specification changes.

An example is a class responsible for validating a particular class type.
Rather than putting the validation code in one method in this class, code which will likely need to change as spec is refined.
Instead we could maintain a list of Validation Class instances and take that list as a parameter in the classes constructor.


##L=Liskov substitution principle
Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program

##I=Interface segregation principle
Many client-specific interfaces are better than one general-purpose interface

##D=Dependency inversion principle
One should “Depend upon Abstractions. Do not depend upon concretions

A useful patern here is the stairway pattern.
Define an assembly which contains only interfaces for a layer.
The classes which implement those interfaces should be in a separate assembly.

This allows the higher code layer to reference the interface assembly and not the implementation assembly.
The lower code level can reference the implementation assembly only.

This reduces coupling which is a good thing, it provides better dependency control and thus provides more maintainable code.
