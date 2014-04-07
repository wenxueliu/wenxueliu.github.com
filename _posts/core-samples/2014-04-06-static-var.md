---
layout: post
category : cpplang 
tagline: "静态成员"
tags : [static, c++/c]
---
{% include JB/setup %}

静态成员多次初始化，需要注意的问题:

    #include <iostream>
    #include <stdlib.h>
    #include <memory.h>
    using namespace std;

    class CDemo
    {
    public:
        CDemo(const char* str);
        ~CDemo();
        void showObjectName();   

    private:
        char name[20];
        static int static_var;
    };

    int CDemo::static_var = 1000;
    
    CDemo::CDemo(const char* str)
    {
        strncpy(name,str,20);
        cout << "Constructor called for " << name << "\n";
        cout << "Constructor called for " << CDemo::static_var << "\n";
    }

    CDemo::~CDemo()
    {
        cout << "Destructor called for " << name << "\n";
    };

    void CDemo::showObjectName()   //显示对象名
    {
        cout << "showObjectName() name: " << name << endl;
        cout << "showObjectName() static_var: " << static_var << endl;
    }


    void func()
    {
        cout << "enter func" << endl; 
        CDemo LocalObjectInFunc("LocalObjectInFunc"); 
        static CDemo staticObject("StaticObject");    
        staticObject.showObjectName();
        CDemo* pHeapObjectInFunc = new CDemo("HeapObjectInFunc"); 
        cout << "exit func" << endl; 
    } 


    CDemo GlobalObject("GlobalObject"); 
    
    int main()
    {
        cout << "enter main" << endl;
        static CDemo staticObjectInMain("staticObjectInMain");
        CDemo localObjectInMain("LocalObjectInMain");  
        CDemo* pHeapObjectInMain = new CDemo("HeapObjectInMain"); 
    
        cout << "In main, before calling func\n\n\n"; 
        func();
        cout << "\n\n";                             

        func();                 //staticObject静态对象已经存在，不再创建!                                    
        cout << "\n\nIn main, after calling func\n";  //14

        
        //staticObject.showObjectName();  //error C2065: “staticObject”: 未声明的标识符
        staticObjectInMain.showObjectName();
        GlobalObject.showObjectName();
        cout << "In main, after GlobalObject.showObjectName(): ";
        return 0;
    }

代码+执行结果已经很明了。

