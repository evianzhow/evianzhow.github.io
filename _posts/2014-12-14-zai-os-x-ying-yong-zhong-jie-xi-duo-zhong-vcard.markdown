---
layout: post
title: 在 OS X 应用中解析多重 vCard
date: '2014-12-14 10:00:00'
permalink: "/zai-os-x-ying-yong-zhong-jie-xi-duo-zhong-vcard/"
tags:
- os-x
---

最近在开发一个 Cocoa Application 的过程中遇到了需要导入 vCard 数据的问题。我想把一个 vCard 文件读入并显示在一个 NSTableView 中。显示的过程不难，网上也有非常多介绍 NSTableView 的资料，但是介绍 vCard 导入的资料大多都是关于 iOS 开发的。

首先我们来看一下一个有效的 vCard 格式是怎么样的：

```
BEGIN:VCARD
VERSION:2.1
N;CHARSET=UTF-8;...
FN;CHARSET=UTF-8;...
TEL;CELL;...
...
END:VCARD
```

需要注意的是，以上只是单个联系人的 vCard，一个 vCard 文件支持包含多个联系人，也就是一个文件中含有多段以上的代码。

iOS 和 OS X 都有 AddressBook 这个 Framework，但是其中的方法有一些细微的区别。iOS 的 AddressBook 下的 ABPerson 类有一个比较方便的方法 `ABPersonCreatePeopleInSourceWithVCardRepresentation` 能将包含多个联系人的 vCard 顺利读取导入，返回结果是一个 CFArrayRef，可以 Bridge Casting 到 NSArray。但很奇怪的是，**OS X 的 Framework 里没有这个方法**，要解析 vCard 数据只能通过 `initWithVCardRepresentation:` 方法，而这个方法返回值是 **一个 ABPerson 对象**，不适用于一个 vCard 包含多个联系人的情况，如果传入多人的 vCard 时只会解析多个联系人中的第一个。

由于我们了解到 vCard 都是以 BEGIN:VCARD 和 END:VCARD 来分隔的，我们可以写一段代码来 Hack 一下。

下面给出我的代码：

```
NSString *string = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:nil];
            NSArray *array = [string componentsSeparatedByString:@"\n"];
            NSMutableArray *singleCard = [@[] mutableCopy];
            for (NSString *line in array) {
                if ([line hasPrefix:@"BEGIN:VCARD"])
                    [singleCard addObject:line];
                else if ([line hasPrefix:@"END:VCARD"]) {
                    [singleCard addObject:line];
                    vCardReader([[singleCard valueForKey:@"description"] componentsJoinedByString:@"\n"]);
                    [singleCard removeAllObjects];
                }
                else
                    [singleCard addObject:line];
            }
```

原理很简单，而函数 `vCardReader` 就是 `initWithVCardRepresentation:` 方法的一个简单 Wrapper。

```
void vCardReader(NSString *content)
{
    ABPersonRef person = (__bridge ABRecordRef)[[ABPerson alloc] initWithVCardRepresentation:[content dataUsingEncoding:NSUTF8StringEncoding]];
    NSLog(@"%@", ABRecordCopyValue(person, (__bridge CFStringRef)kABFirstNameProperty));
    NSLog(@"%@", ABRecordCopyValue(person, (__bridge CFStringRef)kABLastNameProperty));
    
    ABMultiValueRef multiPhones = ABRecordCopyValue(person, (__bridge CFStringRef)kABPhoneProperty);
    for (CFIndex i = 0; i < ABMultiValueCount(multiPhones); i++) {
        CFStringRef phoneNumberRef = ABMultiValueCopyValueAtIndex(multiPhones, i);
        NSString *phoneNumber = (__bridge NSString *)phoneNumberRef;
        CFRelease(phoneNumberRef);
        NSLog(@"%@", [phoneNumber stringByReplacingOccurrencesOfString:@"-" withString:@""]);
    }
    CFRelease(multiPhones);
}
```

这样就可以提取出所有联系人的姓名和电话了。由于底层涉及 C 的部分，所以 ARC 没法自动帮你插入 retain 和 release，需要使用 CFRelease 来防止内存泄漏。