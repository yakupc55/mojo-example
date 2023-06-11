# Main Functions


```mojo
from Pointer import DTypePointer
from DType import DType
from Random import rand,random_float64,randint,random_ui64,random_si64
from Memory import memset_zero
from PythonInterface import Python
from Time import now
from String import String
from Vector import DynamicVector

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

# Some Functions


```mojo
fn toInt(num:Float64)->Int:
    return num.cast[DType.int32]().to_int()


fn round(num:Float64)->Int:
    let intNum:Int = toInt(num)
    let value:Float64 = num-intNum
    if(value<0.5):
        return intNum
    else:
        return intNum+1

fn roundBigF(num:Float64)->Float64:
    let num2 = num.cast[DType.float32]()
    return num2.cast[DType.float64]()
```

# Network Class


```mojo
struct HNn:
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
        
#     fn addRow(self,size:Int):
#         let newList = List[Float64]()
#         let rowSize = self.rowSize.get()
#         newList.addMany(size,Float64(0.0))
#         let prSize = self.nList[rowSize].size()
#         self.rowSize.set(rowSize+1)
#         self.nList.add(newList)
#         self.nListp.add(newList.copy())
#         self.addWeightRow(size,prSize)
    
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
            
    # fn forwardFeed(self, inputList: List[Float64]):
    #     self._copyToList(self.inputList,inputList)
    #     for r in range(1,self.nList.size()):
    #         for i in range(self.nList[r].size()):
    #             var total:Float64 = Float64(0.0)
    #             for j in range(self.nList[r-1].size()):
    #                 total = total + (self.weights[r-1][i][j] * self.nList[r-1][j])
    #             self.nList[r][i] = total
    
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
        # var add:Float64 = Float64(0.0)
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
                    # add = self.weights[r-1][i][j]*multi
                    self.weights[r-1][i][j] = self.weights[r-1][i][j] + multi
                    
                for i in range(self.nList[r-1].size()):
                    distanceM = distanceT/size
                    self.nListp[r-1][i] = distanceM + self.nList[r-1][i]                
                    
    fn forwardAndbackward(self,inputList:List[Float64],outputList:List[Float64]):
        self.forwardFeed(inputList)
        self.backwardFeed(outputList)
    
    fn giveTheFormula(self):
        let alpha:String = "xyzabcdefghijklmnoprstuvw"
        
        var result:String = "f("
        var process:String = "x"
        for i in range(1,self.inputSize.get()):
            process += String(",") + alpha[i]
        result += process + ") : "
        
        let x:Float64= roundBigF(self.weights[0][0][0])
        process = String(x)+"x"
        for i in range(1,self.inputSize.get()):
            let num:Float64= roundBigF(self.weights[0][0][i])
            if num>=0:
                process += String("+")
            process += String(num) + alpha[i]
        result += process
        print(result)
```

# train and test functions


```mojo
fn trainNetwork(nn:HNn,inputList:List[List[Float64]], outputList:List[Int],trainSize:Int,trainLoop:Int,shakeLoop:Int):
    let output = List[Float64]()
    output.add(0)
    for shake in range(shakeLoop):
        nn.shakeWeights()
        for train in range(trainLoop):
            for i in range(trainSize):
                output[0]= outputList[i]
                nn.forwardAndbackward(inputList[i],output)
                
fn testNetworkRate(nn:HNn,inputList:List[List[Float64]], outputList:List[Int], showAll:Bool,showError:Bool):
    let total:Int=inputList.size()
    var success:Int = 0
    for i in range(total):
        nn.forwardFeed(inputList[i])
        let output = nn.outputList[0]
        if(round(output)==outputList[i]):
            success = success + 1
        else:
            if (not showAll and showError): 
                print(output,outputList[i])
            else:
                pass
        if showAll:
            print(output,outputList[i])
    let rate:Float64 = Float64(success)/total*100
    print("success rate:",rate,"%")
```

## create input List


```mojo
var inputList = List[List[Float64]]()
for i in range(8):
    let values = List[Float64]()
    values.add(random_si64(2,40).to_int())
    values.add(random_si64(2,40).to_int())
    inputList.add(values)
ListManager.print2D(inputList)
```

    2d[
    [40.0,21.0,]
    [12.0,5.0,]
    [38.0,4.0,]
    [21.0,16.0,]
    [12.0,37.0,]
    [22.0,20.0,]
    [38.0,3.0,]
    [31.0,32.0,]
    ]


## create output List


```mojo
var outputList = List[Int]()
for i in range(inputList.size()):
    let num1 = inputList[i][0]
    let num2 = inputList[i][1]
    let calculate = ((num1*13)+(num2*-11))
    outputList.add(toInt(calculate))
outputList.printArray()
```

    [289,101,450,97,-251,66,461,51,]


## normal type result : 


```mojo
#craete networl
var normalNetwork:HNn = HNn(2,1,0.02)
normalNetwork.finishRows()

#train network
let start= now()
trainNetwork(normalNetwork,inputList,outputList,8,20,1)
let end = now()
var result = end - start
print(result,"ns",result/1000000.0,"ms")

#test network
testNetworkRate(normalNetwork,inputList,outputList,True,True)
```

    6353 ns 0.0063530000000000001 ms
    289.0 289
    101.0 101
    450.0 450
    97.0 97
    -251.0 -251
    66.0 66
    461.0 461
    51.0 51
    success rate: 100.0 %



```mojo
normalNetwork.printWeights()
```

    weights:
    3d[
    2d[
    [13.0,-11.0,]
    ]
    ]



```mojo
normalNetwork.giveTheFormula()
```

    f(x,y) : 13.0x-11.0y



```mojo

```


```mojo

```
