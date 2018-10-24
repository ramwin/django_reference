**Xiang Wang @ 2018-08-07 15:25:20**

### [Introduction to models 简介](https://docs.djangoproject.com/en/2.1/topics/db/models/)

### [Field Options 字段选项](https://docs.djangoproject.com/en/2.1/ref/models/fields/#field-options)
* null = True,    # 是否可以是NULL
* default = '0',  # 默认的数值
* blank=True      # admin界面是不是可以不填写。不填写的话就是NULL, 但是不影响model的创建
* related_name = 'table'  # 设置反向查找的功能
* verbose_name = '名字'   # admin界面用来给人看的名称,账号，而不是username
* help_text = ''  # 在每个model的form下面有一小行字符串。显示帮助信息。账号必须多于6个字符等等
* through = ''
* choices = TUPLE
* #### unique (False)
    1. 是否允许重复, 如果设置了True，并且一个model里面有2个True，get_or_create就必须把每个这样的字段设置好，不然就会报错
    2. [关于null和unique同时存在的问题](https://stackoverflow.com/questions/454436/unique-fields-that-allow-nulls-in-django), unique校验只对非null的进行唯一校验，包括空字符串，也不能重复
* primary_key = True # 是否为主键。最多一个，并且会自动加上null=False, unique=True
* unique
* null 和 blank
    * [null和blank的问题](https://stackoverflow.com/questions/8609192/differentiate-null-true-blank-true-in-django/50015717#50015717)
    * [关于如何在django里面插入null](https://code.djangoproject.com/ticket/4136)
    * 如果是其他field，空值会变成null。但是如果是charfield和textfield，因为form的缺陷，无法传递null，所以会导致永远不可能insertnull，只会insert空字符串。

### [Field Types 字段类型](https://docs.djangoproject.com/en/2.1/ref/models/fields/#field-types)
* AutoField, BigAutoField, BigIntegerField, BinaryField
* [BooleanField](https://docs.djangoproject.com/en/2.1/ref/models/fields/#booleanfield)  
> before 1.11 version: use NullBooleanField  
> after 2.0 version: user BooleanField(null=True)
* CharField, DateField
* #### DateTimeField
    * 参数
        * `auto_now_add = True`: 保存为当前时间，不再改变
        * `auto_now = True`: 保存未当前时间，每次保存时候会自动变更
    * 示例
        * 如果`timezone = 'UTC'`
            ```
            DateTimeModel.objects.create(time=timezone.now())  # 没问题
            DateTimeModel.objects.create(time=datetime.now())  # 自动保存为当前时间, 因为datetime.now()会自动变化，所以这个时间也是正确的
            DateTimeModel.objects.create(time='2017-12-12 10:24:00')  # 保存为UTC的10点了，如果是客户直接上传的，就会到处差了8小时
            DateTimeModel.objects.create(time='2017-12-12T10:24:00+08:00')  # 这么精确，也没问题
            ```
        * 如果`timezone = 'Asia/Shanghai'
            ```
            DateTimeModel.objects.create(time=timezone.now())  # 没问题
            DateTimeModel.objects.create(time=datetime.now())  # 自动保存为当前时间, 因为datetime.now()会自动变化，所以这个时间也是正确的
            DateTimeModel.objects.create(time='2017-12-12 10:24:00')  # 保存为Asia/Shanghai的10点了，如果是客户直接上传的，因为恰好客户和我们的服务器是同一个时区，所以也没问题
            DateTimeModel.objects.create(time='2017-12-12T10:24:00+08:00')  # 这么精确，也没问题
            ```
        * 结论: 服务器端都用timezone，客户端都用带iso 8601
* #### DecimalField
decimal:    '1.1', 1.1, decimal.Decimal('1.1')
```
decimal_places = 2  # 小数尾数
max_digits = 3  # 数字的位数(包括小数)
```
* DurationField, EmailField
* #### FileField:
`class FileField(upload_to="uploads/%Y/%m/%d")`
* FilePathField, FloatField, ImageField
* #### IntegerField
integer:    1, '1', 不可以是 '2.9', 但是可以是 2.9(之后存入2), 调用的是int函数
    * 基础
        ```
        models.IntegerField()   # 整数 -2147483648 - -2147483648
        models.PositiveSmallIntegerField()  # 0 - 32767
        models.SmallIntegerField()  #  -32767 - 32767
        models.PositiveIntegerField() # 0 - 2147483647

        models.FloatField() # 小数
            如果设置了unique的话，就会在数据库层面设置unique。不过数据库显示的时候偶尔会看上去一样
            实际上二者的二进制数据有点区别，换成是十进制后显示不出来。在python内部可以显示
        models.DecimalField()   # 精确小数
        ```
    * AutoField
        ```
        # 不使用原来的id，而是使用自定义的主键。注意一个model里面primary_key只能有一个，autofield也只能有一个
        models.AutoField(primary_key=True)
        ```
* GenericIPAddressField
* NullBooleanField
> Like BooleanField with null=True. Use that instead of this field as it’s likely to be deprecated in a future version of Django.
* PositiveIntegerField, PositiveSmallIntegerField,
* #### SlugField
    * 包含`[a-zA-Z_-]`，可以用在一些变量名上面
    * max_length 默认50
    * allow_unicode: 默认False，是否允许非ascii的名字
* SmallIntegerField, TextField, TimeField, URLField, 
* UUIDField
```
    import uuid
    models.UUIDField(default=uuid.uuid4)
```

### [Relationship fields 关联字段](https://docs.djangoproject.com/en/2.1/ref/models/fields/#module-django.db.models.fields.related)
    * [ ] ForeignKey
        * Example 例子  
            ```
            def get_default_user():
                return User.objects.first()
            
            limit_choices_to={'is_staff': True}, # 只能设置给 is_staff 的User
            related_name = "+" # 设置成+或者以+结尾，就会没有反向查找
            models.ForeignKey(Model,
                on_delete=models.CASCADE # 默认连带删除(2.0以后参数必须传)
                on_delete=models.SET(get_default_user)  # 删除后调用函数设置连带关系的默认直
            )
            ```
        * [on_delete参数参考](https://docs.djangoproject.com/en/1.10/ref/models/fields/#django.db.models.CASCADE)  
            * models.CASCADE: `连带删除`
            * models.PROTECT: `报错`
            * models.SET_NULL: `设置为空`
            * models.SET_DEFAULT: `设置为默认`
            * models.SET(): `调用函数`

    * [OneToOneField](https://docs.djangoproject.com/en/2.1/ref/models/fields/#onetoonefield)
    ```
    models.ForeignKey(Model)    # 关联到另一个Model
    models.OneToOneField(Model, related_name="profile", db_index=True)
    ```

### [Instance methods 实例方法](https://docs.djangoproject.com/en/2.1/ref/models/instances/)
#### Refreshing objects from database
```
obj = MyModel.objects.first()
del obj.field
obj.field  # loads the only field from database 会重载这个field, 不会重载其他的field
obj.refresh_from_db()  # reload all the fields
```

#### [save](https://docs.djangoproject.com/en/2.1/ref/models/instances/#django.db.models.Model.save)
save的时候，会把model的所有数据全量更新一遍，所以两个线程来了，只会save最后一个的数据
* 主键有就是update，主键没有就是insert
* save的时候发生了什么
    1. 触发model的pre—save信号
    2. 处理数据，每个field触发`pre_save`，比如`auto_now_add`和`auto_now`
    3. 处理给数据库的数据，每个field触发`get_db_prep_save`
    4. 插入数据
    5. 触发post-save信号
* django怎么区分update和insert
* 指定更新哪些field: `product.save(update_fields=["name"])`
    * 更新后，并不会触发`refresh_from_db`
* 如果是queryset的update操作，不会触发自定义的save方法。比如save的时候计算总分，如果update某个分数，总分并不会自动更新 `python3 manage.py test testapp.test_queries.TestMethodTestCase`

#### to be continued
* [ ] creating objects 创建数据
* [ ] validating objects 数据校验
* [ ] deleting objects 删除数据
* [ ] pickling objects 序列化数据
* [ ] other instance methods 其他方法
* [ ] extra instance methods 额外方法
* [ ] other attributes 其他属性
