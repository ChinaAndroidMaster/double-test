# double-test
关于java中Double类型的运算精度问题


如果我们编译运行下面这个程序会看到什么？
public class Test{
    public static void main(String args[]){
        System.out.println(0.05+0.01);
        System.out.println(1.0-0.42);
        System.out.println(4.015*100);
        System.out.println(123.3/100);
    }
};
你没有看错！结果确实是
0.060000000000000005
0.5800000000000001
401.49999999999994
1.2329999999999999
Java中的简单浮点数类型float和double不能够进行运算。不光是Java，在其它很多编程语言中也有这样的问题。在大多数情况下，计算的结果是准确的，但是多试几次（可以做一个循环）就可以试出类似上面的错误。现在终于理解为什么要有BCD码了。
这个问题相当严重，如果你有9.999999999999元，你的计算机是不会认为你可以购买10元的商品的。
在有的编程语言中提供了专门的货币类型来处理这种情况，但是Java没有。现在让我们看看如何解决这个问题。
 
四舍五入
我们的第一个反应是做四舍五入。Math类中的round方法不能设置保留几位小数，我们只能象这样（保留两位）：
public double round(double value){
    return Math.round(value*100)/100.0;
}
非常不幸，上面的代码并不能正常工作，给这个方法传入4.015它将返回4.01而不是4.02，如我们在上面看到的
4.015*100=401.49999999999994
因此如果我们要做到精确的四舍五入，不能利用简单类型做任何运算
java.text.DecimalFormat也不能解决这个问题：
System.out.println(new java.text.DecimalFormat("0.00").format(4.025));
输出是4.02
 
BigDecimal
在《Effective Java》这本书中也提到这个原则，float和double只能用来做科学计算或者是工程计算，在商业计算中我们要用 java.math.BigDecimal。BigDecimal一共有4个够造方法，我们不关心用BigInteger来够造的那两个，那么还有两个，它们是：
BigDecimal(double val) 
          Translates a double into a BigDecimal. 
BigDecimal(String val) 
          Translates the String repre sentation of a BigDecimal into a BigDecimal.
上面的API简要描述相当的明确，而且通常情况下，上面的那一个使用起来要方便一些。我们可能想都不想就用上了，会有什么问题呢？等到出了问题的时候，才发现上面哪个够造方法的详细说明中有这么一段：
Note: the results of this constructor can be somewhat unpredictable. One might assume that new BigDecimal(.1) is exactly equal to .1, but it is actually equal to .1000000000000000055511151231257827021181583404541015625. This is so because .1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the long value that is being passed in to the constructor is not exactly equal to .1, appearances nonwithstanding. 
The (String) constructor, on the other hand, is perfectly predictable: new BigDecimal(".1") is exactly equal to .1, as one would expect. Therefore, it is generally recommended that the (String) constructor be used in preference to this one.
原来我们如果需要精确计算，非要用String来够造BigDecimal不可！在《Effective Java》一书中的例子是用String来够造BigDecimal的，但是书上却没有强调这一点，这也许是一个小小的失误吧。
 
解决方案
现在我们已经可以解决这个问题了，原则是使用BigDecimal并且一定要用String来够造。
但是想像一下吧，如果我们要做一个加法运算，需要先将两个浮点数转为String，然后够造成BigDecimal，在其中一个上调用add方法，传入另一个作为参数，然后把运算的结果（BigDecimal）再转换为浮点数。你能够忍受这么烦琐的过程吗？下面我们提供一个工具类Arith来简化操作。它提供以下静态方法，包括加减乘除和四舍五入：
public static double add(double v1,double v2)
public static double sub(double v1,double v2)
public static double mul(double v1,double v2)
public static double div(double v1,double v2)
public static double div(double v1,double v2,int scale)
public static double round(double v,int scale)