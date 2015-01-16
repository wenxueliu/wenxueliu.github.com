

在这次的项目开发中遇到动态的删除ArrayList中的一些元素，假如我们有如下的一个List:

List<Integer> list = new LinkedList<Integer>();
list.add(4);
list.add(2);
list.add(1);
list.add(1);
list.add(2);

    一种错误的方式：

for(int i = 0, len = list.size(); i < len; i++){  
    if(list.get(i) == 1) {  
       list.remove(i);  
    }  
}

上面这种方式会抛出如下异常：

Exception in thread "main" java.lang.IndexOutOfBoundsException: Index: 3, Size: 3  
    at java.util.ArrayList.RangeCheck(Unknown Source)  
    at java.util.ArrayList.get(Unknown Source)

因为你删除了元素，但是未改变迭代的下标，这样当迭代到最后一个的时候就会抛异常。

    正确的做法是：

for(int i = 0, len = list.size(); i < len; i++){  
    if(list.get(i) == 1){  
       list.remove(i);  
       len--;
       i--;
    }  
}

    更好的一个做法

List接口内部实现了Iterator接口，提供开发者一个iterator()得到当前list对象的一个iterator对象。所以我们还有一个更好的做法是：

Iterator<Integer> iterator = list.iterator();  
while(iterator.hasNext()){  
    int i = iterator.next();  
    if(i == 1){  
        iterator.remove();  
    }  
}


