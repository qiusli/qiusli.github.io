---
layout: post
title: "Java Annotation学习笔记"
date: 2013-07-27 20:23
comments: true
categories: 
---
1. Annotation是什么
   Annotation是类，方法或字段的一种注解或元数据，当程序被JVM编译的时候，annotation会和类编译在一起，作为一种元数据去描述被修饰的数据
2. Annotation的定义 

<!-- more -->

``` java
package annotationApplication;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    public String name();

    public int age();
}
```
其中，Retention中的字段表明，该注解能在运行时通过反射得到。Target表明该注解用于何处。此处为类或者接口处可以使用。

3. Annotation的使用（类、接口范围）
annotation的定义
``` java
package annotationDefinition;

import annotationApplication.MyAnnotation;

@MyAnnotation(name = "Mikel Arteta", age = 29)
public class MyAnnotationUsage {
}
```
annotation的使用
``` java
package annotationDefinition;

import annotationApplication.MyAnnotation;

@MyAnnotation(name = "Mikel Arteta", age = 29)
public class MyAnnotationUsage {
}
```
下面是我定义的使用类注解的类的查找器：找到并输出注解类的annotation的value
``` java
package annotationDefinition;

import annotationApplication.MyAnnotation;
import java.lang.annotation.Annotation;

public class MyAnnotationFinder {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> myAnnotationUsage = Class.forName("annotationDefinition.MyAnnotationUsage");
        Annotation[] annotations = myAnnotationUsage.getAnnotations();
        for(Annotation annotation : annotations) {
            MyAnnotation annotation1 = (MyAnnotation)annotation;
            System.out.println(annotation1.name());
            System.out.println(annotation1.age());
        }
    }
}
```
运行结果
``` text
annotation's name value -- > Mikel Arteta
annotation's age value -- > 29
``` 
4. Annotation的使用（方法范围）
annotation的定义
``` java
package annotationApplication;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MethodAnnotation {
    public String methodName();
    public String methodReturnType();
}
```
下面是我定义的使用类注解的类的查找器：找到并输出注解类的annotation的value
``` java
package annotationDefinition;  
  
import annotationApplication.MethodAnnotation;  
  
import java.lang.annotation.Annotation;  
import java.lang.reflect.Method;  
  
public class MethodAnnotationFinder {  
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {  
        Class<?> clazz = Class.forName("annotationDefinition.MethodAnnotationUsage");  
        Method method = clazz.getDeclaredMethod("testMethod");  
        Annotation annotation = method.getAnnotation(MethodAnnotation.class);  
        MethodAnnotation annotation1 = (MethodAnnotation) annotation;  
        System.out.println("annotationed method name -- > " + annotation1.methodName());  
        System.out.println("annotationed method's return type -- > " + annotation1.methodReturnType());  
    }  
}  
```

运行结果：
``` text
annotationed method name -- > testMethod  
```
5. Annotation的使用（Parameter范围）
annotation的范围
``` java
package annotationApplication;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.PARAMETER)  
public @interface ParameterAnnotation {  
    public String parameterName();  
    public String parameterType();  
}  
```

annotation的使用
``` java
package annotationDefinition;  
  
import annotationApplication.ParameterAnnotation;  
  
public class ParameterAnnotationUsage {  
    public void testMethod(@ParameterAnnotation(parameterName = "pname", parameterType = "int") int value) {  
        value = 10;  
    }  
}  
```
下面是我定义的使用类注解的类的查找器：找到并输出注解类的annotation的value
``` java
package annotationDefinition;  
  
import annotationApplication.ParameterAnnotation;  
  
import java.lang.annotation.Annotation;  
import java.lang.reflect.Method;  
  
public class ParameterAnnotationFinder {  
  
    public void parameterAnnotationFinder() throws NoSuchMethodException {  
        Class<?> clazz = ParameterAnnotationUsage.class;  
        Method method = clazz.getMethod("testMethod", int.class);  
        Annotation[][] annotations = method.getParameterAnnotations();  
        //Method.getParameterAnnotations() method returns a two-dimensional Annotation array,  
        // containing an array of annotations for each method parameter.  
        Class<?>[] parameterTyeps = method.getParameterTypes();  
  
        int i = 0;  
        for (Annotation[] annotation : annotations) {  
            Class<?> parameterTyep = parameterTyeps[i++];  
            for (Annotation annotation1 : annotation) {  
                ParameterAnnotation parameterAnnotation1 = (ParameterAnnotation) annotation1;  
                System.out.println("annotation paraName : " + parameterAnnotation1.parameterName());  
                System.out.println("annotation paraType : " + parameterAnnotation1.parameterType());  
                System.out.println("parameter Type : " + parameterTyep);  
            }  
        }  
    }  
  
    public static void main(String[] args) throws NoSuchMethodException {  
        new ParameterAnnotationFinder().parameterAnnotationFinder();  
    }  
}  
```

运行结果：
``` text
annotation paraName : pname  
annotation paraType : int  
parameter Type : int  
```