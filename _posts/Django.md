##一：DRF框架

DRF框架是建立在Django框架基础之上，二次开发的开源项目，是一个用于构建Web API 的强大而又灵活的工具。

特点：

1.提供了定义序列化器Serializer的方法，可以快速根据 Django ORM 或者其它库自动序列化/反序列化；

2.提供了丰富的类视图、Mixin扩展类，简化视图的编写；
3.丰富的定制层级：函数视图、类视图、视图集合到自动生成 API，满足各种需要；
4.多种身份认证和权限认证方式的支持；
5.内置了限流系统；
6.直观的 API web 界面；
7.可扩展性，插件丰富

##二：一般书写

1.创建序列化器用于序列化和反序列化

2.编写视图实现逻辑

3.定义路由

4.运行测试

##三：序列化器

序列化：将不可传输的对象转换成可存储或可传输的过程 instance -> native_data  -> json

即：响应过程

反序列化：将可传输的数据转换成对象 json -> native_data -> instance -> sql_data

即：请求过程

1.定义序列化器

```
a. serializer不只是能为数据库模型类定义，也可以为非数据库模型类的数据定义。
如：redis中的字段
b. 字段：UUIDField DecimalField
c. 选项参数：allow_blank trim_whitespace(是否截断空白)
d. 创建Serializer对象  Serializer(instance=None, data=tempty, **kwargs)
```

2.创建序列化器

 Serializer(instance=None, data=tempty, **kwargs)

```
a. 用于序列化时，将模型类对象传入instance参数，通过data属性获取序列化后的数据(json);当传入的是一个查询集时，需添加many=True参数补充
留坑：关联对象嵌套序列化
b. 用于反序列化时，将要被反序列化的数据传入data参数(data=data)
c. 在构造Serializer对象时，还可以通过context参数额外添加数据
serializer = AccountSerializer(account, context={'request': request})
```

通过context参数额外附加的数据，可以通过Serialzier对象的context属性获取

3.反序列化使用

验证：

用于反序列化时，将要被反序列化的数据传入data参数(data=data)

```
a. 在获取反序列化的数据前，必须调用is_valid()方法进行调用，返回True或者False
b. 验证失败可通过序列化器对象的errors属性获取错误信息，返回字典；验证成功，可以通过序列化对象的validated_data属性获取数据。
c. is_valid()方法还可以在验证失败时抛出异常serializers.ValidationError，可以通过传递raise_exception=True参数开启，REST framework接收到此异常，会向前端返回HTTP 400 Bad Request响应。
```

当需要补充定义验证行为时，可以使用一下三种方法：

```
a. 对单一字段校验  validate_<field_name>(self, value)   return value
b. 对多个字段进行验证  validate(self, attrs) attrs[''] 取字段  return attrs
c. 对单一字段补充验证行为 
举例：
def about_django(value):
	......
btitle = serializers.CharField(label='名称', max_length=20, validators=[about_django])
```

保存：验证完成后，想要基于validated_data完成数据对象的创建，可以通过实现create()和update()两个方法来实现。

```python
def create(self, validated_data):
        """新建"""
        return BookInfo.objects.create(**validated_data)

def update(self, instance, validated_data):
    """更新，instance为要更新的对象实例"""
        instance.btitle = validated_data.get('btitle', instance.btitle)
        instance.bpub_date = validated_data.get('bpub_date', instance.bpub_date)
        instance.bread = validated_data.get('bread', instance.bread)
        instance.bcomment = validated_data.get('bcomment', instance.bcomment)
        return instance
```

实现上述两个方法后再反序列化数据时就可以通过save()方法返回一个数据对象实例了。

当没有传递instance实例，则调用create()，当传递了instance实例，则调用update().

两点说明：

1）在对序列化器进行save()保存时，可以额外传递数据，这些数据可以在create()和update()中的validated_data参数获取到。

```python
serializer.save(owner=request.user)
```

2）默认序列化器必须传递所有required的字段，否则会抛出验证异常。但是我们可以使用partial参数来允许部分字段更新

```python
 Update `comment` with partial data
serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)
```

4.模型化序列化器ModelSerializer

ModelSerializer与常规的Serializer相同，但提供了：

- 基于模型类自动生成一系列字段
- 基于模型类自动为Serializer生成validators，比如unique_together
- 包含默认的create()和update()的实现

```
 class Meta:
        model = BookInfo
        fields = '__all__'  # 指明为模型类的哪些字段生成
        fields = ('id', 'btitle')  # 使用fields来明确字段
        exlude = ('image')  # 使用exclude可以明确排除掉哪些字段
        read_only_fields = ('id', 'bread', 'bcomment')  # 指明只读字段，即仅用于序列化输出的字段
        extra_kwargs = {
            'bread': {'min_value': 0, 'required': True}},
            'bcomment': {'max_value': 0, 'required': True}},
        }  # 添加或修改原有的选项参数
```



四：视图

1. request

   1)**Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。**即无论前端发送的哪种格式的数据，我们都可以以统一的方式读取数据。

   2）.data属性

   request.data返回解析之后的请求体数据（包含了对POST，PUT, 请求方式解析后的数据）

   3）.query_params，得到请求参数中的数据

2. Response(需要修改配置)

   1）.data    传给response对象的序列化后，但尚未render处理的数据

   2)  .status_code  状态码的数字

   3)  .content	经过render处理后的响应数据

3. 状态码

```
HTTP_400_BAD_REQUEST
HTTP_403_FORBIDDEN
HTTP_404_NOT_FOUND
HTTP_405_METHOD_NOT_ALLOWED

HTTP_500_INTERNAL_SERVER_ERROR
HTTP_502_BAD_GATEWAY
HTTP_507_INSUFFICIENT_STORAGE
```

4. 两个视图基类

   1）APIView  在进行调度（）分发前，会对请求进行身份认证，权限检查，流量控制。

   ```python
   apiView解决3件事
   # Ensure that the incoming request is perimitted
   self.perform_authentication(request)  #身份认证
   self.check_permissions(request)	 # 权限检查
   self.check_throttles(request)  # 流量控制
   ```

   在APIView中仍以常规的类视图定义方法实现get(), post()或者其他请求方法

   ```
   def get(self, request):
           books = BookInfo.objects.all()
           serializer = BookInfoSerializer(books, many=True)
           return Response(serializer.data)
   ```

   2) GenericAPIView  继承自`APIVIew`，增加了对于列表视图和详情视图可能用到的通用支持方法

   支持定义的属性：

   - **queryset** 列表视图的查询集

   - **serializer_class** 视图使用的序列化器
   - **pagination_class** 分页控制类

   提供的方法：

   **get_queryset(self)**  返回视图使用的查询集，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，可以重写。

   **get_serializer_class(self)**  返回序列化器类，默认返回`serializer_class`，可以重写。

   ##### get_serializer(self, *args, \**kwargs)  返回序列化器对象，被其他视图或扩展类使用，如果我们在视图中想要获取序列化器对象，可以直接调用此方法。

   **注意，在提供序列化器对象的时候，REST framework会向对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。**

   **get_object(self)** 返回详情视图所需的模型类数据对象。

5. 五个扩展类

   1）ListModelMixin  提供了list(self, request, *args, **kwargs)快速实现列表视图

   ```python
   class ListModelMixin(object):
       """
       List a queryset.
       """
       def list(self, request, *args, **kwargs):
           # 过滤
           queryset = self.filter_queryset(self.get_queryset())
           # 分页
           page = self.paginate_queryset(queryset)
           if page is not None:
               serializer = self.get_serializer(page, many=True)
               return self.get_paginated_response(serializer.data)
           # 序列化
           serializer = self.get_serializer(queryset, many=True)
           return Response(serializer.data)
       
   class BookListView(ListModelMixin, GenericAPIView):
       queryset = BookInfo.objects.all()
       serializer_class = BookInfoSerializer
   
       def get(self, request):
           return self.list(request)
   ```

   2) CreateModelMixin  提供create(self, request, *args, **kwargs)方法快速实现创建资源的视图，成功返回201状态码。

   ```python
   class CreateModelMixin(object):
       """
       Create a model instance.
       """
       def create(self, request, *args, **kwargs):
           # 获取序列化器
           serializer = self.get_serializer(data=request.data)
           # 验证
           serializer.is_valid(raise_exception=True)
           # 保存
           self.perform_create(serializer)
           headers = self.get_success_headers(serializer.data)
           return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)
   
       def perform_create(self, serializer):
           serializer.save()
   
       def get_success_headers(self, data):
           try:
               return {'Location':
                       str(data[api_settings.URL_FIELD_NAME])}
           except (TypeError, KeyError):
               return {}
   ```

   3） RetrieveModelMixin   详情视图扩展类，提供retrieve(self, request, *args, **kwargs)方法，可以快速实现返回一个存在的数据对象。

   ```python
   class RetrieveModelMixin(object):
       """
       Retrieve a model instance.
       """
       def retrieve(self, request, *args, **kwargs):
           # 获取对象，会检查对象的权限
           instance = self.get_object()
           # 序列化
           serializer = self.get_serializer(instance)
           return Response(serializer.data)
       
       
   class BookDetailView(RetrieveModelMixin, GenericAPIView):
       queryset = BookInfo.objects.all()
       serializer_class = BookInfoSerializer
   
       def get(self, request, pk):
           return self.retrieve(request)
   ```

   4)   UpdateModelMixin  更新视图扩展类，提供update(self, request, *args, **kwargs)方法，可以快速实现更新一个存在的数据对象。

   ```python
   class UpdateModelMixin(object):
       """
       Update a model instance.
       """
       def update(self, request, *args, **kwargs):
           partial = kwargs.pop('partial', False)
           instance = self.get_object()
           serializer = self.get_serializer(instance, data=request.data, partial=partial)
           serializer.is_valid(raise_exception=True)
           self.perform_update(serializer)
   
           if getattr(instance, '_prefetched_objects_cache', None):
               # If 'prefetch_related' has been applied to a queryset, we need to
               # forcibly invalidate the prefetch cache on the instance.
               instance._prefetched_objects_cache = {}
   
           return Response(serializer.data)
   
       def perform_update(self, serializer):
           serializer.save()
   
       def partial_update(self, request, *args, **kwargs):
           kwargs['partial'] = True
           return self.update(request, *args, **kwargs)
   ```

   5） DestroyModelMixin 删除视图扩展类，提供destroy(self, request, *args, **kwargs)方法，可以快速实现删除一个存在的数据对象。

6. 7个可用子类视图

   1）CreateAPIView  提供 post 方法  继承自： GenericAPIView、CreateModelMixin

   2）ListAPIView  提供 get 方法  继承自： GenericAPIView、ListModelMixin

   3）RetireveAPIView  提供 get 方法  继承自： GenericAPIView、RetireveModelMixin

   4）DestoryAPIView  提供 delete 方法  继承自： GenericAPIView、DestoryModelMixin

   5）UpdateAPIView  提供 put, patch 方法  继承自： GenericAPIView、UpdateModelMixin

   6）RetrieveUpdateAPIView  提供 get put patch 方法  继承自： GenericAPIView、RetrieveModelMixin  UpdateModelMixin

   7）RetrieveUpdateDestoryAPIView  提供 get put patch delete 方法  继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin

7. 视图集ViewSet














五：路由



六：运行测试

