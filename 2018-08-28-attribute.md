---
layout: post
title: __attribute__常见使用
tags: [c]
---

__attribute__ 可作用于函数、变量、类型、标签、枚举。语法形式为__attribute__(((attribute-list)), attribute-list用逗号分隔，多个__attribute__用空格分隔。
本文介绍常用于函数、变量的attribute:objc_runtime_name、objc_requires_super、objc_subclassing_restricted、cleanup、constructor and destructor、
enable_if、overloadable、objc_boxable、nonnull、const、always_inline等。废话少说，直接上代码，你可以通过终端运行下面demo程序。
编译命令：  clang -g  -ObjC -framework Foundation  attribute_demo1.c
clang版本：9.0.0 x86_64-apple-darwin17.7.0
---
<!--more-->

##attribute_demo1.c

```
#include <stdio.h>
#import <Foundation/Foundation.h>

#pragma mark - objc_runtime_name， 重新指定类名，可用于混淆代码
__attribute__((objc_runtime_name("ConfusionObject")))
@interface Person : NSObject


#pragma mark - objc_requires_super 标志子类复写这个方法时必须调用super的方法，否则给出编译警告
- (void)play __attribute__((objc_requires_super));

@end

@implementation Person

- (void)play {
    NSLog(@"Call %s", __func__);
}

@end

@interface Man : Person
@end

@implementation Man

- (void)play {
    [super play];
    NSLog(@"Call %s", __func__);
}

@end

#pragma mark - objc_subclassing_restricted, 定义一个final类
__attribute__((objc_subclassing_restricted))
@interface FinalObject : NSObject
@end

@implementation FinalObject

@end

//Compile error
// @interface ChildObject : FinalObject
// @end
// @implementation ChildObject
// @end



#pragma mark - cleanup，声明在一个变量上，变量作用域结束调用指定的注册函数
void intCleanup(int *num) {
    printf("Call %s, %d\n", __func__, *num);
}

void stringCleanup(char **ppstr) {
    printf("Call %s, %s\n", __func__, *ppstr);
}

void testCleanup() {
    //调用顺序后进先出
    int a __attribute__((cleanup(intCleanup))) = 10;
    char *ptr __attribute__((cleanup(stringCleanup))) = "string";
    printf("Call %s, a:%d, str:%s\n", __func__, a, ptr);
}

#pragma mark -- constructor and destructor，在main()函数之前或之后调用
//constructor,优先级值小的先调用
__attribute__((constructor(101)))
    void before1() {
        printf("Before main, func:%s\n", __func__);
    }

__attribute__((constructor(102)))
    void before2() {
        printf("Before main, func:%s\n", __func__);
    }

//destructor，优先级值大的先调用
__attribute__((destructor(201)))
    void after1() {
        printf("After main, func:%s\n", __func__);
    }

void after2() __attribute__((destructor(202)));
void after2() {
    printf("After main, func:%s\n", __func__);
}

#pragma mark - enable_if, 在编译阶段判断参数是否可用，不可用则报编译错误
void printAge(int age) __attribute__((enable_if(age >= 0 && age <= 120, "Invalid age!"))) {
    printf("Age:%d\n", age);
}

#pragma mark - overloadable，实现c函数重载
void print(const char *str) __attribute__((overloadable)) {
    printf("value:%s\n", str);
}

void print(int i) __attribute__((overloadable)) __attribute__((enable_if(i > 0, "i must gt 0"))) {
    printf("value:%d\n", i);
}


#pragma mark - objc_boxable，实现struct自定义数据类型装箱
typedef struct __attribute__((objc_boxable)) {
    CGFloat x, y, width, height;
} MyRect;

#pragma mark - nonnull, 被定义的函数中第1, 2, ... n个参数不能为null
void *my_memcpy(void *dest __attribute__((nonnull)), const void *src  __attribute__((nonnull)), size_t len)  {
    return memcpy(dest, src, len);
}

#pragma mark - const, 如果函数参数不变，直接返回值
 __attribute__((const)) int increaseOne(int a) {
    return a + 1;
}

#pragma mark - always_inline, 强制内联
 __attribute__((always_inline)) int square(int a) {
    return a * a;
}




int main(int argc, char **argv) {
    printf("func:%s\n", __func__);
    printAge(40);
    testCleanup();
    print("abc");
    print(5);
    NSLog(@"ClassName:%@", [Person class]);
    MyRect rect = {0, 0, 100, 200};
    NSValue *value = @(rect);
    Man *lisi = [Man new];
    [lisi play];
    char dst[32];
    my_memcpy(dst, "123", strlen("123"));
    int ret = increaseOne(3);
    ret = increaseOne(3);
    ret = square(5);

    return 0;
}
``` 
实践出真知，建议你亲自动手调试上面的代码。     

