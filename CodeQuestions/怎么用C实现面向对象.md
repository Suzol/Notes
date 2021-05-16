```c
#include <stdio.h>
#ifndef C_Class
#define C_Class struct
#endif

C_Class A{
    C_Class A *A_this;
    void (*Foo)(C_Class A *A_this);
    int a;
    int b;
};

C_Class B{
    C_Class B *B_this;
    void (*Foo)(C_Class B *B_this);
    int a;
    int b;
    int c;
};
void A_Foo(C_Class A* A_this)
{
    printf("A.a = %d\n", A_this->a);
}

void B_Foo(C_Class B* B_this)
{
    printf("B.c = %d\n", B_this->c);
}

void A_Creat(struct A* p)
{
    p->Foo = A_Foo;
    p->a = 1;
    p->b = 2;
    p->A_this = p;
}

void B_Creat(struct B* p)
{
    p->Foo = B_Foo;
    p->a = 11;
    p->b = 12;
    p->c = 13;
    p->B_this = p;
}


int main()
{
    C_Class A *pa,a;
    C_Class B *pb,b;
    A_Creat(&a);    //初始化
    B_Creat(&b);

    pa = &a;    //指向
    pb = &b;
    pa = (C_Class A*)pb;    //这里是地址,pa解释了pb的部分内容
    pb->Foo(pa);

    a.Foo(&a);

    return 0;
}
```

这里显现了C没有Ｃ＋＋　ｐｒｉｖａｔｅ和ｐｒｏｔｅｃｔ属性的问题

怎么用Ｃ去模拟Ｃ＋＋？