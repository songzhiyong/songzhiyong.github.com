---
layout: post
title: "Android R.plurals.name"
date: 2014-08-23 11:06
comments: true
categories: 
---
###Quantity Strings (Plurals)

getQuantityString(int id, int quantity)
getQuantityString(int id, int quantity, Object... formatArgs)

```
    <plurals name="search_results">
        <item quantity="one">%1$d result for \"%2$s\"</item>
        <item quantity="other">%1$d results for \"%2$s\"</item>
    </plurals>
```

```
//第一个参数count 用于选择xml中选择那个item
//第二个参数用于插入占位符
// 
String resultHint = getResources().getQuantityString(R.plurals.search_results, 					count,new Object[] { count, query });

```