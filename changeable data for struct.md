```mojo
@register_passable
struct Data[Type: AnyType]:
    var __data:Pointer[Type]

    fn __init__()->Self:
        return Self{__data:Pointer[Type].alloc(1)}

    fn __init__(value:Type)->Self:
        let data = Pointer[Type].alloc(1)
        data.store(0,value)
        return Self{__data:data}
    
    fn __copyinit__(self)->Self:
        return Self{__data:self.__data}
    
    fn set(self,value:Type):
        self.__data.store(0,value)
    
    fn get(self)->Type:
        return self.__data.load(0)
```


```mojo
#you need let but , let does not support yet
#you can use data method for changeable values
from String import String
@value
struct Employee:
    var name: String
    var age: Int
    var department: String
    var salary: Data[F64] #changeable data
```


```mojo
let employee = Employee("bob",34,"engineering",Data[F64](9000))
```


```mojo
print(employee.salary.get())
```

    9000.0



```mojo
#now the value of salary is changed
employee.salary.set(7888)
print(employee.salary.get())
```

    7888.0



```mojo

```
