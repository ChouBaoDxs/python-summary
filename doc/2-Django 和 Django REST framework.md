[TOC]

# Django REST framework
## ViewSet 增强，以 Mixin 的形式支持多 Serializer 等功能
默认情况下，APIView 和 ViewSet 都只支持单个 serializer_class，如果要针对不同的接口采用不同的 serializer_class，需要自己重写 `get_serializer_class()` 方法：
```python
class XxxViewSet(viewsets.ModelViewSet):
    def get_serializer_class(self):
        if self.action == 'retrieve':
            return RetrieveSerializer
        elif self.action == 'create':
            return CreateSerializer
        else:
            return self.serializer_class
```
写一个 Mixin：
```python
class SerializerMixin:
    def get_serializer_class(self):
        """
        让 ViewSet 支持以下写法，而不是serializer_class（这段代码来自 jumpserver 源码
        serializer_classes = {
            'default': serializers.AssetUserWriteSerializer,
            'list': serializers.AssetUserReadSerializer,
            'retrieve': serializers.AssetUserReadSerializer,
        }
        """
        serializer_class = None
        if hasattr(self, 'serializer_classes') and isinstance(self.serializer_classes, dict):
            serializer_class = self.serializer_classes.get(self.action, self.serializer_classes.get('default'))
        if serializer_class:
            return serializer_class
        return super().get_serializer_class()

    def get_request_serializer(self, *args, **kwargs):
        """
        校验请求数据并返回请求serializer
        """
        if 'data' not in kwargs:
            kwargs['data'] = self.request.data
        serializer = self.get_serializer(*args, **kwargs)
        serializer.is_valid(raise_exception=True)
        return serializer
```
日常使用：
```python
class UserViewSet(SerializerMixin, viewsets.GenericViewSet):
    serializer_classes = {
        'default': UserDefaultSer,
        'me': UserMeSer,
        'create_or_update_profile': UserProfileCreateOrUpdateReqSer
    }

    @action(detail=False, methods=['POST'])
    def create_or_update_profile(self, request):
        req_serializer: UserProfileCreateOrUpdateReqSer = self.get_request_serializer()
        user_profile = req_serializer.save()
        res_serializer = UserProfileDisplaySer(user_profile)
        return Response(res_serializer.data)
```

# Django
## 逻辑删除 Model
```python
class LogicDeleteQuerySet(models.QuerySet):
    def delete(self):  # queryset 的逻辑删除方法
        # return self.update(is_deleted=True, deleted_at=timezone.now())
        return self.update(deleted_at=timezone.now())


class LogicDeleteManager(models.manager.BaseManager.from_queryset(LogicDeleteQuerySet)):
    pass


class LogicDeleteModel(models.Model):
    # is_delete = models.BooleanField('删除标记', default=False, editable=False)
    deleted_at = models.DateTimeField('删除时间', null=True, db_index=True)

    class Meta:
        abstract = True

    def delete(self, using=None, keep_parents=False):  # 单个实例的逻辑删除方法
        # self.is_delete = True
        self.deleted_at = timezone.now()
        # self.save(update_fields=['is_delete', 'deleted_at'])
        self.save(update_fields=['deleted_at'])

    class NotDeleteManager(LogicDeleteManager):
        def get_queryset(self):
            # return super().get_queryset().filter(is_delete=False)
            return super().get_queryset().filter(deleted_at__isnull=True)

    objects = NotDeleteManager()
    all_objects = models.Manager()  # 需要查找被逻辑删除的数据时使用这个 all_objects
```

## 读取数据库表生成 Model 定义
一般用于已有项目（数据库表不是由 django migrate 生成的）上进行 django 开发
- 命令：`python manage.py inspectdb`
- 可以选择输出到文件：`python manage.py inspectdb > models.py`

## 收集静态文件
```python
# settings.py
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```
执行命令：`python manage.py collectstatic`

由于 DEBUG 模式下，django 会通过 STATICFILES_DIRS 帮忙处理静态文件，而 STATICFILES_DIRS 和 STATIC_ROOT 是冲突配置，所以我习惯下面这样，在需要收集静态文件时将 DEBUG 设置为 False： 
```python
# settings.py
if DEBUG:
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]
else:
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```
