---
layout: post
title: "SQLite Notes"
date: 2015-01-12 14:43
comments: true
categories: SQLite
---
The Offsets Function of fts

Integer Interpretation

0 The column number that the term instance occurs in (0 for the leftmost column of the FTS table, 1 for the next leftmost, etc.).
1 The term number of the matching term within the full-text query expression. Terms within a query expression are numbered starting from 0 in the order that they occur.
2 The byte offset of the matching term within the column.
3 The size of the matching term in bytes.
