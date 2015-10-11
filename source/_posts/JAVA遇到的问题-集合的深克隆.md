title: "集合的深克隆"
date: 2015-04-28 12:36:30
tags: [Java]
categories: Java遇到的问题
description: 什么是深克隆？JAVA类如何深克隆？集合如何深克隆？附加小菜：Marker Interface
---
#什么是深克隆
##浅克隆
- 被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。

换言之，浅复制仅仅复制所考虑的对象，而不复制它所引用的对象
```java
ClassA  a = new ClassA();
ClassA shallowClone = a;//浅拷贝
```
浅拷贝中，如果改变了a指向的对象，会影响shallowClone指向的对象
![浅拷贝例子](http://askingwindy-gitcafe.qiniudn.com/浅拷贝.png)
为了避免这种情况，需要进行深克隆

##深克隆
- 被复制对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。

换言之，深复制把要复制的对象所引用的对象都复制了一遍。
```java
ClassA  a = new ClassA();
ClassA shallowClone = a.clone();//浅拷贝
```
注意，这里可以调用`a.clone()`方法，那么`ClassA`必须实现了`Cloneable`接口
![深拷贝例子](http://askingwindy-gitcafe.qiniudn.com/深拷贝.png)
##Cloneable接口
- Cloneable接口里面没有任何的方法，它用来指明一个类可以逐位复制一个对象——Marker Interface（标识接口，没有定义任何的方法，如Cloneable和Serializable接口）
	- 如果你试图对一个没有实现Cloneable接口的类调用clone()方法，一个CloneNotSupportedException 就会抛出。
	- 在复制时，被复制的对象的构造器并没有被调用。复制对象就是原来对象的拷贝
- 需要override的clone()方法是属于Object类的。但是Object类的clone()方法访问权限是：protected
- clone()产生了一个调用它的对象的复制;只有实现了Cloneable接口的类才可以被复制

### 如何让一个类实现Cloneable接口
1.在派生类中覆盖基类的clone()方法，并声明为public()Object类中的clone()方法为protected的)。
2.在派生类的clone()方法中，调用super.clone()
3.在派生类中实现Cloneable接口
```java
public class Student implements Cloneable{
	private String name;
	private Teacher teacher;
	
	@Override
    public Student  clone() throws CloneNotSupportedException {
	    //手动对每一个成员变量进行复制
	    Student o = (Student)super.clone();//必不可少的一句话
        o.name = (String)this.name.clone();
        o.teacher = (Teacher)this.teacher.clone();//这里对Teacher类进行了克隆
        return o;
    }
}
```
下面的例子说明了如何对一个类进行深拷贝:
```java
Student s1 = new Student();
Student s2 = s1.clone();
```
同时，我们注意到，Student类里包含了一个Teacher类，在对Student类进行clone()方法复写时，我们需要对Teacher类进行复写：
```java
public class Teacher implements Cloneable{
	private String name;
	
	@Override
    public Teacher clone() throws CloneNotSupportedException {
	    Teacher o = (Teacher )super.clone();
        o.name = (String)this.name.clone();
        return o;
    }
}
```
如果没有对Teacher没有实现Cloneable接口，在JVM虚拟堆里面的情况：
![嵌套类的浅拷贝](http://askingwindy-gitcafe.qiniudn.com/嵌套类的浅拷贝.png)
#集合的深克隆
集合的模板如果是类，这个类必须实现了Cloneable接口（如上述的Student类）
##List深克隆：循环复制(最方便)
```java
#已经有一个Student的list:stuList
List<Student> copyList = new ArrayList<>();
for(Student stu : stuList){
	copyList.add(stu.clone())；
}
```
***
#遇到的问题
项目中遇到如下问题：
实现功能：有一个复杂的类Edge，得到List<Edge>后，希望调用另外一个方法删除Edge e = 特定条件的类
##原始代码
```java
List<Edge> edgeList = A.getEdgeList();
for(Edge e : edgeList){
	if(e.equals(new Edge(d)){
		func(e);//func()里面可能会删除这个e
	}	
}
```
##错误信息
在运行时，会出现Java Concurrent Modification Exception Error 错误
##解决方法

```java
//进行深拷贝
List<Edge> edgeList = new ArrayList<>();
for(Edge e: A.getEdgeList()){
	edgeList.add(e.clone());
}
//对edgeList的操作不会影响A.getEdgeList
for(Edge e : edgeList){
	if(e.equals(new Edge(d)){
		func(e);//func()里面可能会删除这个e
	}	
}
```