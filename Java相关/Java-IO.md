Q:Java IO
===
![Image text](https://github.com/IceDarron/Note/blob/master/Image/java_io.png)

### File类
> File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。 File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。  

### 流的概念和作用
> 流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。 

### IO流的分类
+ 根据处理数据类型的不同分为：字符流和字节流
+ 根据数据流向不同分为：输入流和输出流

### 字符流和字节流
> 字符流的由来： 因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。 字节流和字符流的区别：
读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。
结论：只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。

### 输入流和输出流
> 对输入流只能进行读操作，对输出流只能进行写操作，程序中需要根据待传输数据的不同特性而使用不同的流。

### Java IO流对象
> 输入字节流InputStreamIO 中输入字节流的继承图可见上图，可以看出：
+ InputStream 是所有的输入字节流的父类，它是一个抽象类。
+ ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。PipedInputStream 是从与其它线程共用的管道中读取数据。
+ ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）。
> 输出字节流OutputStreamIO 中输出字节流的继承图可见上图，可以看出：
+ OutputStream 是所有的输出字节流的父类，它是一个抽象类。
+ ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。PipedOutputStream 是向与其它线程共用的管道中写入数据。
+ ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流。

### 字符输入流Reader
+ Reader 是所有的输入字符流的父类，它是一个抽象类。
+ CharReader、StringReader是两种基本的介质流，它们分别将Char数组、String中读取数据。PipedReader是从与其它线程共用的管道中读取数据。
+ BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。
+ FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。
+ InputStreamReader是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。我们可以从这个类中得到一定的技巧。Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致。

### 字符输出流Writer
+ Writer 是所有的输出字符流的父类，它是一个抽象类。
+ CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。PipedWriter 是向与其它线程共用的管道中写入数据。
+ BufferedWriter 是一个装饰器为Writer 提供缓冲功能。
+ PrintWriter 和PrintStream 极其类似，功能和使用也非常相似。
+ OutputStreamWriter 是OutputStream 到Writer 转换的桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。功能和使用和OutputStream 极其类似。

### 字符流与字节流转换
InputStreamReader:字节到字符的桥梁
OutputStreamWriter:字符到字节的桥梁
这两个流对象是字符体系中的成员，它们有转换作用，本身又是字符流，所以在构造的时候需要传入字节流对象进来。

### RandomAccessFile
> 该对象并不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。 该对象特点：
只能操作文件，所以构造函数接收两种类型的参数：a.字符串文件路径；b.File对象。
既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模式(r,rw)

### 提供部分不完整代码，例如流关闭应该全部在finally里处理
```Java
public class test {
    
    public static void main(String[] args){
        try {
            char[] buffer = new char[1024];
            FileReader fr = new FileReader("testr.txt");
            FileWriter fw = new FileWriter("testw.txt");
            int numberRead = 0;
        
            while ((numberRead = fr.read(buffer))!=-1) {
                fw.write(buffer, 0, numberRead);
            }
            fr.close();
            fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        try {
            int index = 0;
            byte[] buffer = new byte[1024];
            File fileOrign = new File("testo.txt");
            File fileCopy = new File("testc.txt");
            InputStream is = new FileInputStream(fileOrign);
            OutputStream os = new FileOutputStream(fileCopy);
        
            while ((index = is.read(buffer)) != -1) {
                os.write(buffer);
            }
            is.close();
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        try {
            FileReader fr = new FileReader("test.txt");
            BufferedReader br = new BufferedReader(fr);
            String str = "";
        
            while ((str = br.readLine()) != null) {
                System.out.println(str);
            }
            fr.close();
        } catch (IOException e) {
            e.printStackTrace();
        } 
    }
}
```