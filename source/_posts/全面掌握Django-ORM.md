---
title: 全面掌握Django ORM
tags:
  - django
  - orm
comments: true

date: 2021-07-17 21:18:10
categories:
index_img:
banner_img:
typora-root-url: 全面掌握Django-ORM
---

# `ORM`介绍

![](5e64d5dc0001020119201080.jpg)

# 字段类型和参数

## 常用的字段

```python
# 自增长字段
Auto = models.AutoField()
BigAuto = models.BigAutoField()

# 二进制数据
Binary = models.BinaryField()

# 布尔型
Boolean = models.BooleanField()
NullBoolean = models.NullBooleanField()

# 整型
PositiveSmallInteger = models.PositiveSmallIntegerField(db_column="age")  # 5个字节
SmallInteger = models.SmallIntegerField(primary_key=False)  # 6个字节
PositiveInteger = models.PositiveIntegerField()  # 10个字节
Integer = models.IntegerField(verbose_name="11个字节大小")  # 11个字节
BigInteger = models.BigIntegerField(unique=True)  # 20个字节

# 字符串类型
Char = models.CharField(max_length=100, null=True, blank=True, db_index=True)  # varchar
Text = models.TextField(help_text="这个是longtext")  # longtext

# 时间日期类型
Date = models.DateField(unique_for_date=True, auto_now=True)
DateTime = models.DateTimeField(editable=False, unique_for_month=True, auto_now_add=True)
Duration = models.DurationField()  # int, Python timedelta实现

# 浮点型
Float = models.FloatField()
Decimal = models.DecimalField(max_digits=4, decimal_places=2)  # 11.22, 16.34

# 其它字段
Email = models.EmailField()  # 邮箱
Image = models.ImageField()
File = models.FileField()
FilePath = models.FilePathField()
URL = models.URLField()
UUID = models.UUIDField()
GenericIPAddress = models.GenericIPAddressField()
```

## 关系型字段

- 一对一（`OneToOneField`）
  - `models.OneToOneField(Test)`
- 多对一（`ForeignKey`）
  - `foreign = models.ForeignKey(Other)`
- 多对多（`ManyToManyField`），默认或自定义中间表
  - `models.ManyToManyField(B)`

## 字段参数

1. 所有字段都有的属性值

   - `editable`: 是否可以编辑, 默认为`False`

   - `help_text`: 在表单中显示帮助信息的参数
   - `db_index`: 为当前字段建立索引, 默认为`False`
   - `null/blank`: 字段是否可以为空, `null`约束数据库层面, `blank`约束前端表单提交时是否为空
   - `unique`: 唯一性约束, 默认为`False`
   - `verbose_name`: 设置字段别名（或备注）
   - `primary_key`: 设置当前字段是否为主键, 默认为`False`
   - `db_column`: 设置当前字段的名称，在数据库中表的名称

2. 属于个别字段的参数
   - `max_length[CharField]`：最大长度
   - `unique_for_date[DateField]`: 字段日期必须唯一
   - `unique_for_month[DateField]`：月份唯一
   - `auto_now[DateField]`: 修改记录时是否自动更新当前日期
   - `auto_now_add[DateField]`: 添加记录时是否自动设置当前日志
   - `max_digits[DecimalField]`: 总共有多少位
   - `decimal_places[DecimalField]`: 小数点后数字的个数

3. 关系型字段的参数
   - `related_name`: 外键关联中的反向查询，由父表查询子表的信息
   - `on_delete`: 当一个被外键关联的对象被删除时，`Django`将模仿`on_delete`参数定义的`SQL`约束执行相应操作
     - `CASCADE`: 模拟`SQL`语言中的`ON DELETE CASCADE`约束，将定义有外键的模型对象同时删除（该操作为当前`Django`版本的默认操作）
     - `PROTECT`: 阻止上面的删除操作, 弹出`ProtectedError`异常
     - `SET_NULL`: 将外键字段设为`null`, 只有当字段设置了`null=True`时, 方可使用该值
     - `SET_DEFAULT`: 将外键字段设为默认值, 只有当字段设置了`default`参数时，方可使用
     - `DO_NOTHING`: 什么也不做
     - `SET()`: 设置为一个传递给`SET()`的值或者一个回调函数的返回值, 注意大小写

## 自关联

在同一个模型中一个字段关联另一个字段，模型内部的字段自关联。

```python
# 自关联的两种写法
pid = models.ForeignKey('self', null=True, blank=True)
pid = models.ForeignKey('${TableName}', null=True, blank=True)
```

# 元数据Meta

`Meta`数据

- `db_table`: 指定数据表的名称
- `ordering`: 指定按照哪些字段来排序, 类型为一元元组
- `verbose_name`: 为模型类设置一个直观可读的名称
- `abstract:` `=True`将该类设置为基类, 不生成数据表, 仅供其它类继承
- `permissions`: 为数据表设置额外的权限, 通过二元元组来实现`(('定义好的权限', '权限的说明'), )`
- `managed`: 表示是否按照`django`既定的规则来管理数据表, 默认是`True`
- `unique_together`: 指定联合唯一键，可以使用一元元组或二元元组(即多个唯一约束)
- `app_label`: 定义模型类属于哪一个应用
- `db_tablespace`: 定义数据表空间的名字

官网文档中定义的`Meta`元数据中一共有25个。

## 模型开发示例

![](5d7dfd7a00016d1119201080.jpg)

```python
class Teacher(models.Model):
    """讲师信息表"""
    nickname = models.CharField(max_length=30, primary_key=True, db_index=True, verbose_name="昵称")
    introduction = models.TextField(default="这位同学很懒，什么都没说", verbose_name="简介")
    fans = models.PositiveIntegerField(default='0', verbose_name="粉丝数")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    updated_at = models.DateTimeField(auto_now_add=True, verbose_name="更新时间")

    class Meta:
        verbose_name = "讲师信息表"
        verbose_name_plural = verbose_name


class Course(models.Model):
    """课程信息表"""
    title = models.CharField(max_length=100, primary_key=True, db_index=True, verbose_name="课程名")
    teacher = models.ForeignKey(Teacher, on_delete=models.CASCADE, verbose_name='课程讲师')
    type = models.CharField(choices=((1, '实战课'), (2, '免费课'), (0, '其他')), max_length=1, default=0, verbose_name='课程类型')
    price = models.PositiveSmallIntegerField(verbose_name='价格')
    volume = models.BigIntegerField(verbose_name='销量')
    online = models.DateField(verbose_name='上线时间')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now_add=True, verbose_name='更新时间')

    class Meta:
        verbose_name = '课程信息表'
        verbose_name_plural = verbose_name

    def __str__(self):
        return f'{self.get_type_display()}-{self.title}'


class Student(models.Model):
    """学生信息表"""
    nickname = models.CharField(max_length=30, primary_key=True, db_index=True, verbose_name="昵称")
    course = models.ManyToManyField(Course, verbose_name='课程')
    age = models.PositiveIntegerField(verbose_name='年龄')
    gender = models.CharField(choices=((1, '男'), (2, '女'), (0, '保密')), max_length=1, default=0, verbose_name='性别')
    study_time = models.PositiveIntegerField(verbose_name='学习时长')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now_add=True, verbose_name='更新时间')

    class Meta:
        verbose_name = '学生信息表'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname


class TeacherAssistant(models.Model):
    """助教信息表"""
    nickname = models.CharField(max_length=30, primary_key=True, verbose_name="昵称")
    teacher = models.OneToOneField(Teacher, on_delete=models.SET_NULL, verbose_name='讲师', null=True)
    hobby = models.CharField(max_length=100, null=True, verbose_name='爱好')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now_add=True, verbose_name='更新时间')

    class Meta:
        verbose_name = '助教信息表'
        db_table = 'courses_assistant'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.nickname
```

![](image-20210718092929934.png)

# `Django`数据表操作

## 更改数据表

```python
# Django 更改数据表的方法:
 python manage.py makemigrations #生成迁移文件
 python manage.py migrate #生成(更新)数据库表
```

删除某个模型类的完整操作:

- 在已创建的`app`下, 首先删除`models.py`中需要删除的模型类
- 删除该模型类在迁移脚本`migrations`中的对应文件
- 删除该项目在`django_migrations`中的对应记录删除数据库中对应的数据表
- 最后删除数据库中的表

## 如何导入数据

### `django-shell`

`python manage.py shell`调取`shell`。

```python
(venv) D:\PycharmProjects\onlineshop\shop>python manage.py shell
Python 3.9.5 (tags/v3.9.5:0a7dcbd, May  3 2021, 17:27:52) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from course.models import Teacher 
>>> t = Teacher(nickname='holy')
>>> t.save()

```

相应的值保存到数据库中。

![](image-20210718103714740.png)

### 脚本导入

```python
import os
import sys
import django
import random

project_path = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(project_path)
print(project_path)

os.environ['DJANGO_SETTINGS_MODULE'] = 'shop.settings'
django.setup()

from course.models import Teacher, Course, Student, TeacherAssistant


def import_data():
    """使用django ORM导入数据"""

    # 课程数据 bulk_create()

    Course.objects.bulk_create([Course(title=f'Python系列教程{i}', teacher=Teacher.objects.get(nickname="Jack"),
                                       type=random.choice(0, 1, 2),
                                       price=random.randint(200, 300), volume=random.randint()) for i
                                in range(1, 5)])

    Course.objects.bulk_create([Course(title=f'Java系列教程{i}', teacher=Teacher.objects.get(nickname="Allen"),
                                       type=random.choice(0, 1, 2),
                                       price=random.randint(200, 300), volume=random.randint()) for i
                                in range(1, 5)])
    Course.objects.bulk_create([Course(title=f'Golang系列教程{i}', teacher=Teacher.objects.get(nickname="Henry"),
                                       type=random.choice(0, 1, 2),
                                       price=random.randint(200, 300), volume=random.randint()) for i
                                in range(1, 5)])

    # 学生数据 update_or_create()
    Student.objects.update_or_create(nickname="A同学", defaults={"age": random.randint(18, 58),
                                                               "gender": random.choice(0, 1, 2),
                                                               "study_time": random.randint(9, 999)})
    Student.objects.update_or_create(nickname="B同学", defaults={"age": random.randint(18, 58),
                                                               "gender": random.choice(0, 1, 2),
                                                               "study_time": random.randint(9, 999)})
    Student.objects.update_or_create(nickname="C同学", defaults={"age": random.randint(18, 58),
                                                               "gender": random.choice(0, 1, 2),
                                                               "study_time": random.randint(9, 999)})
    Student.objects.update_or_create(nickname="D同学", defaults={"age": random.randint(18, 58),
                                                               "gender": random.choice(0, 1, 2),
                                                               "study_time": random.randint(9, 999)})
    # 正向添加
    # 销量大于等于1000的课程
    Student.objects.get(nickname="A同学").course.add(*Course.objects.filter(volume__gte=1000))
    # 销量大于5000的课程
    Student.objects.get(nickname="B同学").course.add(*Course.objects.filter(volume__gt=5000))
    # 反向添加
    Course.objects.get(title="Python系列教程1").student_set.add(*Student.objects.filter(study_time__gte=500))
    Course.objects.get(title="Python系列教程2").student_set.add(*Student.objects.filter(study_time__lte=500))

    # 助教数据 get_or_create()
    TeacherAssistant.objects.get_or_create(nickname='助教1',
                                           defaults={"hobby": "学习英语", "teacher": Teacher.objects.get(nickname='Jack')})
    TeacherAssistant.objects.get_or_create(nickname='助教2',
                                           defaults={"hobby": "学习德语", "teacher": Teacher.objects.get(nickname='Allen')})
    TeacherAssistant.objects.get_or_create(nickname='助教2',
                                           defaults={"hobby": "学习汉语", "teacher": Teacher.objects.get(nickname='Henry')})
    return True


if __name__ == '__main__':
    try:

        import_data()
        print("数据导入成功！")
    except Exception as e:
        print("数据导入失败", e)
```

## # `Models API`

## 查询集`QuerySet`

```python
from django.shortcuts import render
from django.views.generic import View
from .models import *


def course_view(request):
    # 1.查询、检索、过滤
    teachers = Teacher.objects.all()
    print(teachers)  # 返回查询集
    # <QuerySet [<Teacher: Teacher object (holy)>, <Teacher: Teacher object (Jack)>, <Teacher: Teacher object (Allen)>]>

    teacher1 = Teacher.objects.get(nickname='Jack')  # 返回一条结果
    print(teacher1, type(teacher1))
    # Teacher object (Jack) <class 'course.models.Teacher'>

    teacher2 = Teacher.objects.filter(fans__gte=500)  # QuerySet 可以是多条结果
    for t in teacher2:
        print(f'讲师姓名{t.nickname}--粉丝数{t.fans}')
        # 讲师姓名Jack--粉丝数666
        # 讲师姓名Henry--粉丝数899

    # 2. 字段字符匹配，大小写敏感
    teacher3 = Teacher.objects.filter(fans__in=[666, 1200])
    print(teacher3)
    # <QuerySet [<Teacher: Teacher object (Jack)>]>

    teacher4 = Teacher.objects.filter(nickname__icontains='A')
    print(teacher4)
    # <QuerySet [<Teacher: Teacher object (Jack)>, <Teacher: Teacher object (Allen)>]>

    # 3.结果切片、排序、链式查询
    print(Teacher.objects.all()[:1])

    teacher5 = Teacher.objects.all().order_by('-fans')
    for teacher in teacher5:
        print(teacher.nickname, teacher.fans)
        # Henry 899
        # Jack 666
        # Allen 123
        # holy 0
    print(Teacher.objects.filter(fans__gte=500).order_by('nickname'))
    # <QuerySet [<Teacher: Teacher object (Henry)>, <Teacher: Teacher object (Jack)>]>

    # 4. 查看执行的原生SQL     str(xx.query)
    print(str(Teacher.objects.all().query))
    # SELECT "course_teacher"."nickname", "course_teacher"."introduction",
    #    "course_teacher"."fans", "course_teacher"."created_at", "course_teacher"."updated_at" FROM "course_teacher"

    return render(request, 'course.html')
```




[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


