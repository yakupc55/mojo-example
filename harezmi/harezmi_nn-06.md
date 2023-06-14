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

fn roundBigF(num:Float64)->Float64:
    let num2 = num.cast[DType.float32]()
    return num2.cast[DType.float64]()

fn round(_num:Float64)->Int:
    let num = roundBigF(_num)
    let intNum:Int = toInt(num)
    let value:Float64 = num-intNum
    var add:Int=0
    if(value<0.5):
        add = 0
    else:
        add = 1
    if intNum>=0:
        return intNum + add
    else:
        return intNum - add


```

## noron and synapsis structures


```mojo
alias Synapsis_factorType:Int = 0
alias Synapsis_addType:Int = 1

@register_passable
struct Synapsis:
    var backwardFeedRate:Data[Float64]
    var isChangeable:Bool
    var startNoronIndex: Data[Int]
    var endNoronIndex: Data[Int]
    var value: Data[Float64]
    var type: Data[Int]
    fn __init__(start:Int,end:Int,type:Int,value:Float64,isChangeable:Bool,rate:Float64)->Self:
        return Self{startNoronIndex:start,endNoronIndex:end,type:type , value:value,isChangeable:isChangeable,backwardFeedRate:rate}

@register_passable
struct Noron:
    var data: Data[Float64]
    var index: Data[Int]
    var synapsisIndexList: List[Int] 
    fn __init__()->Self:
        return Self{data:Float64(0),synapsisIndexList: List[Int](),index:0}
    
    fn _print(self):
        print("data : ",self.data.get()," index : ",self.index.get())
        print("synapsis index list len : ", self.synapsisIndexList.size())
        print("----")
        self.synapsisIndexList.printArray()
        print("----")
```


```mojo
struct HNn2:
    var inputSize: Data[Int]
    var outputSize: Data[Int]
    var networkRate: Data[Float64]
    var inputNorons: List[Noron]
    var outputNoron: Noron
    var rowSize: Data[Int]
    var noronRowLists: List[List[Noron]]
    var backRowList: List[List[Noron]]
    var noronList: List[Noron]
    var synapsisList: List[Synapsis]
    var remainNoron: Noron
    var factorNoron: Noron
    var remainResultNoron: Noron
    var factorSynapsises: List[Synapsis]
    var outputSynapsises: List[Synapsis]
    var remainSynapsis: Data[Synapsis]
    var layerSize:Int
    
    fn __init__(inout self,inputSize:Int,outputSize:Int,networkRate:Float64):
        self.layerSize = 2
        self.inputSize = Data[Int](inputSize)
        self.outputSize = Data[Int](outputSize)
        self.rowSize = Data[Int](0)
        self.networkRate = Data[Float64](networkRate)
        self.inputNorons = List[Noron]()
        self.remainNoron = Noron()
        self.outputNoron = Noron()
        self.factorNoron = Noron()
        self.remainResultNoron = Noron()
        self.noronList = List[Noron]()
        self.noronRowLists = List[List[Noron]]()
        self.backRowList = List[List[Noron]]()
        self.synapsisList = List[Synapsis]()
        self.factorSynapsises = List[Synapsis]()
        self.outputSynapsises = List[Synapsis]()
        self.remainSynapsis = Data[Synapsis](Synapsis(0,0,0,0,True,1.0))
        #functions
        self.addNorons()
        self.createNoronList()
        self.createSynapsisList()
        self.createBackList()
        self.shakeWeights()
        
    fn addNorons(self):
        for i in range(self.inputSize.get()):
            self.inputNorons.add(Noron())
            
    fn createNoronList(self):
        #create input Row:
        let inputRow = List[Noron]()
        for i in range(self.inputNorons.size()):
            inputRow.add(self.inputNorons[i])
        #change data of remain Noron
        self.remainNoron.data.set(1.0)
        inputRow.add(self.remainNoron)
        self.noronList.add(inputRow)
        self.noronRowLists.add(inputRow)
        
        #create process Row:
        let processRow = List[Noron]()
        processRow.add(self.factorNoron)
        processRow.add(self.remainResultNoron)
        self.noronList.add(processRow)
        self.noronRowLists.add(processRow)
        
        #output row:
        let outputRow = List[Noron]()
        outputRow.add(self.outputNoron)
        self.noronList.add(outputRow)
        self.noronRowLists.add(outputRow)
        
        #give index
        for i in range(self.noronList.size()):
            self.noronList[i].index.set(i)
            
    fn createSynapsisList(self):
        #factor synapsis
        for i in range(self.inputNorons.size()):
            self.factorSynapsises.add( Synapsis(self.inputNorons[i].index.get(),self.factorNoron.index.get(),Synapsis_factorType,1,True,1.0) )
            self.factorNoron.synapsisIndexList.add(i)
        self.synapsisList.add(self.factorSynapsises)
        
        #remain synapsis
        self.remainSynapsis.set( Synapsis(self.remainNoron.index.get(),self.remainResultNoron.index.get(),Synapsis_addType,0.0,True,10.0) )
        self.synapsisList.add(self.remainSynapsis.get())
        self.remainResultNoron.synapsisIndexList.add(self.inputSize.get())
        
        var lastIndex = self.inputSize.get()+1
        #outpus synapsis
        self.outputSynapsises.add( Synapsis(self.factorNoron.index.get(),self.outputNoron.index.get(),Synapsis_factorType,1,False,1.0) )
        self.outputNoron.synapsisIndexList.add(lastIndex)
        lastIndex = lastIndex + 1
        self.outputSynapsises.add( Synapsis(self.remainResultNoron.index.get(),self.outputNoron.index.get(),Synapsis_factorType,1,False,1.0) )
        self.outputNoron.synapsisIndexList.add(lastIndex)
        self.synapsisList.add(self.outputSynapsises)
    
    fn createBackList(self):
        for r in range(0, self.noronRowLists.size() ):
            let noronList = self.noronRowLists[r]
            let newNoronList = List[Noron]()
            for l in range(noronList.size()):
                newNoronList.add(Noron())
            self.backRowList.add(newNoronList)
            
    fn forwardFeed(self, inputList: List[Float64]):
        for i in range(self.inputSize.get()):
            self.inputNorons[i].data.set( inputList[i] )
            
        #we focus processing row and output row
        for r in range(1,self.noronRowLists.size()):
            let noronList = self.noronRowLists[r]
            for l in range(noronList.size()):
                let noron = noronList[l]
                var total = Float64(0.0)
                for i in range( noron.synapsisIndexList.size() ):
                    let synapsis = self.synapsisList[ noron.synapsisIndexList[i] ]
                    let inputNoron = self.noronList[ synapsis.startNoronIndex.get() ]
                    var process = Float64(0.0)
                    if synapsis.type.get() == Synapsis_factorType:
                        process = synapsis.value.get() * inputNoron.data.get()
                    elif synapsis.type.get() == Synapsis_addType:
                        process = synapsis.value.get() + inputNoron.data.get()
                    # print("value",synapsis.value.get())
                    total = total + process
                    if noron.synapsisIndexList.size()>0:
                        noron.data.set(total)
    
    fn getOutput(self)->Float64:
        return self.outputNoron.data.get()
    
    fn noronListToFloatList(self,noronList:List[Noron])->List[Float64]:
        let newList = List[Float64]()
        for i in range( noronList.size() ):
            newList.add(noronList[i].data.get())
        return newList.copy()
    
    fn synapsisListToFloatList(self,synapsisList:List[Synapsis])->List[Float64]:
        let newList = List[Float64]()
        for i in range( synapsisList.size() ):
            newList.add(synapsisList[i].value.get())
        return newList.copy()
    
    fn copyOutputToBackList(self, output:Float64):
        #output row is 2
        let noronList = self.backRowList[2]
        #output noron is first
        let noron = noronList[0]
        noron.data.set(output)
        
    fn shakeWeights(self):
        for i in range(self.synapsisList.size()):
            let sp = self.synapsisList[i]
            if sp.isChangeable:
                sp.value.set(( (random_float64()/10) + 0.95) * sp.value.get())
                    
    fn backwardFeed(self,output:Float64):
        self.copyOutputToBackList(output)
        
        var meanT:Float64 = Float64(0.0)
        var distance:Float64 = Float64(0.0)
        var multi:Float64 = Float64(0.0)
        var data:Float64 = Float64(0.0)
        var rate:Float64 = Float64(0.0)
        # var add:Float64 = Float64(0.0)
        var distanceM:Float64 = Float64(0.0)
        var distanceT:Float64 = Float64(0.0)

        for r in range(self.layerSize,0,-1):
            distanceT = Float64(0.0)
            
            let noronsStartList = self.noronRowLists[r-1]
            let noronsEndList = self.noronRowLists[r]
            let backList = self.backRowList[r]
            let startSize = noronsStartList.size()
            let endSize = noronsEndList.size()
            let dataList = self.noronListToFloatList(noronsStartList)
            meanT = ListManager.getMeanList(dataList)
            # print("r",r,"meanT",meanT)
            # print("startsize :" ,startSize," endSize : ", endSize)
            for l in range(endSize):
                let endNoron = noronsEndList[l]
                let backNoron = backList[l]
                distance = backNoron.data.get() - endNoron.data.get()
                # print("backnoron : ",backNoron.data.get()," endnoron :" ,endNoron.data.get())
                distanceT = distanceT + distance
                distanceM = distance / startSize
                # print("l : ",l,"  --- distance : ",distance,"distanceT : ",distanceT,"distanceM : ",distanceM," ---")
                let synapsisSize = endNoron.synapsisIndexList.size()
                for i in range(synapsisSize):
                    let synapsis = self.synapsisList[ endNoron.synapsisIndexList[i] ]
                    if synapsis.isChangeable:
                        # print("index : ",endNoron.synapsisIndexList[i]) 
                        let startNoron = self.noronList[ synapsis.startNoronIndex.get() ]
                        data = startNoron.data.get()
                        rate = data/meanT
                        multi = rate * distance * self.networkRate.get() * synapsis.backwardFeedRate.get()
                        # print("i : ",i," ---- data : ",data," rate : ",rate," multi : ",multi)
                        # print(" value : " , synapsis.value.get())
                        synapsis.value.set(synapsis.value.get() + multi)
                    
                for i in range(startSize):
                    distanceM = distanceT/startSize
                    let newBackList = self.backRowList[r-1]
                    newBackList[i].data.set( distanceM + noronsStartList[i].data.get())
        
    fn forwardAndBackwardFeed(self,inputList: List[Float64],output:Float64):
        self.forwardFeed(inputList)
        self.backwardFeed(output)
        
    fn printSynapsisList(self):
        let theList = self.synapsisListToFloatList(self.synapsisList)
        theList.printArray()
        
    fn printNoronList(self):
        let theList = self.noronListToFloatList(self.noronList)
        theList.printArray()
        
    fn giveTheFormula(self):
        let alpha:String = "xyzabcdefghijklmnoprstuvw"
        
        #function
        var result:String = "f("
        var process:String = "x"
        for i in range(1,self.inputSize.get()):
            process += String(",") + alpha[i]
        result += process + ") : "
        
        #factors
        let x:Float64= roundBigF(self.synapsisList[0].value.get())
        process = String(x)+"x"
        for i in range(1,self.inputSize.get()):
            let num:Float64= roundBigF(self.synapsisList[i].value.get())
            if num>=0:
                process += String("+")
            process += String(num) + alpha[i]
        result += process
        
        #remain
        let remain =  roundBigF(self.remainSynapsis.get().value.get())+1.0
        if remain >0.000001 or remain<-0.000001:
            if remain < 0:
                process =  String(remain)
            else:
                process = String("+")+String(remain)
            result += process
        
        print(result)
    
```


```mojo
fn trainNetwork2(nn:HNn2,inputList:List[List[Float64]], outputList:List[Int],trainSize:Int,trainLoop:Int,shakeLoop:Int):

    for shake in range(shakeLoop):
        # nn.shakeWeights()
        for train in range(trainLoop):
            for i in range(trainSize):
                nn.forwardAndBackwardFeed(inputList[i],Float64(outputList[i]))

fn trainNetwork3(nn:HNn2,inputList:List[List[Float64]], outputList:List[Int],trainSize:Int,trainLoop:Int,shakeLoop:Int):

    for shake in range(shakeLoop):
        # nn.shakeWeights()
        for train in range(trainLoop):
            for i in range(trainSize):
                for s in range(3):
                    let index=(50*s)+i   
                    nn.forwardAndBackwardFeed(inputList[index],Float64(outputList[index]))
                
fn testNetworkRate2(nn:HNn2,inputList:List[List[Float64]], outputList:List[Int], showAll:Bool,showError:Bool):
    let total:Int=inputList.size()
    var success:Int = 0
    for i in range(total):
        nn.forwardFeed(inputList[i])
        let output = nn.getOutput()
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


```mojo
alias trainSize:Int = 12
var inputList = List[List[Float64]]()
for i in range(trainSize):
    let values = List[Float64]()
    values.add(random_si64(2,40).to_int())
    values.add(random_si64(1,41).to_int())
    values.add(random_si64(3,39).to_int())
    inputList.add(values)
ListManager.print2D(inputList)

var outputList = List[Int]()
for i in range(inputList.size()):
    let num1 = inputList[i][0]
    let num2 = inputList[i][1]
    let num3 = inputList[i][2]
    let calculate = (num1*-2)+(num2*1)+(num3*2)+10
    outputList.add(toInt(calculate))
outputList.printArray()
```

    2d[
    [25.0,8.0,33.0,]
    [8.0,41.0,12.0,]
    [11.0,5.0,11.0,]
    [26.0,29.0,32.0,]
    [29.0,31.0,27.0,]
    [26.0,3.0,25.0,]
    [10.0,14.0,28.0,]
    [6.0,32.0,22.0,]
    [23.0,25.0,15.0,]
    [29.0,6.0,8.0,]
    [20.0,36.0,33.0,]
    [23.0,31.0,14.0,]
    ]
    [34,59,15,51,37,11,60,74,19,-26,72,23,]



```mojo
#craete networl
var normalNetwork:HNn2 = HNn2(3,1,0.03)

#train network
let start= now()
trainNetwork2(normalNetwork,inputList,outputList,trainSize,5000,1)

let end = now()
var result = end - start
print(result,"ns",result/1000000.0,"ms")
# normalNetwork.printSynapsisList()
# normalNetwork.printNoronList()

#test network
testNetworkRate2(normalNetwork,inputList,outputList,True,True)
```

    49582183 ns 49.582183000000001 ms
    33.999999999999957 34
    58.999999999999829 59
    14.999999999999783 15
    51.000000000000014 51
    37.000000000000057 37
    10.999999999999957 11
    59.999999999999794 60
    73.999999999999787 74
    18.999999999999972 19
    -26.0 -26
    71.999999999999957 72
    22.999999999999982 23
    success rate: 100.0 %



```mojo
normalNetwork.giveTheFormula()
```

    f(x,y,z) : -2.0x+1.0y+2.0z+10.0



```mojo

```
