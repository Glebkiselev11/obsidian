Also known as: Wrapper

## Structure

![[Pasted image 20221028214837.png]]

1. The **Component** declares the common interface for both wrappers and wrapper object.
2. **Concrete component** is a class of objects being wrapped. It defines the basic behavior, which can be altered by decorators. 
3. The **Base decorator** class has a field for referencing a wrapped object. The field's type should be declared as the component intarface so it can contain both concrete components and decarators. The base decorator delegates all operations to the wrapped object.
4. **Concrete Decorators** define extra behaviors that can be added to components dynamically. Concrete decorators override methods of the base decorator and execute their behaviour either before or after calling the parent method. 
5. The **Client** can wrap components in multiple layers of decorators, as long as it works with all objects via the component interface.