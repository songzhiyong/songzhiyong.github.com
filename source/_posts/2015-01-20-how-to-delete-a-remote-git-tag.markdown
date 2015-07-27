---
layout: post
title: "How to Delete a remote Git tag"
date: 2015-01-20 22:02
comments: true
categories: git
---
You probably won't need to do this often (if ever at all) but just in case, here is how to delete a tag from a remote Git repository.

If you have a tag named 'demo' then you would just do this:

```
git tag -d demo
git push origin :refs/tags/demo
That will remove 'demo' from the remote repository.
```

