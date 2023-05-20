sorry my code is complex and writing with bad code


```mojo
from Pointer import DTypePointer
from DType import DType
from Random import rand,random_f64
from Memory import memset_zero
from PythonInterface import Python
from Time import now
from String import String
```


```mojo
struct Array[Type: AnyType]:
    var data: Pointer[Type]
    var size: Int
    var cap: Int

    fn __init__(inout self):
        self.cap = 16
        self.size = 0
        self.data = Pointer[Type].alloc(self.cap)
    
    fn zero(inout self):
        memset_zero(self.data, self.size)
            
    fn __init__(inout self, size: Int, value: Type):
        self.cap = size * 2
        self.size = size
        self.data = Pointer[Type].alloc(self.cap)
        for i in range(self.size):
            self.data.store(i, value)

    fn __copyinit__(inout self, other: Self):
        self.cap = other.cap
        self.size = other.size
        self.data = Pointer[Type].alloc(self.cap)
        for i in range(self.size):
            self.data.store(i, other.data.load(i))
    
    fn __getitem__(self, i: Int) -> Type:
        return self.data.load(i)
    
    fn __setitem__(self, i: Int, value: Type):
        return self.data.store(i, value)
            
    fn __del__(owned self):
        self.data.free()
```


```mojo
struct CList:
    var size:Int
    var data:Pointer[F64]
    
    fn __init__(inout self,size:Int):
        self.size = size
        self.data = Pointer[F64].alloc(size)
        
    fn __init__(inout self,size:Int,cList:CList):
        self.size = size
        self.data = Pointer[F64].alloc(size)
        for i in range(size):
            self.data.store(i,cList[i])
        
    fn zero(inout self):
        memset_zero(self.data, self.size)
        
    fn __del__(owned self):
        self.data.free()
        
    fn autoAdd(self,start: F64,add: F64):
        var current:F64 = 0.0
        current = current + start
        for i in range(self.size):
            self.data.store(i,current)
            current = add+ current
            
    @always_inline
    fn __getitem__(self,  x: Int) -> F64:
        return self.data.load(x)
    
    @always_inline
    fn __setitem__(self, x: Int, val: F64):
        return self.data.store(x,val)
```


```mojo
struct Matrix:
    var data: DTypePointer[DType.f64]
    var rows: Int
    var cols: Int

    fn __init__(inout self, rows: Int, cols: Int):
        self.data = DTypePointer[DType.f64].alloc(rows * cols)
        rand(self.data, rows*cols)
        self.rows = rows
        self.cols = cols

    fn __del__(owned self):
        self.data.free()

    fn zero(inout self):
        memset_zero(self.data, self.rows * self.cols)

    @always_inline
    fn __getitem__(self, y: Int, x: Int) -> F64:
        return self.load[1](y, x)

    @always_inline
    fn __getitem__(self, y: Int) -> Array[F64]:
        let array:Array[F64] = Array(self.cols,F64(0))
        for i in range(self.cols):
            array[i] = self.load[1](y, i)
        return array
    
    @always_inline
    fn load[nelts:Int](self, y: Int, x: Int) -> SIMD[DType.f64, nelts]:
        return self.data.simd_load[nelts](y * self.cols + x)

    @always_inline
    fn __setitem__(self, y: Int, x: Int, val: F64):
        return self.store[1](y, x, val)

    @always_inline
    fn store[nelts:Int](self, y: Int, x: Int, val: SIMD[DType.f64, nelts]):
        self.data.simd_store[nelts](y * self.cols + x, val)
```


```mojo
struct Matrix3:
    var data: DTypePointer[DType.f64]
    var rows: Int
    var cols: Int
    var subs: Int

    fn __init__(inout self, rows: Int, cols: Int,subs: Int):
        self.data = DTypePointer[DType.f64].alloc(rows * cols*subs)
        # rand(self.data, rows*cols*subs)
        self.rows = rows
        self.cols = cols
        self.subs = subs

    fn __del__(owned self):
        self.data.free()

    fn zero(inout self):
        memset_zero(self.data, self.rows * self.cols * self.subs)

    @always_inline
    fn __getitem__(self, y: Int, x: Int, z: Int) -> F64:
        return self.load[1](y, x,z)

    @always_inline
    fn load[nelts:Int](self, y: Int, x: Int, z: Int) -> SIMD[DType.f64, nelts]:
        return self.data.simd_load[nelts]((y * self.cols * self.subs)*x * self.subs + z)

    @always_inline
    fn __setitem__(self, y: Int, x: Int, z: Int, val: F64):
        return self.store[1](y, x,z, val)

    @always_inline
    fn store[nelts:Int](self, y: Int, x: Int, z: Int, val: SIMD[DType.f64, nelts]):
        self.data.simd_store[nelts]((y * self.cols * self.subs)*x * self.subs + z, val)
```


```mojo
struct MatrixTools:
    fn __init__(inout self):
        pass
    fn getMeanIndexMatrix(self,matrix : Matrix,index: Int)->F64:
        let size :Int = matrix.cols
        var result:F64 = (matrix[index,0]+matrix[index,1])/2.0
        for i in range(2,size):
            result = result + ((matrix[index,i] - result)/(i+1)) 
        return result
    fn getMeanIndexList(self,cList : CList)->F64:
        let size :Int = cList.size
        var result:F64 = (cList[0]+cList[1])/2.0
        for i in range(2,size):
            result = result + ((cList[i] - result)/(i+1)) 
        return result
```


```mojo
fn getMeanIndexList(cList : CList)->F64:
        let size :Int = cList.size
        var result:F64 = (cList[0]+cList[1])/2.0
        for i in range(2,size):
            result = result + ((cList[i] - result)/(i+1)) 
        return result
fn getMeanIndexMatrix(matrix : Matrix,index: Int)->F64:
    let size :Int = matrix.cols
    var result:F64 = (matrix[index,0]+matrix[index,1])/2.0
    for i in range(2,size):
        result = result + ((matrix[index,i] - result)/(i+1)) 
    return result

fn splitOutput(value:F64,size:Int)->Array[F64]:
    let array:Array[F64] = Array(size,F64(0))
    let number:F64 = value/size
    for i in range(size):
        array[i] = number
        # print(i,number,cList[i])
    return array

fn sumOutput(array:Array[F64])->F64:
    let size:Int = array.size
    var total:F64 = F64()
    # print("size",size)
    for i in range(size):
        total = total + array[i]
        # print(i,cList[i],total)
    return total

fn toInt(num:F64)->Int:
    return num.cast[DType.si32]().to_int()

fn printMatrix(matrix:Matrix):
    var result:String = String("[ ")
    for i in range(matrix.rows):
        result = result + "["
        for j in range(matrix.cols):
            result = result + String(matrix[i,j]) + ","
        result = result + "] "
    result = result + " ]"
    print(result)

fn printArray(array:Array[Int]):
    var result:String = String("[")
    for i in range(array.size):
        result = result + String(array[i]) + ","
    result = result + "]"
    print(result)

fn printArray(array:Array[F64]):
    var result:String = String("[")
    for i in range(array.size):
        result = result + String(array[i]) + ","
    result = result + "]"
    print(result)
    
fn autoAddArray(array:Array[F64],start:F64,add:F64):
    var current:F64 = F64(0)
    current = current + start
    for i in range(array.size):
        array[i]=current
        current = add + current

fn round(num:F64)->Int:
    let intNum:Int = toInt(num)
    let value:F64 = num-intNum
    if(value<0.5):
        return intNum
    else:
        return intNum+1
```


```mojo
struct MyNn:
    var sizec:Int
    var sizer:Int
    var networkRate:F64
    var nList:Matrix
    var nListp:Matrix
    var inputList:Array[F64]
    var outputList:Array[F64]
    var weights:Matrix3
    
    fn __del__(owned self):
        self.inputList.zero()
        self.outputList.zero()
        
    fn __init__(inout self,sizec: Int,sizer :Int,networkRate:F64):
        self.sizec = sizec
        self.sizer = sizer
        self.networkRate = networkRate
        self.nList = Matrix(sizer+1,sizec)
        self.nList.zero()
        self.nListp = Matrix(sizer+1,sizec)
        self.nListp.zero()
        self.inputList = Array(sizec,F64(0))
        self.outputList = Array(sizec,F64(0))
        self.weights = Matrix3(sizer,sizec,sizec*sizec)
        self.firstWeights()
        
    fn firstWeights(self):
        for x in range(self.sizer):
            for y in range(self.sizec):
                for z in range(self.sizec*self.sizec):
                    self.weights[x,y,z]=1.0/self.sizec
    
    fn shakeWeights(self):
        for x in range(self.sizer):
            for y in range(self.sizec):
                for z in range(self.sizec*self.sizec):
                    self.weights[x,y,z]=self.weights[x,y,z]*( (random_f64()/10) + 0.95)
        
    fn copyToList(self,copyList:Matrix,inputList : Array[F64],index:Int):
        for i in range(self.sizec):
            copyList[index,i] = inputList[i] 
            
    fn forwardFeed(self, inputList: Array[F64]):
            self.copyToList(self.nList,inputList,0)
            for r in range(1,self.sizer+1):
                for i in range(self.sizec):
                    var total:F64 = F64(0.0)
                    for j in range(self.sizec):
                        total = total + (self.weights[r-1,i,j] * self.nList[r-1,j])
                    self.nList[r,i] = total
                    
    fn backwardFeed(self,outputList:Array[F64]):
        var meanT:F64 = F64(0.0)
        var distance:F64 = F64(0.0)
        var multi:F64 = F64(0.0)
        var data:F64 = F64(0.0)
        var rate:F64 = F64(0.0)
        var add:F64 = F64(0.0)
        var distanceM:F64 = F64(0.0)
        var distanceT:F64 = F64(0.0)
        let size: Int = self.sizer
        self.copyToList(self.nListp,outputList,self.sizec)
        # printMatrix(self.nListp)
        
        for r in range(self.sizec,0,-1):
            distanceT = F64(0.0)
            meanT = getMeanIndexMatrix(self.nList,r-1)
            
            for i in range(size):
                distance = self.nListp[r,i]-self.nList[r,i]
                # print("r",r)
                # print("nListp")
                # printArray(self.nListp[r])
                # print("nList")
                # printArray(self.nList[r])
                distanceT = distanceT + distance
                distanceM = distance / size
                # print("distance",distance,"distanceM",distanceM ,"meanT",meanT)
                for j in range(size):
                    data = self.nList[r-1,j]
                    rate = data/meanT
                    multi = rate * distance * self.networkRate
                    add = self.weights[r-1,i,j]*multi
                    # print("weight",self.weights[r-1,i,j])
                    self.weights[r-1,i,j] = self.weights[r-1,i,j] + add
                    # print("data",data,"rate",rate,"multi",multi,"add",add,"weight",self.weights[r-1,i,j])
            for i in range(size):
                distanceM = distanceT/size
                self.nListp[r-1,i] = distanceM + self.nList[r-1,i]
                    
    fn forwandAndbackward(self,inputList:Array[F64],outputList:Array[F64]):
        self.forwardFeed(inputList)
        self.backwardFeed(outputList)
        
    fn getOuptutArray(self)->Array[F64]:
        return self.nList[self.sizec]
```


```mojo
let dt = Python.import_module("sklearn.datasets")
var iris = dt.load_iris()
```


```mojo
#we using manula convert method
print(iris.data.size)
var size:Int = iris.data.size.to_index()
var colSize:Int = iris.data[0].size.to_index()
var rowSize:Int = toInt(F64(size)/colSize);
let myMatrix:Matrix = Matrix(rowSize,colSize)
for i in range(rowSize):
    for j in range(colSize):
        myMatrix[i,j] = iris.data[i][j].to_f64()
# let matrix:Matrix = Matrix(data[0].size,data.size/data[0].size)
```

    600



```mojo
#we using manula convert method
print(iris.target.size)
var size:Int = iris.target.size.to_index()
var myArray:Array[Int] = Array(size,0)

for i in range(size):
    myArray[i] = iris.target[i].to_index()
```

    150



```mojo
printArray(myArray)
```

    [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,]



```mojo
#deleted after add lists
dt = PythonObject()
iris = PythonObject()
```


```mojo
let test1:MyNn = MyNn(4,4,0.09)
```


```mojo
test1.forwardFeed(myMatrix[100])
let output = sumOutput(test1.getOuptutArray())
print(output,myArray[100])
```

    18.100000 2



```mojo
test1.backwardFeed(splitOutput(F64(myArray[100]),4))
```


```mojo
# test1.shakeWeights()
```


```mojo
print(test1.weights[3,2,3])
```

    0.245991



```mojo
printMatrix(test1.nList)
printMatrix(test1.nListp)
```

    [ [6.300000,3.300000,6.000000,2.500000,] [4.525000,4.525000,4.525000,4.525000,] [4.525000,4.525000,4.525000,4.525000,] [4.525000,4.525000,4.525000,4.525000,] [4.525000,4.525000,4.525000,4.525000,]  ]
    [ [2.275000,-0.725000,1.975000,-1.525000,] [0.500000,0.500000,0.500000,0.500000,] [0.500000,0.500000,0.500000,0.500000,] [0.500000,0.500000,0.500000,0.500000,] [0.500000,0.500000,0.500000,0.500000,]  ]



```mojo
fn trainIrisNetwork(nn:MyNn,inputMatrix:Matrix, outputArray:Array[Int],trainSize:Int,trainLoop:Int,shakeLoop:Int):
    for shake in range(shakeLoop):
        nn.shakeWeights()
        for train in range(trainLoop):
            for s in range(trainSize):
                for t in range(3):
                    let index=(50*t)+s
                    nn.forwandAndbackward(inputMatrix[index],splitOutput(F64(outputArray[index]+1),4))
```


```mojo
fn testIrisNetworkRate(nn:MyNn,inputMatrix:Matrix, outputArray:Array[Int]):
    let total:Int=outputArray.size
    var success:Int = 0
    for i in range(total):
        nn.forwardFeed(inputMatrix[i])
        let output = sumOutput(nn.getOuptutArray())
        if(round(output)==outputArray[i]+1):
            success = success + 1
        else:
            print(output,outputArray[i]+1)
    let rate:F64 = F64(success)/total*100
    print("success rate:",rate,"%")
```


```mojo
let first= now()
trainIrisNetwork(test1,myMatrix,myArray,5,17,10)

let second = now()
var result = second-first
print(result,"ns",result/1000000.0,"ms",result/1000000000.0,"sn")
```

    966059 ns 0.966059 ms 0.000966 sn



```mojo
testIrisNetworkRate(test1,myMatrix,myArray)
```

    2.573220 2
    2.336453 3
    2.212024 3
    2.328228 3
    2.293025 3
    success rate: 96.666667 %



```mojo
print(test1.nListp[4,0])
```

    0.750000



```mojo
printArray(splitOutput(F64(myArray[0]+1),4))
```

    [0.250000,0.250000,0.250000,0.250000,]



```mojo
print(test1.weights[3,2,3])
```


```mojo

```
