---
title: Lombok
date:  2019-11-28 12:49:33 +0800
category: Reference
tags: Developer_Tools
excerpt: Lombok介绍、使用方法和总结
---

## Lombok背景介绍
官方介绍如下：

> Project Lombok makes java a spicier language by adding 'handlers' that know how to build and compile simple, boilerplate-free, not-quite-java code.

大致意思是Lombok通过增加一些“处理程序”，可以让java变得简洁、快速。
## Lombok使用方法
Lombok能以简单的注解形式来简化java代码，提高开发人员的开发效率。例如开发中经常需要写的javabean，都需要花时间去添加相应的getter/setter，也许还要去写构造器、equals等方法，而且需要维护，当属性多时会出现大量的getter/setter方法，这些显得很冗长也没有太多技术含量，一旦修改属性，就容易出现忘记修改对应方法的失误。

Lombok能通过注解的方式，在编译时自动为属性生成构造器、getter/setter、equals、hashcode、toString方法。出现的神奇就是在源码中没有getter和setter方法，但是在编译生成的字节码文件中有getter和setter方法。这样就省去了手动重建这些代码的麻烦，使代码看起来更简洁些。

Lombok的使用跟引用jar包一样，可以在官网（https://projectlombok.org/download ）下载jar包，也可以使用maven添加依赖：

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.20</version>
        <scope>provided</scope>
    </dependency>

接下来我们来分析Lombok中注解的具体用法。

## @ Data
@ Data注解在类上，会为类的所有属性自动生成setter/getter、equals、canEqual、hashCode、toString方法，如为final属性，则不会为该属性生成setter方法。

官方实例如下：

    import lombok.AccessLevel;
    import lombok.Setter;
    import lombok.Data;
    import lombok.ToString;
    
    @Data public class DataExample {
      private final String name;
      @Setter(AccessLevel.PACKAGE) private int age;
      private double score;
      private String[] tags;
      
      @ToString(includeFieldNames=true)
      @Data(staticConstructor="of")
      public static class Exercise<T> {
        private final String name;
        private final T value;
      }
    }
    

如不使用Lombok，则实现如下：

    import java.util.Arrays;
    
    public class DataExample {
      private final String name;
      private int age;
      private double score;
      private String[] tags;
      
      public DataExample(String name) {
        this.name = name;
      }
      
      public String getName() {
        return this.name;
      }
      
      void setAge(int age) {
        this.age = age;
      }
      
      public int getAge() {
        return this.age;
      }
      
      public void setScore(double score) {
        this.score = score;
      }
      
      public double getScore() {
        return this.score;
      }
      
      public String[] getTags() {
        return this.tags;
      }
      
      public void setTags(String[] tags) {
        this.tags = tags;
      }
      
      @Override public String toString() {
        return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
      }
      
      protected boolean canEqual(Object other) {
        return other instanceof DataExample;
      }
      
      @Override public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof DataExample)) return false;
        DataExample other = (DataExample) o;
        if (!other.canEqual((Object)this)) return false;
        if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
        if (this.getAge() != other.getAge()) return false;
        if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
        if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
        return true;
      }
      
      @Override public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        final long temp1 = Double.doubleToLongBits(this.getScore());
        result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
        result = (result*PRIME) + this.getAge();
        result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
        result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
        return result;
      }
      
      public static class Exercise<T> {
        private final String name;
        private final T value;
        
        private Exercise(String name, T value) {
          this.name = name;
          this.value = value;
        }
        
        public static <T> Exercise<T> of(String name, T value) {
          return new Exercise<T>(name, value);
        }
        
        public String getName() {
          return this.name;
        }
        
        public T getValue() {
          return this.value;
        }
        
        @Override public String toString() {
          return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
        }
        
        protected boolean canEqual(Object other) {
          return other instanceof Exercise;
        }
        
        @Override public boolean equals(Object o) {
          if (o == this) return true;
          if (!(o instanceof Exercise)) return false;
          Exercise<?> other = (Exercise<?>) o;
          if (!other.canEqual((Object)this)) return false;
          if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
          if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
          return true;
        }
        
        @Override public int hashCode() {
          final int PRIME = 59;
          int result = 1;
          result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
          result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
          return result;
        }
      }
    }
    

## @ Getter/@ Setter

如果觉得@ Data太过残暴（因为@ Data集合了@ ToString、@ EqualsAndHashCode、@ Getter/@ Setter、@ RequiredArgsConstructor的所有特性）不够精细，可以使用@ Getter/@ Setter注解，此注解在属性上，可以为相应的属性自动生成Getter/Setter方法，示例如下：

    import lombok.AccessLevel;
    import lombok.Getter;
    import lombok.Setter;
    
    public class GetterSetterExample {
    
      @Getter @Setter private int age = 10;
      
      @Setter(AccessLevel.PROTECTED) private String name;
      
      @Override public String toString() {
        return String.format("%s (age: %d)", name, age);
      }
    }
    

如果不使用Lombok：

    public class GetterSetterExample {
    
      private int age = 10;
    
      private String name;
      
      @Override public String toString() {
        return String.format("%s (age: %d)", name, age);
      }
      
      public int getAge() {
        return age;
      }
      
      public void setAge(int age) {
        this.age = age;
      }
      
      protected void setName(String name) {
        this.name = name;
      }
    }
    
	

## @ NonNull

该注解用在属性或构造器上，Lombok会生成一个非空的声明，可用于校验参数，能帮助避免空指针。

示例如下：

    import lombok.NonNull;
    
    public class NonNullExample extends Something {
      private String name;
      
      public NonNullExample(@NonNull Person person) {
        super("Hello");
        this.name = person.getName();
      }
    }
	

不使用Lombok：

    import lombok.NonNull;
    
    public class NonNullExample extends Something {
      private String name;
      
      public NonNullExample(@NonNull Person person) {
        super("Hello");
        if (person == null) {
          throw new NullPointerException("person");
        }
        this.name = person.getName();
      }
    }
	

## @ Cleanup
该注解能帮助我们自动调用close()方法，很大的简化了代码。

示例如下：

    import lombok.Cleanup;
    import java.io.*;
    
    public class CleanupExample {
      public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      }
    }
	

如不使用Lombok，则需如下：

    import java.io.*;
    
    public class CleanupExample {
      public static void main(String[] args) throws IOException {
        InputStream in = new FileInputStream(args[0]);
        try {
          OutputStream out = new FileOutputStream(args[1]);
          try {
            byte[] b = new byte[10000];
            while (true) {
              int r = in.read(b);
              if (r == -1) break;
              out.write(b, 0, r);
            }
          } finally {
            if (out != null) {
              out.close();
            }
          }
        } finally {
          if (in != null) {
            in.close();
          }
        }
      }
    }
	

## @ EqualsAndHashCode
默认情况下，会使用所有非静态（non-static）和非瞬态（non-transient）属性来生成equals和hasCode，也能通过exclude注解来排除一些属性。

示例如下：

    import lombok.EqualsAndHashCode;
    
    @EqualsAndHashCode(exclude={"id", "shape"})
    public class EqualsAndHashCodeExample {
      private transient int transientVar = 10;
      private String name;
      private double score;
      private Shape shape = new Square(5, 10);
      private String[] tags;
      private int id;
      
      public String getName() {
        return this.name;
      }
      
      @EqualsAndHashCode(callSuper=true)
      public static class Square extends Shape {
        private final int width, height;
        
        public Square(int width, int height) {
          this.width = width;
          this.height = height;
        }
      }
    }
	

## @ ToString
类使用@ ToString注解，Lombok会生成一个toString()方法，默认情况下，会输出类名、所有属性（会按照属性定义顺序），用逗号来分割。

通过将includeFieldNames参数设为true，就能明确的输出toString()属性。这一点是不是有点绕口，通过代码来看会更清晰些。
使用Lombok的示例：

    import lombok.ToString;
    
    @ToString(exclude="id")
    public class ToStringExample {
      private static final int STATIC_VAR = 10;
      private String name;
      private Shape shape = new Square(5, 10);
      private String[] tags;
      private int id;
      
      public String getName() {
        return this.getName();
      }
      
      @ToString(callSuper=true, includeFieldNames=true)
      public static class Square extends Shape {
        private final int width, height;
        
        public Square(int width, int height) {
          this.width = width;
          this.height = height;
        }
      }
    }
	

不使用Lombok的示例如下：

    import java.util.Arrays;
    
    public class ToStringExample {
      private static final int STATIC_VAR = 10;
      private String name;
      private Shape shape = new Square(5, 10);
      private String[] tags;
      private int id;
      
      public String getName() {
        return this.getName();
      }
      
      public static class Square extends Shape {
        private final int width, height;
        
        public Square(int width, int height) {
          this.width = width;
          this.height = height;
        }
        
        @Override public String toString() {
          return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
        }
      }
      
      @Override public String toString() {
        return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
      }
    }
	

## @ NoArgsConstructor, @ RequiredArgsConstructor and @ AllArgsConstructor
无参构造器、部分参数构造器、全参构造器。Lombok没法实现多种参数构造器的重载。

    Lombok示例代码如下：
    
    import lombok.AccessLevel;
    import lombok.RequiredArgsConstructor;
    import lombok.AllArgsConstructor;
    import lombok.NonNull;
    
    @RequiredArgsConstructor(staticName = "of")
    @AllArgsConstructor(access = AccessLevel.PROTECTED)
    public class ConstructorExample<T> {
      private int x, y;
      @NonNull private T description;
      
      @NoArgsConstructor
      public static class NoArgsExample {
        @NonNull private String field;
      }
    }

不使用Lombok的示例如下：

    public class ConstructorExample<T> {
      private int x, y;
      @NonNull private T description;
      
      private ConstructorExample(T description) {
        if (description == null) throw new NullPointerException("description");
        this.description = description;
      }
      
      public static <T> ConstructorExample<T> of(T description) {
        return new ConstructorExample<T>(description);
      }
      
      @java.beans.ConstructorProperties({"x", "y", "description"})
      protected ConstructorExample(int x, int y, T description) {
        if (description == null) throw new NullPointerException("description");
        this.x = x;
        this.y = y;
        this.description = description;
      }
      
      public static class NoArgsExample {
        @NonNull private String field;
        
        public NoArgsExample() {
        }
      }
    }