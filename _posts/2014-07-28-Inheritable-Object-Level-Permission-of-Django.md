---
layout: post
title: Implementation of Inheritable Object-level Permission of Django
tags: Django python
categories: Linux Django practical python
---

Struggled out of a Django-based project recently, I'd like to post such rather generic function **Implementation of Inheritable Object-level Permissions**, while I am always trying to improve it still. First of all, as I said, I didn't research what others have done on this before I started my implementation, well, in fact, I feel hard to clearly describe the requirement and google it.

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

Implementation
--------------

Here I am directly giving the source code with some comments on the detail implementation. And it is rather easy to describe that in the *Employee* model:
        
        import logging
        # I have implemented the [bulk_*](https://github.com/vitan/django-guardian) around django-guardian by myself as the latest upstream version didn't implement such bulk action yet. trying to contribute my work [pull request](https://github.com/lukaszb/django-guardian/pull/245) to upstream.
        from guardian.shortcuts import bulk_remove_perm, bulk_assign_perm

        from django.contrib.auth.models import AbstractUser
        from mptt.models import MPTTModel, TreeForeignKey
        
        class Employee(MPTTModel, AbstractUser):
            """Employee model recording the basic info of employee
            """
            manager = TreeForeignKey('self', null=True, blank=True, related_name='members')
            class Meta:
                # Declare the permission--change_employee_info
                permissions=(('change_employee_info', 'Can change employee info'),)
            
            def save(self, *args, **kwargs):
                old_manager_pk = None
                if getattr(self, 'pk', None) is not None:
                    try:
                        obj = Employee.objects.get(pk=self.pk)
                        if obj.manager:
                            #Record the old manager pk in memory for future perm remove.
                            old_manager_pk = obj.manager.pk
                        except Employee.DoesNotExist:
                            pass

                super(Employee, self).save(*args, **kwargs)

                #Record the new manager pk in memory for future perm assign.
                new_manager_pk = self.manager.pk if self.manager else None
                if new_manager_pk != old_manager_pk:
                    self._manager_change(old_manager_pk, new_manager_pk)
            
            def _manager_change(self, old_manager_pk, new_manager_pk):
                """Assign inherited change-employee-info permission to new manager,
                
                at the same time, remove such inherited permission from old manager.
                Note: plz keep the bulk_remove_perm() function called before bulk_assign_perm()
                """

                #Using MPTT api, quickly query out the current employee's descendants for his/her old manager perm removing.
                descendants = self.get_descendants(include_self=True)
                try:
                    ancestors = Employee.objects.get(pk=old_manager_pk).get_ancestors(include_self=True)
                    bulk_remove_perm('change_employee_info', ancestors, descendants)
                except Employee.DoesNotExist:
                    logger.debug("old managers doesn't exist yet!")

                try:
                    #Also using MPTT api, quickly query out the new manager's ancestors for his/her perm assigning.
                    ancestors = Employee.objects.get(pk=new_manager_pk).get_ancestors(include_self=True)
                    bulk_assign_perm('change_employee_info', ancestors, descendants)
                except Employee.DoesNotExist:
                    logger.debug("new managers doesn't exist yet")
