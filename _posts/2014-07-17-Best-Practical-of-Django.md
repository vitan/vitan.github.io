---
layout: post
title: Best Practical of Django
tags: Django python
categories: Linux Django practical python
---

As a heavy user of Django, here I'd like to share some experiences based on my owner Django working practice.

Try to implement it at the right level, or avoid to cumulate codes in views.py/forms.py
---------------------------------------------------------------------------------------

Identify which level of model, view, or form implementing related features is rather important to achieve code reuse, avoid requirement missed, especially during team work, and even strength project long term maintenance.

For example, **Our requirement is expecting to reject one employee taking both *owner* and *assistant* at the same group.**

Assume the project model as following,

        class Employee(models.Model):
            """Record employees' information."""
            employee_number = models.IntegerField()
            name = models.CharField(max_length=255)

        class Group(models.Model):
            """Record groups' information."""
            name = models.CharField(max_length=255)
            description = models.CharField(max_length=255)
            owner = models.ForeignKey(
                "Employee", related_name="direct_owned_groups")
            assistant = models.ForeignKey(
                "Employee", related_name="direct_manage_groups")

###Bad Implementation###

Implement the restriction at *view*/*form* level, or in file views/group.py&forms/group.py, assume form description as following,

        class GroupEditForm(forms.ModelForm):

            class Meta:
                model = Group
                fields = ['name', 'description', 'owner', 'assistant']
                widgets = {
                    'name': forms.TextInput(),
                    'description': forms.TextInput(),
                    'owner': forms.Select(),
                    'assistant': forms.Select(),
                }

            def clean(self):
                cleaned_data = super(GroupEditForm, self).clean()
                owner = cleaned_data.get('owner')
                assistant = cleaned_data.get('assistant')
                if all([owner, assistant, owner is assistant]):
                    self._errors['assistant'] = self.error_class(
                        ["Can't set %s as the same group's owner&assistant" % owner.name])

                return cleaned_data

with the maybe view,

        class GroupEditView(UpdateView):
            """Edit group's info."""
            model = Group
            pk_url_kwarg = 'group_id'
            template_name = 'group/edit.html'
            form_class = GroupEditForm

###Good Implementation###

Implement the restriction at *model* level, or before the action of model object save, we can code a listener connected with signal pre_save. Assume we write it in listener.py as following,

        def decide_before_group_save(sender, instance, **kwargs):

            if instance.owner is instance.assistant:
                raise ValidationError("Can't set %s as the same group's owner&assistant" % owner.name)

at the same time, connect the above listener with pre_save(),

        pre_save.connect(decide_before_group_save, Sender=Group)

Here I ignored the interactive with front end.


Obviously, the later is closer with DB than the former. The advantage of the later implementation is:

* To prevent future model action acted against our requirement, especially the directly group object operation.
* To implement this feature reuse. We don't need consider it in the coming any group object operations yet.
* Strength project long term maintenance. Of course we can tell nothing to our candidate on this.
