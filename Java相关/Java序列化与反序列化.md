Q:Java序列化与反序列化
===
>  Java序列化就是把对象转换成字节序列，而Java反序列化就是把字节序列还原成Java对象。
采用Java序列化与反序列化技术，一是可以实现数据的持久化，在MVC模式中很是有用；二是可以对象数据的远程通信。

### 序列化与反序列化的实现
#### JDK类库中序列化API
> java.io.ObjectOutputStream：表示对象输出流，它的writeObject(Object obj)方法可以对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中。
java.io.ObjectInputStream：表示对象输入流，它的readObject()方法源输入流中读取字节序列，再把它们反序列化成为一个对象，并将其返回。

#### 实现Serializable或Externalizable接口
方法一：实现了Serializable接口，则可以按照以下方式进行序列化和反序列化
ObjectOutputStream采用默认的序列化方式，对对象的非transient的实例变量进行序列化。
ObjcetInputStream采用默认的反序列化方式，对对象的非transient的实例变量进行反序列化。
方法二：若类仅仅实现了Serializable接口，并且还定义了readObject(ObjectInputStream in)和writeObject(ObjectOutputSteam out)，则采用以下方式进行序列化与反序列化。
ObjectOutputStream调用对象的writeObject(ObjectOutputStream out)的方法进行序列化。
ObjectInputStream会调用对象的readObject(ObjectInputStream in)的方法进行反序列化。
方法三：若类实现了Externalnalizable接口，且类必须实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法。
