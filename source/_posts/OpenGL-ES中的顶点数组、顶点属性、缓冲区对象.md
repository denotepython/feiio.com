title: "OpenGL ES中的顶点数组、顶点属性、缓冲区对象"
date: 2015-05-31 17:29:23
categories: 游戏开发
tags:  OpenGLES
---
##什么是顶点数据？
在计算机中图元的位置是通过x,y,z,w来存储或颜色数据是通过RGBA的数组格式存储的,然后通过多个点来进行图元装配和光栅化出图形.比如三角形3个点使用X,Y,Z表示如下:
<!--more-->
<pre>
  GLfloat vertex[]={
			0.0f,1.0f,1.0f, //x,y,z
			1.0f,0.5f,1.0f,
			0.0f,0.5f,1.0f			
			}

</pre>

## 顶点属性数组
顶点属性数组对象就是存储在应用程序缓冲区的顶点数据,又分两种方式来分配和存储顶点属性数组：结构数组和数据结构,结构数组就是所有顶点数据使用一个数组来存储数据,把位置、颜色、法线、纹理UV坐标等数据放进一个数组结构中.数组结构就是把每个数据通过不同的数组结构存放顶点数据.结构数组的效率要高于数组结构,因为在通过结构数组的方式访问顶点数据是按照在内存中的顺序来访问的.缺点是修改了其中的数据必须重新更新已经保存在缓冲区中数据的跨距.


```
#define VERTEX_POS_INDX 0
#define VERTEX_POS_SIZE 3

#define VERTEX_COLOR_INDX 1
#define VERTEX_COLOR_SIZE 4

#define VERTEX_ATTRIB_SIZE (VERTEX_POS_SIZE+VERTEX_COLOR_SIZE)

GLfloat vertex[]={
                        0.0f,1.0f,1.0f, //x,y,z
			1.0f,0.0f,0.0f,1.0f, //RGBA
			1.0f,0.5f,1.0f,
			0.0f,1.0f,0.0f,1.0f,
			0.0f,0.5f,1.0f,
			0.0f,0.0f,1.0f,1.0f
		 }
//设置顶点位置属性数据数组
glVertexAttribPointer(VERTEX_POS_INDX,VERTEX_POS_SIZE,GL_FLOAT,GL_FALSE,VERTEX_ATTRIB_SIZE*sizeof(GLfloat),vertex);
//设置颜色属性数据数组
glVertexAttribPointer(VERTEX_COLOR_INDX,VERTEX_COLOR_SIZE,GL_FLOAT,GL_FALSE,(VERTEX_ATTRIB_SIZE*sizeof(GLfloat),vertex+VERTEX_POS_SIZE));
//启用顶点数组数据
glEableVertexAttribArray(VERTEX_POS_INDE);
glEnableVertexAttribArray(VERTEX_COLOR_INDE);
//根据顶点数据绘制三角形
glDrawArrays(GL_TRIANGLES,0,3);
//禁用顶点数组数据
glDisableVertexAttribArray(VERTEX_POS_INDE);
glDisableVertexAttribArray(VERTEX_COLOR_INDE);
```


## 顶点缓冲区对象
简单点说就是把图形顶点数据和索引数据缓存在图形设备内存GPU中,改善每次绘制图形都要复制应用程序中的顶点数据.缓冲区顶点数据对象分为两种:1.图元索引数据(GL_ELEMENT_ARRAY_BUFFER）和顶点数据(GL_ARRAY_BUFFER).

###使用缓冲冲区顶点对象的步骤：
	1. 分配缓冲区对象:glGenBuffer
	2. 绑定缓冲区对象:glBindBuffer,第一次调用为分配空间，以后调用都是使用缓冲区对象.
	3. 存储数据到缓冲区对象: glBufferData
	4. 删除缓冲区对象:glDeleteBuffers,在不需要使用缓冲区对象的时候删除.

拿上边三角形的顶点数组数据来说,代码如下:
```
	......................
	
	GLunit buffer[2];
	glGenBuffer(2,&buffer);
	//绑定顶点数据到缓冲区对象
	glBindBuffer(GL_ARRAY_BUFFER,buffer[0]);
	glBufferData(GL_ARRAY_BUFFER,sizeof(vertex),vertex,GL_STATIC_DRAW);
	//绑定顶点索引数据到缓冲区对象
	GLShort indices[3]={0,1,2};
	glBindBuffer(GL_ELEMENTS_ARRAY_BUFFER,buffer[1]);
	glBufferData(GL_ELEMENTS_ARRAY_BUFFER,sizeof(GLShort)*3,indices,GL_STATIC_DRAW);

	//使用缓冲区对象
	glBindBuffer(GL_ARRAY_BUFFER,buffer[0]);
	glEnableVertexAttibArray(VERTEX_POS_INDX);
	glVertexAttribPointer(VERTEX_POS_INDX,VERTEX_POS_SIZE,GL_FLOAT,GL_FALSE,(VERTEX_ATTRIB_SIZE*sizeof(GLfloat)),(const void*)0);
	
	glVertexAttribPointer(VERTEX_COLOR_INDX,VERTEX_COLOR_SIZE,GL_FLOAT,GL_FALSE,(VERTEX_ATTRIB_SIZE*sizeof(GLfloat),(cons void*)(VERTEX_POS_SIZE*sizeof(GLfloat)));
	
	................

```
glDeleteBuffers删除已经存在的缓冲区对象

##顶点数组对象（VAO）
这个特性是在opengl es3.0 中引入的,提供包含在顶点数组/顶点缓冲区对象配置之间切换所需要的所有状态的单一对象.只需要在设置缓冲区数据的时候glGenVertexArrays一个顶点数组对象ID，然后调用glBindVertexArray绑定已经生成数组对象的ID,在绘制图元的时候调用glBindVertexArray切换相应缓冲区内的数据并进行绘制.glDeleteVertexArray删除一个或者多个顶点数组对象.

## 映射缓冲区对象
应用程序设置缓冲区对象数据调用glBufferData的时候,可以将缓冲区对象数据存储映射到应用程序的内存地址空间,优点：减少应用程序的内存占用，只需要存储数据的一个副本.避免数据复制步骤通过映射出的GPU存储缓冲区对象的内存地址.
调用glMapBufferRange返回缓冲区存储数据范围的指针,出现错误则反悔NULL，glUnmapBuffer取消之前缓冲区映射.

##着色器中如何关联顶点数据

* 在顶点着色器源代码中使用layout(location=N) in限定符来获取顶点属性的值
* 使用glBindAtttibLocation绑定索引到着色器代码变量.

--------EOF--------