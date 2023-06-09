# MyNn dynamic version
-with list system


```mojo
from Pointer import DTypePointer
from DType import DType
from Random import rand,random_float64
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
    fn printArray(array:Pointer[Float64],size:Int):
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
        if Type==Float64:
            PrintService.printArray(self.data.load(0).bitcast[Float64](),self.size())
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
    fn print2D(array: List[List[Float64]]):
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
    fn print3D(array: List[List[List[Float64]]]):
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
    fn print4D(array: List[List[List[List[Float64]]]]):
        print("4d[")
        for i in range(array.size()):
            let new = array[i]
            Self.print3D(new)
        print("]")
        
    @staticmethod
    fn getMeanList(inList :List[Float64])->Float64:
        let size :Int = inList.size()
        
        if (size == 0):
            return Float64(0)
        if (size == 1):
            return Float64(inList[0])
        
        var result:Float64 = (inList[0]/2.0)+(inList[1]/2.0)
        for i in range(2,size):
            result = result + ((inList[i] - result)/(i+1)) 
        return result
```


```mojo
struct MyNn:
    var rowSize: Data[Int]
    var inputSize: Data[Int]
    var outputSize: Data[Int]
    var networkRate: Data[Float64]
    var nList: List[List[Float64]]
    var nListp: List[List[Float64]]
    var inputList: List[Float64]
    var outputList: List[Float64]
    var weights: List[List[List[Float64]]]
        
    fn __init__(inout self,inputSize:Int,outputSize:Int,networkRate:Float64):
        self.inputSize = Data[Int](inputSize)
        self.outputSize = Data[Int](outputSize)
        self.rowSize = Data[Int](0)
        self.networkRate = Data[Float64](networkRate)
        self.nList = List[List[Float64]]()
        self.nListp = List[List[Float64]]()
        self.inputList = List[Float64]()
        self.inputList.addMany(inputSize,Float64(0.0))
        self.outputList = List[Float64]()
        self.outputList.addMany(outputSize,Float64(0.0))
        self.weights = List[List[List[Float64]]]()
        
        #add first list from input list
        self.nList.add(self.inputList)
        self.nListp.add(self.inputList.copy())
    
        
    fn __copyinit__(inout self, other: Self):
        self.rowSize = other.rowSize
        self.inputSize = other.inputSize
        self.outputSize = other.outputSize
        self.networkRate = other.networkRate
        self.nList = List[List[Float64]]()
        self.nListp = other.nListp.copy()
        let size = other.nList.size()-2
        self.inputList = other.inputList.copy()
        self.outputList = other.outputList.copy()
        self.nList.add(self.inputList)
        for i in range(size):
            self.nList.add(other.nListp[i+1])
        self.nList.add(self.outputList)
        self.weights = other.weights.copy()
        
    fn addRow(self,size:Int):
        let newList = List[Float64]()
        let rowSize = self.rowSize.get()
        newList.addMany(size,Float64(0.0))
        let prSize = self.nList[rowSize].size()
        self.rowSize.set(rowSize+1)
        self.nList.add(newList)
        self.nListp.add(newList.copy())
        self.addWeightRow(size,prSize)
    
    fn addWeightRow(self,size:Int,prSize:Int):
        let newRList = List[List[Float64]]()
        let newCList = List[Float64]()
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
        
    fn _copyToList(self,mainList: List[Float64],inputList: List[Float64]):
        for i in range(mainList.size()):
            mainList[i] = inputList[i] 
            
    fn forwardFeed(self, inputList: List[Float64]):
        self._copyToList(self.inputList,inputList)
        for r in range(1,self.nList.size()):
            for i in range(self.nList[r].size()):
                var total:Float64 = Float64(0.0)
                for j in range(self.nList[r-1].size()):
                    total = total + (self.weights[r-1][i][j] * self.nList[r-1][j])
                self.nList[r][i] = total
                
    fn shakeWeights(self):
        for x in range(self.weights.size()):
            for y in range(self.weights[x].size()):
                for z in range(self.weights[x][y].size()):
                    self.weights[x][y][z]=self.weights[x][y][z]*( (random_float64()/10) + 0.95)
                    
    fn backwardFeed(self,outputList:List[Float64]):
        self._copyToList(self.nListp[-1],outputList)
        var meanT:Float64 = Float64(0.0)
        var distance:Float64 = Float64(0.0)
        var multi:Float64 = Float64(0.0)
        var data:Float64 = Float64(0.0)
        var rate:Float64 = Float64(0.0)
        var add:Float64 = Float64(0.0)
        var distanceM:Float64 = Float64(0.0)
        var distanceT:Float64 = Float64(0.0)

        for r in range(self.nList.size()-1,0,-1):
            distanceT = Float64(0.0)
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
                    
    fn forwandAndbackward(self,inputList:List[Float64],outputList:List[Float64]):
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
let inputList1 = List[Float64]()
inputList1.addMany(4,Float64(0.1))
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
    [0.32105125959129199,0.33195500440077397,0.32396530620708258,]
    [0.33929549055802283,0.34782309654089127,0.33398054573409153,]
    ]
    2d[
    [0.47672860552324237,0.50148500965705289,]
    [0.47538490930307992,0.47834211186312836,]
    ]
    2d[
    [0.50933863562045223,0.52152182474848463,]
    ]
    ]



```mojo
let outputList1 = List[Float64]()
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
    [0.32281704151904411,0.33378075692497822,0.32574711539122153,]
    [0.34116161575609194,0.34973612357186618,0.33581743873562903,]
    ]
    2d[
    [0.47935061285362018,0.50424317721016665,]
    [0.47799952630424686,0.48097299347837558,]
    ]
    2d[
    [0.51494136061227724,0.52725856482071798,]
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
    [0.32459253524739884,0.3356165510880656,0.32753872452587324,]
    [0.34303800464275047,0.35165967225151146,0.33766443464867496,]
    ]
    2d[
    [0.48198704122431507,0.50701651468482256,]
    [0.48062852369892023,0.48361834494250666,]
    ]
    2d[
    [0.52060571557901225,0.53305840903374591,]
    ]
    ]



```mojo
test1.forwandAndbackward(inputList1,outputList1)
test1.printAllRows()
test1.outputList.printArray()
test1.printWeights()
```

    all rows:
    2d[
    [0.10000000000000001,0.10000000000000001,0.10000000000000001,]
    [0.098774781086133773,0.1032362111542937,]
    [0.09995062845200145,0.097400902788687169,]
    [0.10395523872681432,]
    ]
    [0.10395523872681432,]
    weights:
    3d[
    2d[
    [0.32630718293813804,0.33738943272213767,0.32926893534913787,]
    [0.34485008982224535,0.35351730106140561,0.3394481341495702,]
    ]
    2d[
    [0.48447688903700764,0.50975396047459265,]
    [0.48311135368416142,0.48622946107753312,]
    ]
    2d[
    [0.52617693576178726,0.53861737023464684,]
    ]
    ]



```mojo

```


```mojo

```
