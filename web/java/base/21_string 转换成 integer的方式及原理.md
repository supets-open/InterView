
## string 转换成 integer的方式及原理

### 答案
	
	1 如何将字串 String 转换成整数 int?
	
	A. 有两个方法:
	1). int i = Integer.parseInt([String]); 或
	i = Integer.parseInt([String],[int radix]);
	2). int i = Integer.valueOf(my_str).intValue();
	注: 字串转成 Double, Float, Long 的方法大同小异.
	
	2 如何将整数 int 转换成字串 String ?

    A. 有叁种方法:
     1.) String s = String.valueOf(i);
     2.) String s = Integer.toString(i);
     3.) String s = "" + i;