

##Checked异常和Runtime异常体系

java异常被分为两大类：Checked异常和Runtime异常（运行时异常）。

所有RuntimeException类及其子类的实例被称为Runtime异常，不是RuntimeException类及其子类的异常实例则被称为Checked异常。

只有java语言提供了Checked异常，其他语言都没有提供，java认为Checked异常都是可以被处理（修复）的异常，所以 java 程序无须显式的处理 Checked 异常。如果程序没有处理 Checked 异常，该程序在编译时就会发生错误，无法通过编译。

Checked异常的处理方式：

* 当方法明确知道如何处理异常，程序应该使用try...catch块来捕获该异常，然后在对应的catch块中修补该异常。
* 当方法不知道如何处理异常，应该在定义该方法时声明抛出该异常。

Runtime 异常无须显式声明抛出，如果程序需要捕捉 Runtime 异常，也可以使用try...catch块来捕获 Runtime 异常。

问题是：大部分的方法总是不能明确知道如何处理异常，这就只能声明抛出异常了。
