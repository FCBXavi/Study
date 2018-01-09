#String、StringBuffer和StringBuilder
##String
String是不可变的，底层使用final数组实现，每次对String进行操作都会生成新的String对象，所以经常改变的字符串最好不要用String。

##StringBuffer
线程安全，大部分方法是synchronized的，可以被修改的，但多个线程同时操作时速度会慢。

##StringBuilder
与StringBuffer类似，但是非线程安全的，单线程使用效率高。