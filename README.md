# 说的道理生成器


## 简介

基于python的简单朴素的说的道理生成器

## 使用

+ 将图片和**你是说的道理.exe**放在一个文件夹中

+ 运行**你是说的道理.exe**

+ 输入图片名称

**示例：**
<img src=https://user-images.githubusercontent.com/53858652/197927507-6f678499-eb2a-42f0-84e1-d1d800aafb1a.png width=500 height=300 />

输入图片：otto.png

<img src=https://user-images.githubusercontent.com/53858652/197927631-4f80a80e-338e-4f7a-8a43-08dbeb29bfdc.png width=400 height=300 />

结果

<img src=https://user-images.githubusercontent.com/53858652/197927635-ae546ce4-5855-4e86-8bf2-6925aa0ec714.png width=300 height=300 />


## 原理

灵感来自DJGun的这个视频https://www.bilibili.com/video/BV1624y1R7MU/

**构造原图的横纵坐标（u，v）到球极投影后的坐标（x‘，y'）的映射**

1. 先将图片贴到一个球上

$$
\begin{align}
x=r*cosu*sinv \\
y=r*sinu*sinv \\
z=rsinv \\
r=width/2π
\end{align}
$$


2. 再球极投影

$$
\begin{align}
x'=r*x/(r+z)\\
y'=r*y/(r+z)
\end{align}
$$

但是如果直接将图片映射到新图上，会发现

由于点过于稀疏，使得图片效果极差

于是

我们考虑**构造新图坐标（x，y）到 原图（u，v）的映射**，使得新图中的每个像素点都能在原图找到一个点

$$
\begin{align}
x'=r*x/(r+z)\\
y'=r*y/(r+z)\\
↓\\
x'^2+y'^2=r^2*(x^2+y^2)/(r+z)^2\\
x^2+y^2+z^2=r^2\\
↓\\
设t=(x'^2+y'^2)/r^2\\
t=(r^2-z^2)/(r+z)^2=(r-z)/(r+z)\\
↓\\
z=(1-t)*r/(1+t)
\end{align}
$$

得到 z

$$
\begin{align}
z=rcosv \\ 
↓ \\
v=arcos(z/r)\\
 \\
tanu=y/x=y'/x'\\
u=arctan(y'/x')
\end{align}
$$

于是我们就得到了转化的过程

python代码：

~~~python
def Yituosi(img):
    height,width=img.shape[:2]
    print(height,width)
    img_out=np.zeros((1001,1001,3), np.uint8)
    r=width/6.2831
    for i in range(-500,500):
        for j in range(-500,500):
            t=math.sqrt((i*i+j*j)/(r*r))
            z=(1-t)*r/(1+t)
            if abs(z)>r:
                continue
            k=z/r
            x=(r+z)*i/r
            y=(r+z)*j/r
            v=math.acos(k)
            u=math.atan2(j,i)
            if u<0:
                u+=2*3.1415926
            xx=int(u*width/(2*3.1415926))
            yy=int(v*height/(3.1415926))
            if xx<0 or yy<0:
                print(xx,yy)
            img_out[-i+500][j+500]=img[yy][xx]
    return img_out  
~~~

## 更新
----------------
对投影后的坐标进行一个缩放，使得图片可以笼罩更大的范围

$$
\begin{align}
令
x''=2*x' \\
y''=2*y'
\end{align}
$$

更新后的代码
~~~python
def Yituosi(img):
    height,width=img.shape[:2]
    print(height,width)
    img_out=np.zeros((1001,1001,3), np.uint8)
    r=width/6.2831
    for i in range(-500,500):
        for j in range(-500,500):
            t=math.sqrt((4*i*i+4*j*j)/(r*r))
            z=(1-t)*r/(1+t)
            if abs(z)>r:
                continue
            k=z/r
            x=(r+z)*2*i/r
            y=(r+z)*2*j/r
            v=math.acos(k)
            u=math.atan2(j,i)
            if u<0:
                u+=2*3.1415926
            xx=int(u*width/(2*3.1415926))
            yy=int(v*height/(3.1415926))
            if xx<0 or yy<0:
                print(xx,yy)
            img_out[-i+500][j+500]=img[yy][xx]
    return img_out 
~~~

效果：
![Yituosi-otto](https://user-images.githubusercontent.com/53858652/197933961-6d58b0d2-7f7a-40df-a589-88df8b5df331.png)


