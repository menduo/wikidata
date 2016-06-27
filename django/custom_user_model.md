---
title: 自定义 Django 用户模型 Custom Django User Model
categories: python, django
...

下文转自：http://wiki.opslinux.com/django.html

## 自定义用户系统
现在自己的项目中建立一个app，也就是一个目录，里面要有 __init__.py 文件

## models.py 

根据官方文档的要求创建并写入models.py文件：

```python
#! /usr/bin/env python
#coding=utf-8

from django.db import models
from django.contrib.auth.models import (
    BaseUserManager, AbstractBaseUser,PermissionsMixin
    )


class UserManager(BaseUserManager):
    def create_user(self, email, username, telephone, password=None):
        """
        Creates and saves a User with the given email, date of
        birth and password.
        """
        if not email:
            raise ValueError('Users must have an email address')

        user = self.model(
            email=self.normalize_email(email),
            username=username,
            telephone=telephone,
        )

        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email,username, password, telephone=""):
        """
        Creates and saves a superuser with the given email, username and password.
        """
        user = self.create_user(email=email,
                                username=username,
                                password=password,
                                telephone=telephone
        )
        user.is_admin = True
        user.save(using=self._db)
        return user


class User(AbstractBaseUser,PermissionsMixin):
    #注意：不继承PermissionsMixin类，是无法实现使用Django Group功能的，本人的项目需要使用所以继承该类。
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )

    username = models.CharField(
        max_length=100,
        unique=True,
        db_index=True
    )

    # avatar = models.URLField(
    #     blank=True
    # )

    telephone = models.CharField(
        max_length=50
    )

    created_at = models.DateTimeField(
        auto_now_add=True
    )

    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)

    objects = UserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    def get_full_name(self):
        # The user is identified by their email address
        return self.email

    def get_short_name(self):
        # The user is identified by their email address
        return self.username

    # On Python 3: def __str__(self):
    def __unicode__(self):
        return self.email

    def has_perm(self, perm, obj=None):
        "Does the user have a specific permission?"
        # Simplest possible answer: Yes, always
        return True

    def has_module_perms(self, app_label):
        "Does the user have permissions to view the app `app_label`?"
        # Simplest possible answer: Yes, always
        return True

    @property
    def is_staff(self):
        "Is the user a member of staff?"
        # Simplest possible answer: All admins are staff
        return self.is_admin
```


##  注册 admin

然后，我们要注册admin，再创建admin.py文件：

```python
from django import forms
from django.contrib import admin
from django.contrib.auth.models import Group
from django.contrib.auth.admin import UserAdmin
from django.contrib.auth.forms import ReadOnlyPasswordHashField

from app_store.account.models import User


class UserCreationForm(forms.ModelForm):
    """A form for creating new users. Includes all the required
    fields, plus a repeated password."""
    password1 = forms.CharField(label='Password', widget=forms.PasswordInput)
    password2 = forms.CharField(label='Password confirmation', widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ('email', 'username', 'telephone')

    def clean_password2(self):
        # Check that the two password entries match
        password1 = self.cleaned_data.get("password1")
        password2 = self.cleaned_data.get("password2")
        if password1 and password2 and password1 != password2:
            raise forms.ValidationError("Passwords don't match")
        return password2

    def save(self, commit=True):
        # Save the provided password in hashed format
        user = super(UserCreationForm, self).save(commit=False)
        user.set_password(self.cleaned_data["password1"])
        if commit:
            user.save()
        return user


class UserChangeForm(forms.ModelForm):
    """A form for updating users. Includes all the fields on
    the user, but replaces the password field with admin's
    password hash display field.
    """
    password = ReadOnlyPasswordHashField()

    class Meta:
        model = User
        fields = ('email', 'password', 'username','telephone', 'is_active', 'is_admin')

    def clean_password(self):
        # Regardless of what the user provides, return the initial value.
        # This is done here, rather than on the field, because the
        # field does not have access to the initial value
        return self.initial["password"]


class UserAdmin(UserAdmin):
    # The forms to add and change user instances
    form = UserChangeForm
    add_form = UserCreationForm

    # The fields to be used in displaying the User model.
    # These override the definitions on the base UserAdmin
    # that reference specific fields on auth.User.
    list_display = ('email', 'username', 'is_admin')
    list_filter = ('is_admin','groups')
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ('username','groups')}),
        ('Permissions', {'fields': ('is_admin',)}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'username', 'password1', 'password2')}
        ),
    )
    search_fields = ('email',)
    ordering = ('email',)
    filter_horizontal = ()

# Now register the new UserAdmin...
admin.site.register(User, UserAdmin)
# ... and, since we're not using Django's built-in permissions,
# unregister the Group model from admin.
# admin.site.unregister(Group)
# admin.site.register(Group)
```


关于admin.site.unregister(Group)，官方写的是不注册到admin，但是根据我自己的需求，我还是需要Group功能来做权限的，所以就注释掉了该行。

然后要将写好的 account 加入到settings文件的INSTALLED_APPS中，如下：

```pyhton
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'test', #自己的应用
    'test.account',  #建立的account应用
)
```

## 更新 settings 

然后设置在settings文件中加入：

`AUTH_USER_MODEL = 'account.User'`

## 同步数据库

接着同步数据库：

`python manager.py sync`


## view 中调用

在view中直接写注册方法，如下

```python
from django.shortcuts import render_to_response
from django.http import HttpResponse
from app_store.account.models import User

def register(request):
    if request.method == 'GET':
        return render_to_response("register.html")
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        email = request.POST['email']
        telephone = request.POST['telephone']
        User.objects.create_user(
            email=email, username=username, password=password, telephone=telephone)
        html = '''<html><head><body><a href="/">回到首页</a><h2>注册成功</h2></body></head></html>'''
        return HttpResponse(html)
```