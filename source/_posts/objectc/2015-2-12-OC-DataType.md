---
layout: post
title:  "OC学习之旅－数据类型"
date:   2015-02-12 18:28:37 +0800
author: Atopom
tags:
- DateType
categories:
- Object-C
published: true
---

> 数据类型，是每门开发语言的基本必修课，下面是为自己总结回顾使用的，分享给大家。如有错误请指正，交流学习，共同进步，谢谢～

# 基本数据类型

## int类型
八进制 以0开头的整型  
NSLog的格式符：  
％o 显示的八进制不带前导0  
％#o 显示的八进制带前导0  

十六进制 以0x开头的整型  
NSLog的格式符：  
％x 显示的十六进制不带前导0x  
％#x 显示的十六进制带签到0x  
％X、％#X 显示的十六进制是大写  

## float类型
NSLog的格式符：％f  
％e 科学技术法显示值  
％g 指数的值小于-4 大 于5，采用％e，否则采用％f  

十六进制的浮点常量包含前导 0x 或 0X，后面紧跟一个或多个十进制或十六进制数字，再后是 p 或 P，最后是可以带符号的二进制指数。  
例:0x0.3p10 表示的值为 3/16*2^10  
注:若无特殊说明，Object-c 将所有的浮点常量看做 double 值，要显示 double 值可使用和 float 一样的格式符。  

## char类型
NSLog的格式符:%c  
long double 常量写成尾部带有字母 l 或者 L 的浮点常量。1.234e+7L  

![](/images/OC/OC-Int-NSLog-Format-Character.png)

注:id类型可以通过类型转化符可以将一般的id类型的对象转换成特定的对象。  

```
_Bool 处理 Boolean(即 0 或 1)  
_Complex 处理复数  
_Imaginary 处理抽象数字  
```

实例变量的初始化值默认为 0  
实例变量作用域的指令:  
@private 实例变量可被定义在该类的方法直接访问，不能被子类定义的方法直接访问。  
@package 对于 64 位图像，可以在实现该类的图像的任何地方访问这个实例变量。  
@protected 实例变量可被该类及任何子类中定义的方法直接访问(默认的情况)。  
@public 实例变量可被该类中定义的方法直接访问，也可被其他类或模块中定义的方法访问。使得其他方法或函数可以通过(->)来访问实例变量(不推荐用)。  

在类中定义静态变量  
说明符voaltile和const正好相反，明确告诉编译器，指定类型变量的值会改变。(I/O端口)  
比如要将输出端口的地址存储在 outPort 的变量中。  

```
volatile char *outPort;  
*outPort = 'O';  
*outPort = 'N';  
```

这样就可以避免编译器将第一个赋值语句从程序中删除。  

## BOOL类型

```
typedef signed char BOOL;
#define YES (BOOL)1;
#define NO (BOOL)0;
```

# nil空类型
也就是C中的NULL，也就是0;  

# NSNumber类型
可以使用对象来封装基本数值。  
NSNumber 类来包装基本数据类型。  

```
+ (NSNumber *)numberWithChar :(char)value;
+ (NSNumber *)numberWithInt :(int)value;
+ (NSNumber *)numberWithFloat :(float)value;
+ (NSNumber *)numberWithBool :(BOOL)value;
```

还有包括无符号版本和各种 long 型数据及 long long 整型数据  
例如:  

```
NSNumber *number = [NSNumber numberWithInt :42];
```

将一个基本类型封装到 NSNumber 后,可以使用下列方法重新获得:  

```
- (char)charValue;
- (int)intValue;
- (float)floatValue;
- (BOOL)boolValue;
- (NSString *)stringValue;
```

# NSValue类型
NSNumber 实际上是 NSValue的子类, NSValue可以封装任意值。可以用NSValue将结构放入 NSArray 和 NSDictionary 中。  

创建新的 NSValue:  

```
+(NSValue*)valueWithBytes:(const void *) value objCType:(const char *)type;
```

@encode 编译器指令可以接受数据类型的名称并为你生成合适的字符串。  

```
NSRect rect = NSMakeRect(1,2,30,40);
NSValue *value;
value = [NSValuevalueWithBytes:&rect objCType:@encode(NSRect)];
```

使用getValue:来提取数值 (传递的是要存储这个数值的变量的地址)(先找地址再取值)  

```
value = [array objectAtIndex : 0];
[value getValue:&rect];
```

注:Cocoa 提供了将常用的 struct 型数据转化成 NSValue 的便捷方法:  

```
+ (NSValue*)valueWithPoint:(NSPoint)point;
+ (NSValue*)valueWithSize:(NSSize)size;
+ (NSValue*)valueWithRect:(NSRect)rect;
- (NSSize)sizeValue;
- (NSRect)rectValue;
- (NSPoint)pointValue;
```

# NSNull类型
在关键字下如果属性是NSNull表明没有这个属性，没有数值的话表明不知道是否有这个属性。  

```
[NSNull null] //总返回一样的值
+ (NSNull *)null;
```

例如:  

```
[contast setObject:[NSNull null] forKey:@"home"];
```

访问它:  

```
id home = [contast objectForKey:@"home"];
if (home==[NSNullnull]){
...
}
```

# NSFileManager类型
允许对文件系统进行操作(创建目录、删除文件、移动文件或获取文件信息)  
创建一个属于自己的 NSFileManager 对象  

```
NSFileManager *manager = [NSFileManager defaultManager] ;
```

将代字符‘~’替换成主目录  

```
NSString *home = [@"~" stringByExpandingTildeInPath];
//输出文件的扩展名
- (NSString*) pathExtension
```

示例:翻查主目录,查找.jpg 文件并输出找到的文件列表  

```
#import <Foundation/Foundation.h>
int main (int argc, const char* argv[]) {
	NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
	NSFileManager *manager;
	manager = [NSFileManager defaultManager];
	NSString *home;
	home = [@"~" stringByExpandingTildeInPath];
	NSDirectoryEnumerator *direnum;
	direnum = [manager enumeratorAtPath: home];
	NSMutableArray *files;
	files = [NSMutableArray arrayWithCapacity:100];
	NSString *filename;
	while (filename = [direnum nextObject]) {
		if ([[filename pathExtension] isEqualTo:@"jpg"]) {
			[files addObject: filename];
		}
	}
	NSEnumerator *fileenum;
	fileenum = [files objectEnumerator];
	while (filename = [fileenum nextObject]) {
		NSLog (@"%@", filename);
	}
	[pool drain];
	return 0;
}
```
