# MyNn dynamic version
-with list system


```mojo
from Pointer import DTypePointer
from DType import DType
from Random import rand,random_f64
from Memory import memset_zero
from PythonInterface import Python
from Time import now
from String import String
from Vector import DynamicVector
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
        
        if (size == 0):
            return F64(0)
        if (size == 1):
            return F64(inList[0])
        
        var result:F64 = (inList[0]/2.0)+(inList[1]/2.0)
        for i in range(2,size):
            result = result + ((inList[i] - result)/(i+1)) 
        return result
```


```mojo
struct MyNn:
    var rowSize: Data[Int]
    var inputSize: Data[Int]
    var outputSize: Data[Int]
    var networkRate: Data[F64]
    var nList: List[List[F64]]
    var nListp: List[List[F64]]
    var inputList: List[F64]
    var outputList: List[F64]
    var weights: List[List[List[F64]]]
        
    fn __init__(inout self,inputSize:Int,outputSize:Int,networkRate:F64):
        self.inputSize = Data[Int](inputSize)
        self.outputSize = Data[Int](outputSize)
        self.rowSize = Data[Int](0)
        self.networkRate = Data[F64](networkRate)
        self.nList = List[List[F64]]()
        self.nListp = List[List[F64]]()
        self.inputList = List[F64]()
        self.inputList.addMany(inputSize,F64(0.0))
        self.outputList = List[F64]()
        self.outputList.addMany(outputSize,F64(0.0))
        self.weights = List[List[List[F64]]]()
        
        #add first list from input list
        self.nList.add(self.inputList)
        self.nListp.add(self.inputList.copy())
        
    fn addRow(self,size:Int):
        let newList = List[F64]()
        let rowSize = self.rowSize.get()
        newList.addMany(size,F64(0.0))
        let prSize = self.nList[rowSize].size()
        self.rowSize.set(rowSize+1)
        self.nList.add(newList)
        self.nListp.add(newList.copy())
        self.addWeightRow(size,prSize)
    
    fn addWeightRow(self,size:Int,prSize:Int):
        let newRList = List[List[F64]]()
        let newCList = List[F64]()
        newCList.addMany(prSize,1.0/prSize)
        for i in range(size):
            newRList.add(newCList.copy())
        self.weights.add(newRList)
        
    fn finishRows(self):
        self.nList.add(self.outputList)
        self.nListp.add(self.outputList.copy())
        let prSize = self.nList[self.rowSize.get()].size()
        self.addWeightRow(self.outputSize.get(),prSize)
    
    fn printAllRows(self):
        print("all rows:")
        ListManager.print2D(self.nList)
    
    fn printAllRowsP(self):
        print("all rows:")
        ListManager.print2D(self.nListp)
    
    fn printWeights(self):
        print("weights:")
        ListManager.print3D(self.weights)
        
    fn _copyToList(self,mainList: List[F64],inputList: List[F64]):
        for i in range(mainList.size()):
            mainList[i] = inputList[i] 
            
    fn forwardFeed(self, inputList: List[F64]):
        self._copyToList(self.inputList,inputList)
        for r in range(1,self.nList.size()):
            for i in range(self.nList[r].size()):
                var total:F64 = F64(0.0)
                for j in range(self.nList[r-1].size()):
                    total = total + (self.weights[r-1][i][j] * self.nList[r-1][j])
                self.nList[r][i] = total
                
    fn shakeWeights(self):
        for x in range(self.weights.size()):
            for y in range(self.weights[x].size()):
                for z in range(self.weights[x][y].size()):
                    self.weights[x][y][z]=self.weights[x][y][z]*( (random_f64()/10) + 0.95)
                    
    fn backwardFeed(self,outputList:List[F64]):
        self._copyToList(self.nListp[-1],outputList)
        var meanT:F64 = F64(0.0)
        var distance:F64 = F64(0.0)
        var multi:F64 = F64(0.0)
        var data:F64 = F64(0.0)
        var rate:F64 = F64(0.0)
        var add:F64 = F64(0.0)
        var distanceM:F64 = F64(0.0)
        var distanceT:F64 = F64(0.0)

        for r in range(self.nList.size()-1,0,-1):
            distanceT = F64(0.0)
            let size = self.nList[r-1].size()
            meanT = ListManager.getMeanList(self.nList[r-1])
            
            for i in range(self.nList[r].size()):
                distance = self.nListp[r][i] - self.nList[r][i]
                distanceT = distanceT + distance
                distanceM = distance / size
                
                for j in range(size):
                    data = self.nList[r-1][j]
                    rate = data/meanT
                    multi = rate * distance * self.networkRate.get()
                    add = self.weights[r-1][i][j]*multi
                    self.weights[r-1][i][j] = self.weights[r-1][i][j] + add
                    
                for i in range(self.nList[r-1].size()):
                    distanceM = distanceT/size
                    self.nListp[r-1][i] = distanceM + self.nList[r-1][i]                
                    
    fn forwandAndbackward(self,inputList:List[F64],outputList:List[F64]):
        self.forwardFeed(inputList)
        self.backwardFeed(outputList)

```


```mojo
var test1:MyNn = MyNn(3,1,0.11)
```


```mojo
test1.addRow(2)
test1.addRow(2)
test1.finishRows()
test1.printAllRows()
test1.printWeights()
```

    all rows:
    2d[
    [0.0,0.0,0.0,]
    [0.0,0.0,]
    [0.0,0.0,]
    [0.0,]
    ]
    weights:
    3d[
    2d[
    [0.33333333333333331,0.33333333333333331,0.33333333333333331,]
    [0.33333333333333331,0.33333333333333331,0.33333333333333331,]
    ]
    2d[
    [0.5,0.5,]
    [0.5,0.5,]
    ]
    2d[
    [0.5,0.5,]
    ]
    ]



```mojo
let inputList1 = List[F64]()
inputList1.addMany(4,F64(0.1))
test1.forwardFeed(inputList1)
test1.printAllRows()
test1.printAllRowsP()
test1.printWeights()
```

    all rows:
    2d[
    [0.10000000000000001,0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,]
    ]
    all rows:
    2d[
    [0.0,0.0,0.0,]
    [0.0,0.0,]
    [0.0,0.0,]
    [0.0,]
    ]
    weights:
    3d[
    2d[
    [0.33333333333333331,0.33333333333333331,0.33333333333333331,]
    [0.33333333333333331,0.33333333333333331,0.33333333333333331,]
    ]
    2d[
    [0.5,0.5,]
    [0.5,0.5,]
    ]
    2d[
    [0.5,0.5,]
    ]
    ]



```mojo
test1.shakeWeights()
test1.printWeights()
```

    weights:
    3d[
    2d[
    [0.33034376785730496,0.34618827903802329,0.32207328505529353,]
    [0.32884463428522365,0.32117031226746851,0.33184357614091542,]
    ]
    2d[
    [0.49761500845789264,0.52158372003287756,]
    [0.48576241920427748,0.52044609427678945,]
    ]
    2d[
    [0.51804299179864688,0.50029779375724437,]
    ]
    ]



```mojo
let outputList1 = List[F64]()
outputList1.add((0.2))
test1.backwardFeed(outputList1)
test1.printAllRows()
test1.printWeights()
```

    all rows:
    2d[
    [0.10000000000000001,0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,]
    ]
    weights:
    3d[
    2d[
    [0.33216065858052013,0.34809231457273243,0.32384468812309763,]
    [0.33065327977379239,0.32293674898493957,0.33366871580969043,]
    ]
    2d[
    [0.50035189100441102,0.52445243049305834,]
    [0.48843411250990099,0.52330854779531177,]
    ]
    2d[
    [0.52374146470843197,0.50580106948857406,]
    ]
    ]



```mojo
test1.backwardFeed(outputList1)
test1.printAllRows()
test1.printWeights()
```

    all rows:
    2d[
    [0.10000000000000001,0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,0.10000000000000001,]
    [0.10000000000000001,]
    ]
    weights:
    3d[
    2d[
    [0.33398754220271298,0.35000682230288244,0.32562583390777466,]
    [0.33247187281254825,0.32471290110435674,0.33550389374664374,]
    ]
    2d[
    [0.50310382640493523,0.52733691886077017,]
    [0.49112050012870545,0.52618674480818595,]
    ]
    2d[
    [0.5295026208202247,0.51136488125294832,]
    ]
    ]



```mojo
test1.forwandAndbackward(inputList1,outputList1)
test1.printWeights()
```

    weights:
    3d[
    2d[
    [0.33570175513163664,0.35180325523581774,0.32729712982137332,]
    [0.33417830646908508,0.32637951133056553,0.33722588944915305,]
    ]
    2d[
    [0.50570787494071212,0.53002062318020038,]
    [0.49366252336949229,0.52886459570267541,]
    ]
    2d[
    [0.53497314705169519,0.51658019898999585,]
    ]
    ]



```mojo

```


```mojo

```
