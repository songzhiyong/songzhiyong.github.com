---
layout: post
title: "MySQL add UNIQUE INDEX"
date: 2014-09-02 21:43
comments: true
categories: basic
---

```
ALTER TABLE table ENGINE MyISAM;

ALTER IGNORE TABLE table ADD UNIQUE INDEX(uid,name);

ALTER TABLE table ENGINE InnoDB;
```