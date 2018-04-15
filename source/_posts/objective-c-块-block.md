---
title: 'objective-c[块]block'
date: 2018-04-15 15:56:44
category: objective-c
tags: [objective-c]
---

###  块的语法
```
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
```
* 注1: Block的声明与赋值只是保存了一段代码段,必须调用才能执行内部代码

如下：
1、 int (^myBlock)(int); // 声明一个名为oneParamBlock的块，该块接收一个int型参数，返回值也是int型的
2、 void (^myBlock)(int,int); // 接收两个int型参数，无返回值
3、 void (^myBlock2)(int parm1, int parm2); // 声明中可以含有参数名称
4、 int (^myBlock)(void); // 没有参数需要写上void，不能省略

### 块的定义

```objc
/*定义属性，block属性可以用strong修饰，也可以用copy修饰 */  
@property (nonatomic, strong) void(^myBlock)();                   //无参无返回值  
@property (nonatomic, strong) void(^myBlock)(NSString *);        //带参数无返回值  
@property (nonatomic, strong) NSString *(^myBlock)(NSString *);  //带参数与返回值 
 
 
/*block被当做方法的参数,格式：(block类型)参数名称  */
- (void)test:(void(^)())testBlock                    //无参无返回值  
- (void)test1:(void(^)(NSString *))testBlock         //带参数无返回值  
- (void)test2:(NSString *(^)(NSString *))testBlock   //带参数与返回值  
  
  
/*使用typedef定义block */
typedef void(^myBlock)();                            //以后就可以使用myBlock定义无参无返回值的block  
typedef void(^myBlock1)(NSString *);                 //使用myBlock1定义参数类型为NSString的block  
typedef NSString *(^myBlock2)(NSString *);           //使用myBlock2定义参数类型为NSString，返回值也为NSString的block  
//定义属性  
@property (nonatomic, strong) myBlock testBlock;  
//定义变量  
myBlock testBlock = nil;  
//当做参数  
- (void)test:(myBlock)testBlock;  
```

### block的赋值
```objc
没有参数没有返回值
myBlock testBlock = ^void(){
     NSLog(@"test");
 };

没有返回值，void可以省略
myBlock testBlock1 = ^(){
     NSLog(@"test1");
 }；

没有参数，小括号也可以省略
myBlock testBlock2 = ^{
     NSLog(@"test2");
 }；

有参数没有返回值
myBlock1 testBlock = ^void(NSString *str) {
      NSLog(str);
}

省略void
myBlock1 testBlock = ^(NSString *str) {
      NSLog(str);
}

有参数有返回值
myBlock2 testBlock = ^NSString *(NSString *str) {
     NSLog(str)
     return @"hi";
}

有返回值时也可以省略返回值类型
 myBlock2 testBlock2 = ^(NSString *str) {
     NSLog(str)
     return @"hi";
}

```

声明Block变量的同时进行赋值
```objc
int(^myBlock)(int) = ^(int num){
    return num * 7;
};

// 如果没有参数列表,在赋值时参数列表可以省略
void(^aVoidBlock)() = ^{
    NSLog(@"I am a aVoidBlock");
};
```
### Block作为OC函数参数
```objc
// 1.定义一个形参为Block的OC函数
- (void)useBlockForOC:(int(^)(int, int))aBlock
{
    NSLog(@"result = %d", aBlock(300,200));
}

// 2.声明并赋值定义一个Block变量
int(^addBlock)(int, int) = ^(int x, int y){
    return x+y;
};

// 3.以Block作为函数参数,把Block像对象一样传递
[self useBlockForOC:addBlock];

// 将第2点和第3点合并一起,以内联定义的Block作为函数参数
[self useBlockForOC:^(int x, int y){
    return x+y;
}];
```


