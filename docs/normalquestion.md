1. java中String的“==”与equals区别？

   ```
   “==”比较的是内存地址
   “equals”比较的内容
   例子：
   String str1 = "a"+"b"+"c";
   String str2 = "abc";
   String str3 = new String ("abc");
   str1==str2;
   str1!=str3;
   str2!=str3;
   三个变量的hashcode相等的，都在堆内存中能找到对应的映射。
   java中String 存在常量池中，String str1 = "a"+"b"+"c"会在常量池中产生四个对象，a,b,c,abc,此时str1会指向常量池中的"abc",同时在执行完String str1 = "a"+"b"+"c"后会在堆内存中的对象映射表中会对应一个对象"abc"，此时常量池中有一个abc堆内存中也有个abc。在执行String str2 = "abc";时会首先去常量池中找,也就是str2会指向常量池中的abc。在执行String str3 = new String ("abc");会直接到堆内存中找
   
   String 的equals比较的是内存序列，及字符串的每一个字符。
   ```

   

2. String为什么是不可变的？

   final类型，长度不能修改