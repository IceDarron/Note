Q:Java泛型。
===
>泛型的定义：泛型是JDK 1.5的一项新特性，它的本质是参数化类型（Parameterized Type）的应用，也就是说所操作的数据类型被指定为一个参数，在用到的时候在指定具体的类型。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口和泛型方法。
+ 泛型类
``` Java
class Pair<T> {  
    private T value;  
        public Pair(T value) {  
                this.value=value;  
        }  
        public T getValue() {  
        return value;  
    }  
    public void setValue(T value) {  
        this.value = value;  
    }  
}
```
+ 泛型接口
``` Java
interface Show<T,U>{  
    void show(T t,U u);  
}  
  
class ShowTest implements Show<String,Date>{  
    @Override  
    public void show(String str,Date date) {  
        System.out.println(str);  
        System.out.println(date);  
    }  
} 
```
+ 泛型方法
``` Java
Map<K, V>
```
+ 泛型变量
>类型限定在泛型类、泛型接口和泛型方法中都可以使用，不过要注意下面几点：
1.不管该限定是类还是接口，统一都使用关键字 extends。
2.可以使用&符号给出多个限定。
3.如果限定既有接口也有类，那么类必须只有一个，并且放在首位置。
``` Java
public static <T extends Object&Comparable&Serializable> T get(T t1,T t2)  
```