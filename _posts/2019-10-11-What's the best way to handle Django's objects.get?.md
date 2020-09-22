---
title: What's the best way to handle Django's objects.get?
date: 2019-10-11 14:00
tags:
- python
- django
categories:
- django
---


# What's the best way to handle Django's objects.get?

## 1. Use the model DoesNotExist exception

```python
try:
    thepost = Content.objects.get(pk=number)
except Content.DoesNotExist:
    thepost = None
```



##  2. use the Django shortcut function [`get_object_or_404`](https://docs.djangoproject.com/en/stable/topics/http/shortcuts/#get-object-or-404) instead of the API directly

```python
from django.shortcuts import get_object_or_404

thepost = get_object_or_404(Content, pk=number)
```