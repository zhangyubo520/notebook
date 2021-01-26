# String类

### 1、概述

```
	 * 1. public final class String implements java.io.Serializable, Comparable
	 *  > 不可被继承
	 *  > Serializable:标识接口，实现序列化机制的接口
	 *  > Comparable:实现对象比较大小的接口
	 *  > String是类，属于引用类型变量。
```

### 2、String不可变

```
String s1 = "hello";
String s2 = "hello";

System.out.println(s1.equals(s2));//true
System.out.println(s1 == s2);//true

s1 = "ahfal"
System.out.println(s2);//hello
```

### 3、String创建

```
1、使用字面量的方式
String s = "";

2、使用构造器
1、String s = new String("");
2、String s = new String(char[] c);
3、String s = new String(byte[] b);
4、String s = new String(StringBuffer);
5、String s = new String(StringBuilder);
```

### 4、String拼接

```
/*
	 *  String的连接操作
	 * 
	 * 1. 连接运算中，如果使用的都是字面量，则在常量池中声明此字符串或使用现有的字符串。
	 * 2. 连接运算中，如果使用的是变量，则需要在堆中重新开辟空间，保存此字符串的值。
	 * 3. 通过字符串调用intern()，返回此字符串在字符串常量池中的字面量。
	 */
	@Test
	public void test3(){
		String s1 = "java";
		String s2 = "hadoop";
		
		String s3 = "javahadoop";
		String s4 = "java" + "hadoop";
		String s5 = "java" + s2;
		String s6 = s1 + "hadoop";
		String s7 = s1 + s2;
		
		String s8 = s7.intern();
		String s9 = s5.intern();
		
		System.out.println(s3 == s4);//true
		System.out.println(s3 == s5);//false
		System.out.println(s3 == s6);//false
		System.out.println(s3 == s7);//false
		System.out.println(s5 == s6);//false
		System.out.println(s5 == s7);//false
		
		System.out.println(s3 == s8);//true
		System.out.println(s3 == s9);//true
		
	}
```

### 5、常用方法

查找类

```

	 * int length()：返回字符串的长度： return value.length
	 * char charAt(int index)： 返回某索引处的字符return value[index]
	 * boolean isEmpty()：判断是否是空字符串：return value.length == 0
	 * boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束 
	 * boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始 
	 * boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
	 * boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
	 * boolean equals(Object obj)：比较字符串的内容是否相同
	 * boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
	 * int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引 
	 * int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始 
	 * int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引 
	 * int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索 
	 * 注：indexOf和lastIndexOf方法如果未找到都是返回-1
```

操作类

```
 * String toLowerCase()：使用默认语言环境，将 String 中的所有字符转换为小写
 * String toUpperCase()：使用默认语言环境，将 String 中的所有字符转换为大写
 * String trim()：返回字符串的副本，忽略前导空白和尾部空白
 * String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。 
 * String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
 * String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的。 
 * String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串。 

```

### 6、String和其他的转换

基本类之间

```
        Integer s = Integer.parseInt("1312421");

        int i = Integer.parseInt("123124");

        String s2 = s.toString();

        String s3 = String.valueOf(s);
```



char[]

```
        char[] c = {'a','d','c','d'};
        
        String s4 = new String(c);

        char[] chars = s4.toCharArray();
```



byte[]

```
        byte[] b = {97,98,99,100};

        String s5 = new String(b,"utf-8");

        byte[] bytes = s5.getBytes("utf-8");
        
        说明：
     * ASCII:给26个英文大小写字母，0-9等都分配了对应的一个字节数值。比如：a --> 97   A--> 65
	 * GBK:兼容了ASCII(如果出现英文字母，0-9时，仍然使用1个字节存储），一个汉字使用2个字节存储
	 * UTF-8:兼容了ASCII(如果出现英文字母，0-9时，仍然使用1个字节存储），一个汉字使用3个字节存储
	 
```

### 7、