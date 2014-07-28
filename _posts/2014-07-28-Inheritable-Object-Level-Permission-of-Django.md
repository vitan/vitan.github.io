---
layout: post
title: Implementation of Inheritable Object-level Permission of Django
tags: Django python
categories: Linux Django practical python
---

Struggled out of a Django-based project recently, I'd like to post such rather generic function **Implementation of Inheritable Object-level Permissions**, although I am always trying to improve it still. First of all, as I said, I didn't research what others have done on this before I started my implementation.

Requirement
------------

There is a django model *Employee* subclassed from *django.contrib.auth.models.AbstractUser* recording corp. employee info such as name, email, manager and so on. we want to implement:

* As a *direct* manager of some employees, I can edit my employees' info.
* As a *indirect* manager of some employees, I can also edit such info based on the *edit* perm inherited from their managers.
* The related *edit* perm will be automatically removed when one employee's *direct/indirect* manager changed.

Prerequisites
-------------

Consider the performance issue of recursively querying out employee's indirect manager, I subclassed [django-mptt](https://github.com/django-mptt/django-mptt) into model *Employee*, at the same time, used [django-guardian](https://github.com/lukaszb/django-guardian) for object-level permission management.

Assume the *Employee* model as following:

    from django.contrib.auth.models import AbstractUser
    from mptt.models import MPTTModel, TreeForeignKey
    class Employee(MPTTModel, AbstractUser):
        """Employee model recording the basic info of employee
        """
        manager = TreeForeignKey('self', null=True, blank=True, related_name='members')
