# UML类图

[toc]



## 类

### 具体类

![43jr348ujtolajoifdfa43](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/43jr348ujtolajoifdfa43.svg)

### 抽象类

同类图。只是将类名写出了*斜体*。



### 接口

同类图。只是类名上面标注`<<interface>>`。



## 关系



### 实现（implements）

![f3fjeioasfjcsdlfew](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/f3fjeioasfjcsdlfew.svg)

### 继承（extends）

![54i309w34olerj34oitmlk43](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/54i309w34olerj34oitmlk43.svg)

### 关联

#### 依赖

如果对象A中用到了对象B，但是和B的关系不太明显。可以称为**对象A依赖对象B**。

下图中，Student依赖Pen。

![q3453490jtlfklgj43tjml4w](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/q3453490jtlfklgj43tjml4w.svg)



#### 聚合

假定有部门类（Department）和员工类（Employee），**部门包含员工，并且部门撤了员工还在**。这种关系可以用聚合表示。

聚合的表示方式和依赖几乎一样，只是在**整体类**处加上空心菱形。

![czxvnkszhfiweaewa43](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/czxvnkszhfiweaewa43.svg)



#### 组合

假定有人类（Person）和头类（Head）。**头属于人的一部分，如果没有人，则头也就无意义。**这种关系可以用组合表示。

![fewjaoij435j3409fg8hr0s5](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/fewjaoij435j3409fg8hr0s5.svg)
