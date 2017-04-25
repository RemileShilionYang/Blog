---
title: Java的反射机制
tags: Java,Reflect
---


Java的反射机制特别适合大型项目尤其是多功能项目的开发。因为它极大的减少了编译时静态加载类的时间并且减少了初期内存的消耗。


### 为什么要有反射机制
__1.在涉及某些应用程序时，我们往往需要动态升级以增加修改功能，而在静态编译体系中一切升级操作都需要对源代码进行更改，这就意味着每升一次级就要对整个源代码编译一次。小程序编译还好说，但如果是大型程序的话，编译一次要几个小时甚至几天才能完成，那这样显然纯静态的编程方式不太适合。而Java也是一门静态语言，为了弥补这个缺点，就有了反射机制。__
__2.Java更多的情况是应用于服务端程序，而服务器上的程序有个很重要的特点就是不能关掉；即使关掉，也要提前给用户通知。那么，如果此时要对系统进行升级，既要保证体系内其他功能正常运行，还要保证升级措施，此时就可以用反射来实现。__
__3.使程序更加清晰，将主程序和功能程序分离开，便于后期管理。__

### 反射机制的应用
__1.杀毒软件病毒库的在线升级__
__2.JDBC数据桥的构建(*Class.forName("数据库驱动包类名称")*)__
__3.应用程序功能的增加__
__4.分析泛型的实质__
__5.分析程序，找bug__

### 反射机制有三种实现方式
__1.得到类类型，同时加载类*Class class = Class.forName("SomethingName")*__
__2.已知类对象，由对象得到其所属类*Class class = new Demo().getClass()*__
__3.直接由类得到其类类型*Class class = Demo.Class*__

### 反射的类
__1.Class类，用来处理类级别的反射；__
__2.Field类，用来处理成员变量级的反射；__
__3.Method类，用来处理方法级反射的类；__
__4.Constructor类，用来处理构造器反射的类；__

### 反射的小例子（以管理系统为例）
```javascript?linenums
import java.lang.reflect.Method;
import java.util.Scanner;

class Main {
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public static void main(String[] args) {
		System.out.println("||Add||   ||Remove||   ||Sort||   ||Change||   ||Seek||");
		System.out.print("Input your choose:");
		Scanner scanner = new Scanner(System.in);
		String string = scanner.nextLine();
		scanner.close();
		try{
			Class class1 = Class.forName(string);     //得到类同时加载类
			Method method = class1.getDeclaredMethod("command", String.class);
			method.invoke(class1.newInstance(), "something");
		}
		catch(Exception e){
			e.printStackTrace();
		}
	}
}

class Add implements Common{
	public void command(String string){
		System.out.println("Add Completed!");
	}
}

class Remove implements Common{
	public void command(String string){
		System.out.println("Remove completed!");
	}
}

class Sort implements Common{
	public void command(String string){
		System.out.println("Sort completed!");
	}
}

class Change implements Common{
	public void command(String string){
		System.out.println("Change completed!");
	}
}

class Seek implements Common{
	public void command(String string){
		System.out.println("Seek completed!");
	}
}

interface Common {
	public void command(String string);
}
```

### 透过反射看泛型
__Java中的泛型本质作用是为了防止编译阶段的输入错误、类型引用错误、参数错误。而在编译之后，Java中就不存在泛型的概念了。这里，可以用反射加以验证。__
```javascript?linenums
import java.lang.reflect.Method;
import java.util.ArrayList;

public class Test {
	@SuppressWarnings({ "rawtypes", "unchecked" })
	public static void main(String[] args){
		ArrayList<String> arrayList = new ArrayList<String>();
		arrayList.add("string");
		System.out.println(arrayList.size());
		Class class1 = arrayList.getClass();
		try {
			Method method = class1.getMethod("add", Object.class);
			method.invoke(arrayList, 100);
			System.out.println(arrayList.size());
		}
		catch (Exception e){
			e.printStackTrace();
		}
		ArrayList arrayList2 = new ArrayList();
		Class class2 = arrayList2.getClass();
		System.out.println(class1 == class2);
	}
}
```
__结果显示为1、2、true，这就表明通过反射，int类型的值被成功添加到最初泛型为String的集合当中，而最后的true也表示编译后两个集合类类型是相同的，不存在泛型。__
