---
layout: post
title:  "STL模型的读取"
date:   2018-2-2 15:40:56
categories: jekyll update
---

STL模型作为一种存储三角面片的三维模型格式现在广泛运用于3d打印行业中。一般来说用3d打印的商业软件可以直接读取模型并且进行后续的切片等操作，但是由于商业软件的封装特点，往往难以得到所需要的信息。实际上STL模型是一种很简单的模型，他只存储了模型的三角面片的三个顶点坐标和法向量坐标，面与面之间的拓扑关系等是没有存储在内的，这样虽然减少了模型的数据量，但是在后续打印中也遗留下一定的困难。

### STL模型简介
STL模型实际上有两种数据存储格式，分别是二进制格式和ASCII格式，其中二进制格式占用空间少，ASCII格式便于阅读，但是不管是哪种格式，存储的内容都是一样的。

### ASCII格式
ASCII码格式的STL文件逐行给出三角面片的几何信息，每一行以1个或2个关键字开头。
在STL文件中的三角面片的信息单元facet是一个带矢量方向的三角面片，STL三维模型就是由一系列这样的三角面片构成。
整个STL文件的首行给出了文件名。
在一个STL文件中，每一个facet由7行数据组成，
facet normal 是三角面片指向实体外部的法矢量坐标，
outer loop 说明随后的3行数据分别是三角面片的3个顶点坐标，3顶点沿指向实体外部的法矢量方向逆时针排列。
ASCII格式的STL文件可以显示如下：
```
solid filename
   facet normal 0.000000e+000 1.000000e+000 0.000000e+000
      outer loop
         vertex 8.440000e+001 2.000000e+001 0.000000e+000
         vertex 0.000000e+000 2.000000e+001 0.000000e+000
         vertex 1.000000e+001 2.000000e+001 1.500000e+001
      endloop
   endfacet
   ```

### 二进制格式
二进制STL文件用固定的字节数来给出三角面片的几何信息。
文件起始的80个字节是文件头，用于存贮文件名；
紧接着用4个字节的整数来描述模型的三角面片个数，
后面逐个给出每个三角面片的几何信息。每个三角面片占用固定的50个字节，依次是:
3个4字节浮点数(角面片的法矢量)
3个4字节浮点数(1个顶点的坐标)
3个4字节浮点数(2个顶点的坐标)
3个4字节浮点数(3个顶点的坐标)
三角面片的最后2个字节用来描述三角面片的属性信息。
一个完整二进制STL文件的大小为三角形面片数乘以50再加上84个字节。
由于解码的问题，二进制格式的STL文件如果直接用记事本打开则显示出来大多数都是乱码。
二进制格式的STL文件可显示如下：
```
UINT8//Header//文件头
UINT32//Numberoftriangles//三角面片数量
//foreachtriangle（每个三角面片中）
REAL32[3]//Normalvector//法线矢量
REAL32[3]//Vertex1//顶点1坐标
REAL32[3]//Vertex2//顶点2坐标
REAL32[3]//Vertex3//顶点3坐标
UINT16//Attributebytecountend//文件属性统计
```

### 两种格式区别
读取STL文件首先要对文件进行分类，二进制文件和ASCII格式文件虽然信息量是相等的，但是读取方法不相同，这样判断一个STL文件到底是二进制文件还是ASCII格式文件就非常重要。
从简单的方法来看，ASCII格式的STL文件，首行读取的前五个字符一定是'solid',第二行开始的第一个非空格字符一定是'f'（由于生成STL格式的软件众多，一般来说f是第二行的第三个字符，但是由于标准不统一，这里采用非空格字符的方法），这样就可以用简单的方法将两类文件区分开来了，用C++代码实现如下：
```
string headStr;
string SecondStr;
getline(in, headStr);
getline(in, SecondStr);
in.close();

if (headStr.empty())
    return false;

int noempty=0;
while(SecondStr[noempty]==' ')
    noempty++;
if((headStr[0] == 's') && (SecondStr[noempty] == 'f'))
{
    cout << "ASCII File." << endl;
    ReadASCII(cfilename);
}
else
{
    cout << "Binary File." << endl;
    ReadBinary(cfilename);
}
```

### 读取ASCII格式文件
读取ASCII格式文件比较简单，由于ASCII格式是按行来储存数据的，利用C++的getline就可以很简单的来读取每一行的数据。
ASCII文件可以直接用输入的方式打开，代码如下：
```
ifstream in;  
in.open(cfilename, ios::in);  
```
读取文件之后，利用循环读取每一行数据，对其中的字母，符号等数据进行剔除，只留下浮点数的数据。由于数据包含了法向量的数据和3个顶点的坐标，法向量的数据实际上可以由三个顶点坐标求出，所以可以选择舍弃法向量数据也可以单独存储，读取代码如下：
```
int flag=0;
do   
{   
    i=0;   
    cnt=0;   
    in.getline(a,100, '\n');   
    while(a[i]!='\0')   
    {   
        if (!islower((int)a[i]) && !isupper((int)a[i]) && a[i]!=' ')   
          break;   
        cnt++;   
        i++;   
    }   

    while(a[cnt]!='\0')           
    {   
        str[j]=a[cnt];   
        cnt++;   
        j++;   
    }   
    str[j]='\0';   
    j=0;   

    if (sscanf(str,"%lf%lf%lf",&x,&y,&z)==3)   
    {   
        flag++;
        if(flag%4==1)
        {
            //save the normal factor
        }
        else
        {
            //save the vertices coordinates
        }
    }  
    pCnt++;  
}while(!in.eof());
```
### 读取二进制格式文件
读取二进制格式的文件时，相较于读取ASCII格式文件，在读取函数中需要添加_ios::binary_，代码如下：
```
ifstream in;
in.open(cfilename, ios::in | ios::binary);
```
由于前80个字节是文件头，可以直接舍弃
```
in.read(str, 80);
```
紧接着的四个字节整型描述模型的三角面片个数，所以可以事先读取出面片个数：
```
//number of triangles  
int unTriangles;
in.read((char*)&unTriangles, sizeof(int));
```
得到所有三角面片数量之后，利用for循环则可以很轻松得存储每个面片的数据信息：
```
for (int i = 0; i < unTriangles; i++)
{
    float coorXYZ[12];
    in.read((char*)coorXYZ, 12 * sizeof(float));
    //save the normal factor
    for(int j = 0 ; j < 4 ; j++)
    {
        //save the vertices coordinates
    }
}
```
### Reference
[1]严梽铭, 钟艳如. 基于VC++和OpenGL的STL文件读取显示[J]. 计算机系统应用, 2009, 18(3):172-175.
[2]https://baike.baidu.com/item/stl%E6%A0%BC%E5%BC%8F/3511640?fr=aladdin
[3]http://blog.csdn.net/chinamming/article/details/16918643