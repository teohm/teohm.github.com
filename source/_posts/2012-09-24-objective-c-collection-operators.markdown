---
layout: post
title: "Objective-C Collection Operators"
date: 2012-09-24 13:52
comments: true
categories: 
- objective-c
---

Calculate average - a shorter way
----
When you have a collection of `Transactions` objects and want to
calculate its average amount, instead of looping through the collection
like this:
```objective-c
double sum = 0;
for (Transaction *transaction in transactions) {
    sum += [transaction.amount doubleValue];
}
NSNumber *avg = [NSNumber numberWithDouble:(sum / [transactions count])];
```

you can **reduce the loop to 1 line of code**, using 
[Objective-C key-value coding](http://developer.apple.com/library/mac/#documentation/cocoa/Conceptual/KeyValueCoding/Articles/CollectionOperators.html):
```objective-c
NSNumber *avg = [transactions valueForKeyPath:@"@avg.amount"];
```

Currently, there is a **fixed set** of collection operators:

- Simple collection operators (`@avg`, `@count`, `@sum`, `@max`, `@min`)
- Object operators (`@distinctUnionOfObjects`, `@unionOfObjects`)
- Array and set operators (`@distinctUnionOfArrays`, `@unionOfArrays`)

For details, refer to more
[usage](http://developer.apple.com/library/mac/#documentation/cocoa/Conceptual/KeyValueCoding/Articles/CollectionOperators.html) [examples](https://github.com/teohm/a-dip-in-objective-c/blob/master/ADipInObjectiveCTests/CollectionOperatorTests.m). 

