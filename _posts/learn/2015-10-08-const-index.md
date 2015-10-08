---
layout: post
title: ZZ指针常量与常量指针以及typedef定义的指针
date: 2015-10-08 13:24:11
category: "learn"
---

__一、指针常量与常量指针__

char string[4] = "abc";  
//常量指针  
const char* c1 = string;  
√ c1++;//常量指针可以改变其指向  
× (*c1)++;//常量指针指向的值为常量不可以修改，VS提示：表达式必须是可以修改的左值  
//指针常量  
char* const c2 = string;  
× c2++;//指针常量表示'c2'本身是一个常量，'c2'指向了一个值后再不能改变它的指向，VS提示：表达式必须是可以修改的左值  
√ (*c2)++;//指针常量可以改变其指向的值的内容  


__二、typedef定义的指针类型__

typedef char * pStr1;//char型的指针  
//在pStr前面加上const  
const pStr1 p1 = string;  
× p1++;//'const pStr1'其实是'char * const'(指针常量)，'p1'的指向不可以改变，VS提示：表达式必须是可以修改的左值  
√ (*p1)++;//'const pStr1'其实是'char * const'(指针常量)，'p1'指向的值的内容可以改变  
//在pStr后面加上const  
pStr1 const p2 = string;  
× p2++;//'pStr1 const'其实仍然是'char * const'(指针常量)，'p2'的指向不可以改变，VS提示：表达式必须是可以修改的左值  
√ (*p2)++;//'pStr1 const'其实仍然是'char * const'(指针常量)，'p2'指向的值的内容可以改变  


typedef const char * pStr2;//char型的常量指针  
//不加修饰  
pStr2  p3 = string;//常量指针  
√ p3++;//常量指针，可以改变其指向  
× (*p3)++;//常量指针指向的值为常量不可以修改，VS提示：表达式必须是可以修改的左值  
//在pStr2前面加上const  
const pStr2 p4 = string;  
× p4++;//'const pStr2'其实是'const char * const'(常量指针常量)，  
//'p4'的指向不可以改变，'p4'指向的值的内容也不可以改变，VS提示：表达式必须是可以修改的左值  
× (*p4)++;//'const pStr2'其实是'const char * const'(常量指针常量)，  
//'p4'的指向不可以改变，'p4'指向的值的内容也不可以改变，VS提示：表达式必须是可以修改的左值  
//在pStr2后面加上const  
pStr2 const p5 = string;  
× p5++;//'const pStr2'其实是'const char * const'(常量指针常量)，  
//'p5'的指向不可以改变，'p5'指向的值的内容也不可以改变，VS提示：表达式必须是可以修改的左值  
× (*p5)++;//'const pStr2'其实是'const char * const'(常量指针常量)，  
//'p5'的指向不可以改变，'p5'指向的值的内容也不可以改变，VS提示：表达式必须是可以修改的左值  


typedef char* const pStr3;//char型指针常量  
//不加修饰  
pStr3 p6 = string;  
× p6++;//指针常量，是一个常量，不可以改变它的指向  
√ (*p6)++;//指针常量，可以改变它指向的值。  
//在pStr3前面加上const  
const pStr3 p7 = string;  
× p7++;//‘const pStr3'其实是'char* const'(指针常量)  ，也就是说pStr3前面的const没有起作用，VS提示：表达式必须是可以修改的左值  
√ (*p7)++;//指针常量，可以改变它指向的值。  
//在pStr3后面加上const  
pStr3 const p8 = string;  
× p8++;//‘pStr3 const'其实是'char* const'(指针常量)  ，也就是说pStr3后面的const也不起作用，VS提示：表达式必须是可以修改的左值  
√ (*p8)++;//指针常量，可以改变它指向的值。  
