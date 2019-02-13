Q:Java重写equals、hashCode和compareTo方法
===
>什么时候需要重写equals方法？为什么重写equals方法，一定要重写HashCode方法？

### 何时需要重写equals()
当一个类有自己特有的“逻辑相等”概念（不同于对象身份的概念）。

### 设计equals()
+ 使用instanceof操作符检查“实参是否为正确的类型”。
+ 对于类中的每一个“关键域”，检查实参中的域与当前对象中对应的域值。
+ 对于非float和double类型的原语类型域，使用==比较；
+ 对于对象引用域，递归调用equals方法；
+ 对于float域，使用Float.floatToIntBits(afloat)转换为int，再使用==比较；
+ 对于double域，使用Double.doubleToLongBits(adouble) 转换为int，再使用==比较；
+ 对于数组域，调用Arrays.equals方法。

### 当改写equals()的时候，总是要改写hashCode()
根据一个类的equals方法（改写后），两个截然不同的实例有可能在逻辑上是相等的，但是，根据Object.hashCode方法，它们仅仅是两个对象。因此，违反了“相等的对象必须具有相等的散列码”。

### 设计hashCode()
+ 把某个非零常数值，例如17，保存在int变量result中；
+ 对于对象中每一个关键域f（指equals方法中考虑的每一个域）：
+ boolean型，计算(f ? 0 : 1);
+ byte,char,short型，计算(int);
+ long型，计算(int) (f ^ (f>>>32));
+ float型，计算Float.floatToIntBits(afloat);
+ double型，计算Double.doubleToLongBits(adouble)得到一个long，再执行long型的计算;
+ 对象引用，递归调用它的hashCode方法;
+ 数组域，对其中每个元素调用它的hashCode方法。
+ 将上面计算得到的散列码保存到int变量c，然后执行 result=37*result+c;
+ 返回result。

### 为什么重写equals方法，一定要重写HashCode方法？
>如果你重载了equals，比如说是基于对象的内容实现的，而保留hashCode的实现不变，那么很可能某两个对象明明是“相等”，而hashCode却不一样。这样，当你用其中的一个作为键保存到hashMap、hasoTable或hashSet中，再以“相等的”找另一个作为键值去查找他们的时候，则根本找不到。
使用HashMap，如果key是自定义的类，就必须重写hashcode()和equals()。
>而对于每一个对象，通过其hashCode()方法可为其生成一个整形值（散列码），该整型值被处理后，将会作为数组下标，存放该对象所对应的Entry（存放该对象及其对应值）。 equals()方法则是在HashMap中插入值或查询时会使用到。当HashMap中插入值或查询值对应的散列码与数组中的散列码相等时，则会通过equals方法比较key值是否相等，所以想以自建对象作为HashMap的key，必须重写该对象继承object的hashCode和equals方法。 
### 本来不就有hashcode()和equals()了么？干嘛要重写，直接用原来的不行么？
>HashMap中，如果要比较key是否相等，要同时使用这两个函数！因为自定义的类的hashcode()方法继承于Object类，其hashcode码为默认的内存地址，这样即便有相同含义的两个对象，比较也是不相等的，例如，生成了两个“羊”对象，正常理解这两个对象应该是相等的，但如果你不重写 hashcode（）方法的话，比较是不相等的！
>HashMap中的比较key是这样的，先求出key的hashcode(),比较其值是否相等，若相等再比较equals(),若相等则认为他们是相等的。若equals()不相等则认为他们不相等。如果只重写hashcode()不重写equals()方法，当比较equals()时只是看他们是否为同一对象（即进行内存地址的比较）,所以必定要两个方法一起重写。HashMap用来判断key是否相等的方法，其实是调用了HashSet判断加入元素是否相等。
>一般来说，如果你要把一个类的对象放入容器中，那么通常要为其重写equals()方法，让他们比较地址值而不是内容值。特别地，如果要把你的类的对象放入散列中，那么还要重写hashCode()方法；要放到有序容器中，还要重写compareTo()方法。
>equals()相等的两个对象，hashcode()一定相等；
>equals（）不相等的两个对象，却并不能证明他们的hashcode()不相等。换句话说，equals()方法不相等的两个对象，hashcode()有可能相等。（我的理解是由于哈希码在生成的时候产生冲突造成的）。
>反过来：hashcode()不等，一定能推出equals()也不等；hashcode()相等，equals()可能相等，也可能不等