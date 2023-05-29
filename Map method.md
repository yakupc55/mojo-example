```mojo
from Memory import memcpy, memcmp
from Pointer import DTypePointer, Pointer
from Assert import debug_assert
from DType import DType
from Vector import DynamicVector
from List import VariadicList
```


```mojo
@register_passable
struct MyString:
    var buffer: DTypePointer[DType.si8]
    var size:Int
    @always_inline
    fn __init__(str: StringRef)->Self:
        let len = str.__len__()
        let buffer = DTypePointer[DType.si8].alloc(len)
        memcpy(buffer.bitcast[DType.si8]().address,DTypePointer[DType.si8](str.data),len,)
        return Self{buffer:buffer,size:len}
        
    @always_inline
    fn __init__(str: StringLiteral)->Self:
        return Self(StringRef(str))
    
    @always_inline
    fn get(self)->StringRef:
        return StringRef {data: self.buffer.bitcast[DType.si8]().address, length: self.size}
```


```mojo
@register_passable
@value
struct Person:
    var name:MyString
    var age: Int
    fn __init__()->Self:
        return Self{name:"-",age:0}
    fn __init__(str:StringLiteral,age:Int)->Self:
        return Self{name:str,age:age}
```


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
@register_passable
struct List[Type: AnyType]:
    var data:Pointer[Pointer[Type]]
    var data_back:Pointer[Pointer[Type]]
    var _size:Data[Int]
    var _cap:Data[Int]
    
    fn __init__()->Self:
        let size = Data[Int](0)
        let cap = Data[Int](2)
        let data = Pointer[Pointer[Type]].alloc(1)
        let data_back = Pointer[Pointer[Type]].alloc(1)
        data.store(0, Pointer[Type].alloc(2))
        data_back.store(0,data.load(0))
        return Self{_size:size,_cap:cap,data:data,data_back:data_back}
    
    fn copy(self)->Self:
        let size = Data[Int](self.size())
        let cap = Data[Int](self.capacity())
        let data = Pointer[Pointer[Type]].alloc(1)
        let data_back = Pointer[Pointer[Type]].alloc(1)
        data.store(0, Pointer[Type].alloc(cap.get()))
        data_back.store(0,data.load(0))
        for i in range(size.get()):
            data.load(0).store(i,self.data.load(0).load(i))
        return Self{_size:size,_cap:cap,data:data,data_back:data_back}
    
    fn size(self)->Int:
        return self._size.get()
        
    fn capacity(self)->Int:
        return self._cap.get()

    fn add(self,value:Type):
        var size = self.size()
        var cap = self.capacity()
        size = size + 1
        self._size.set(size)
        if(size == cap):
            cap = cap * 2
            self.data.store(0, Pointer[Type].alloc(cap))
            self._cap.set(cap)
            
            for i in range(size-1):#previous size
                self.data.load(0).store(i,self.data_back.load(0).load(i))
            self.data.load(0).store(size-1,value)
            self.data_back.store(0,self.data.load(0))
        else:
            self.data.load(0).store(size-1,value)
    
    fn remove(self):
        var size = self.size()
        if(size>0):
            size = size - 1
            self._size.set(size)
            
    fn __setitem__(self, i: Int,value:Type):
        return self.data.load(0).store(i,value)
            
    fn __getitem__(self, x: Int) -> Type:
        var i = x
        if i<0:
            i = self.size()+i
        return self.data.load(0).load(i)
    
    #map using transfer layer
    fn map(self,f:fn(Type,inout Data[Type])->None)->Self:
        let newList:Data[Self] = Data[Self](self.copy())
        for i in range(self.size()):
            var value:Data[Type] = Data[Type]()
            f(self[i],value)
            newList.get()[i]=value.get()
        return newList.get()
```


```mojo
fn growAge(a:Person,inout transfer:Data[Person]):
    transfer.set(Person(a.name.get(),a.age+1))
```


```mojo
let person = Person("bob",6)
```


```mojo
let our_list:List[Person] = List[Person]()
our_list.add(person)
```


```mojo
print(our_list[0].name.get(),our_list[0].age)
```

    bob 6



```mojo
let new_list:List[Person] = our_list.map(growAge)
```


```mojo
print(new_list[0].age,our_list[0].age)
```

    7 6



```mojo

```
