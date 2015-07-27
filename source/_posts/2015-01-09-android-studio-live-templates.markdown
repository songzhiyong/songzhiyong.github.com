---
layout: post
title: "Android Studio Live Templates"
date: 2015-01-09 21:29
comments: true
categories: tricks
---

**Type the abbrs then click `TAB`, template codes will be generated automatically.**

###iterations
- `fori` Create iteration loop
```java
for(int $INDEX$ = 0; $INDEX$ < $LIMIT$; $INDEX$++) {
  $END$
}
```

- `itar` Iterate elements of array
```java
for(int $INDEX$ = 0; $INDEX$ < $ARRAY$.length; $INDEX$++) {
  $ELEMENT_TYPE$ $VAR$ = $ARRAY$[$INDEX$];
  $END$
}
```

- `itco` Iterate elements of java.util.Collection
```java
for($ITER_TYPE$ $ITER$ = $COLLECTION$.iterator(); $ITER$.hasNext(); ) {
  $ELEMENT_TYPE$ $VAR$ =$CAST$ $ITER$.next();
  $END$
}
```

- `iten` Iterate java.util.Enumeration
```java
while($ENUM$.hasMoreElements()){
  $TYPE$ $VAR$ = $CAST$ $ENUM$.nextElement();
  $END$
}
```

- `iter` Iterate Iterable | Array in J2SDK 5.0 syntax
```java
for ($ELEMENT_TYPE$ $VAR$ : $ITERABLE_TYPE$) {
  $END$
}
```

- `itit` Iterate java.util.Iterator
```java
while($ITER$.hasNext()){
  $TYPE$ $VAR$ = $CAST$ $ITER$.next();
  $END$
}
```

- `itli` Iterate elements of java.util.List
```java
for (int $INDEX$ = 0; $INDEX$ < $LIST$.size(); $INDEX$++) {
  $ELEMENT_TYPE$ $VAR$ = $CAST$ $LIST$.get($INDEX$);
  $END$
}
```

- `ittok` Iterate tokens from String
```java
for (java.util.StringTokenizer $TOKENIZER$ = new java.util.StringTokenizer($STRING$); $TOKENIZER$.hasMoreTokens(); ) {
    String $VAR$ = $TOKENIZER_COPY$.nextToken();
    $END$
}
```

- `itve` Iterate elements of java.util.Vector
```java
for(int $INDEX$ = 0; $INDEX$ < $VECTOR$.size(); $INDEX$++) {
  $ELEMENT_TYPE$ $VAR$ = $CAST$ $VECTOR$.elementAt($INDEX$);
  $END$
}
```

- `ritar` Iterate elements of array in reverse order
```java
for(int $INDEX$ = $ARRAY$.length - 1; $INDEX$ >= 0; $INDEX$--) {
  $ELEMENT_TYPE$ $VAR$ = $ARRAY$[$INDEX$];
  $END$
}
```
###other
- geti
- ifn
- inn
- inst
- lazy
- lst
- mn
- mx
- psvm
- toar
###output
- serr
- souf
- sout
- soutm
- soutp
- soutv
###plain
- psf
- psfi
- psfs
- St
- thr