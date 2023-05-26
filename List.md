```mojo
from String import String
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
struct PrintService:
    @staticmethod
    fn printArray(array:Pointer[Int],size:Int):
        var result:String = String("[")
        for i in range(size):
            result = result + String(array.load(i)) + ","
        result = result + "]"
        print(result)

    @staticmethod
    fn printArray(array:Pointer[F64],size:Int):
        var result:String = String("[")
        for i in range(size):
            result = result + String(array.load(i)) + ","
        result = result + "]"
        print(result)
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
    
    fn add(self,_index:Int,value:Type):
        var index =_index
        var size = self.size()
        if index<0:
            index = size+index
        var cap = self.capacity()
        size = size + 1
        self._size.set(size)
        if(size == cap):
            cap = cap * 2
            self.data.store(0, Pointer[Type].alloc(cap))
            self._cap.set(cap)
            for i in range(size-1):#previous size
                self.data.load(0).store(i,self.data_back.load(0).load(i))
            self.__add_Index(index,value)
            self.data_back.store(0,self.data.load(0))
        else:
            self.__add_Index(index,value)
    
    fn __add_Index(self,index:Int,value:Type):

        for i in range(self.size()-1,index,-1):
            self.data.load(0).store(i,self.data.load(0).load(i-1))
        self.data.load(0).store(index,value)
                       
    fn __add_Index(self,index:Int,data:Pointer[Type],size:Int):
        for i in range(self.size()-1,index+size-1,-1):
            self.data.load(0).store(i,self.data.load(0).load(i-size))
        for i in range(size):
            self.data.load(0).store(i+index,data.load(i))
            
    fn add(self,_index:Int,other:Self):
        self.add(_index,other.data.load(0).bitcast[Type](),other.size())
    
    
    fn add(self,_index:Int,data:Pointer[Type],_size:Int):
        var index =_index
        var size = self.size()
        if index<0:
            index = size+index
        var newCap = self.capacity()
        size = size + _size
        self._size.set(size)
        while(size>newCap):
            newCap = newCap * 2
            
        if newCap>self.capacity():
            self.data.store(0, Pointer[Type].alloc(newCap))
            self._cap.set(newCap)
            for i in range(size-1):#previous size
                self.data.load(0).store(i,self.data_back.load(0).load(i))
            self.__add_Index(index,data,_size)
            self.data_back.store(0,self.data.load(0))
        else:
            self.__add_Index(index,data,_size)
            
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
    
    fn add(self,other:Self):
        self.add(other.data.load(0),other.size())
        
    fn add(self,data:Pointer[Type],size:Int):
        for i in range(size):
            self.add(data[i])
            
    fn addMany(self,size:Int,value:Type):
        let data= Pointer[Type].alloc(size)
        for i in range(size):
            data.store(i, value)
        self.add(data,size)
        
    fn addEmpty(self,addSize:Int):
        var size = self.size()
        var newCap = self.capacity()
        size = size + addSize
        self._size.set(size)
        while(size>newCap):
            newCap = newCap * 2
            
        if newCap>self.capacity():
            self.data.store(0, Pointer[Type].alloc(newCap))
            self._cap.set(newCap)
            for i in range(size-1):#previous size
                self.data.load(0).store(i,self.data_back.load(0).load(i))
            self.data_back.store(0,self.data.load(0))
            
    fn addMany(self,index:Int,size:Int,value:Type):
        let data= Pointer[Type].alloc(size)
        for i in range(size):
            data.store(i, value)
        self.add(index,data,size)
        
    fn remove(self):
        var size = self.size()
        if(size>0):
            size = size - 1
            self._size.set(size)
            
    fn remove(self,_index:Int):
        let size = self.size()
        if(size==0):
            return None
        var index =_index
        if index<0:
            index = self.size()+index
        for i in range(index,size-1):
            self.data.load(0).store(i,self.data.load(0).load(i+1))
        self.remove()
        
    fn remove(self,_index:Int,length:Int):
        var size = self.size()
        if(size-length<0):
            return None
        size = size - length
        self._size.set(size)
        var index =_index
        if index<0:
            index = self.size()+index
        for i in range(index,size):
            self.data.load(0).store(i,self.data.load(0).load(i+length))

    
    fn __setitem__(self,i:slice,data:Pointer[Type]):
        for j in range(0,i.end - i.start):
            self.data.load(0).store(j+i.start,data.load(j))
    
    fn __setitem__(self, i: Int,value:Type):
        return self.data.load(0).store(i,value)
            
    fn __getitem__(self, x: Int) -> Type:
        var i = x
        if i<0:
            i = self.size()+i
        return self.data.load(0).load(i)
    
    fn __getitem__(self, s:slice) -> Pointer[Type]:
        let total:Int = (s.end - s.start)
        let newList = Pointer[Type].alloc(total)
        for j in range(total):
            newList.store(j,self.data.load(0).load(j+s.start))
        return newList
    
    fn printArray(self):
        #1d lists
        if Type==Int:
            PrintService.printArray(self.data.load(0).bitcast[Int](),self.size())
        if Type==F64:
            PrintService.printArray(self.data.load(0).bitcast[F64](),self.size())
```


```mojo
struct ListManager:
    @staticmethod
    fn print2D(array: List[List[Int]]):
        print("2d[")
        for i in range(array.size()):
            let new = array[i]
            new.printArray()
        print("]")
        
    @staticmethod
    fn print2D(array: List[List[F64]]):
        print("2d[")
        for i in range(array.size()):
            let new = array[i]
            new.printArray()
        print("]")
        
    @staticmethod
    fn print3D(array: List[List[List[Int]]]):
        print("3d[")
        for i in range(array.size()):
            let new = array[i]
            Self.print2D(new)
        print("]")
    
    @staticmethod
    fn print3D(array: List[List[List[F64]]]):
        print("3d[")
        for i in range(array.size()):
            let new = array[i]
            Self.print2D(new)
        print("]")
        
    @staticmethod
    fn print4D(array: List[List[List[List[Int]]]]):
        print("4d[")
        for i in range(array.size()):
            let new = array[i]
            Self.print3D(new)
        print("]")
    
    @staticmethod
    fn print4D(array: List[List[List[List[F64]]]]):
        print("4d[")
        for i in range(array.size()):
            let new = array[i]
            Self.print3D(new)
        print("]")
        
    @staticmethod
    fn getMeanList(inList :List[F64])->F64:
        let size :Int = inList.size()
        var result:F64 = (inList[0]/2.0)+(inList[1]/2.0)
        for i in range(2,size):
            result = result + ((inList[i] - result)/(i+1)) 
        return result
```


```mojo
var test1:List[Int] = List[Int]()
test1.add(8)
test1.add(56)
test1.add(544)
test1.add(488)
test1.add(-2,45)
test1.printArray()
# test1.remove(-1)
# test1.printArray()
test1.add(2,test1[1:3],2)
test1.printArray()
test1.remove(1,2)
test1.printArray()
test1.addMany(12,9)
test1.printArray()
```

    [8,56,45,544,488,]
    [8,56,56,45,45,544,488,]
    [8,45,45,544,488,]
    [8,45,45,544,488,9,9,9,9,9,9,9,9,9,9,9,9,]



```mojo
test1.remove(3,12)
test1.printArray()
test1.addEmpty(5)
test1.printArray()
```

    [8,45,45,9,9,]
    [8,45,45,9,9,9,9,9,9,9,]



```mojo
var test2:List[List[Int]] = List[List[Int]]()
test2.add(test1.copy())
print(test2.size())
```

    1



```mojo
print(test2[0].size())
```

    10



```mojo
test2[0].add(9)
```


```mojo
print(test2[0][1])
```

    45



```mojo
test2.add(test1.copy())
```


```mojo
test2.addEmpty(2)
```


```mojo
test2[2]= test1.copy()
test2[3]= test1.copy()
```


```mojo
test2[0].add(8)
```


```mojo
ListManager.print2D(test2)
```

    2d[
    [8,45,45,9,9,9,9,9,9,9,9,8,]
    [8,45,45,9,9,9,9,9,9,9,]
    [8,45,45,9,9,9,9,9,9,9,]
    [8,45,45,9,9,9,9,9,9,9,]
    ]



```mojo
var test5 = List[List[List[Int]]]()
```


```mojo
test5.add(test2.copy())
```


```mojo
ListManager.print3D(test5)
```

    3d[
    2d[
    [8,45,45,9,9,9,9,9,9,9,9,8,]
    [8,45,45,9,9,9,9,9,9,9,]
    [8,45,45,9,9,9,9,9,9,9,]
    [8,45,45,9,9,9,9,9,9,9,]
    ]
    ]



```mojo
test2[-1].add(5)
test2[-1].printArray()
```

    [8,45,45,9,9,9,9,9,9,9,5,]



```mojo

```
