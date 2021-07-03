## 拓展类

+ RetrieveModelMixin

  提供 retrieve(request, *args, **kwargs) 方法，获取详情数据(一条数据)

  获取成功，返回 200 OK，否则返回 404 Not Found

- ListModelMixin

  提供 list(request, *args, **kwargs) 方法，获取列表数据(获取多条数据)

  获取成功，返回 200 OK，否则返回 404 Not Found

- CreateModelMixin

  提供 create(request, *args, **kwargs) 方法，创建数据

  获取成功，返回 201 OK；请求参数有误，返回 400 Bad Request

- UpdateModelMixin

  提供 update(request, *args, **kwargs) 方法，全更新

  提供 partial_update(request, *args, **kwargs) 方法，部分更新，支持PATCH方法

  更新成功(更新一条记录)，返回 200 OK

  请求参数有误，返回 400 Bad Request；不存在，返回 404 Not Found

- DestoryModelMixin

  提供 destory(request, *args, **kwargs) 方法，删除数据

  删除成功，返回 204 No Content，不存在，返回 404 Not Found

## 示例

```python
from rest_framework import mixins

#1.首先继承mixins，然后再继承GenericAPIView
class ProjectsList(mixins.ListModelMixin,
                   mixins.CreateModelMixin,
                   GenericAPIView):


    queryset = Projects.objects.all()

    serializer_class = ProjectModelSerializer

    ordering_fields = ['name', 'leader']

    filter_fields = ['name', 'leader', 'tester']

    #7.在某个视图中指定分页类
    pageination_class = PageNumberPaginationManual


    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)


    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


class ProjectsDetail(mixins.RetrieveModelMixin,
                     mixins.UpdateModelMixin,
                     mixins.DestroyModelMixin,
                     GenericAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer



    def get(self, request, *args, **kwargs):
        self.retrieve(request, *args, **kwargs)


    def put(self, request, *args, **kwargs):
        self.update(request, *args, **kwargs)


    def detele(self, request, *args, **kwargs):
        self.destroy(request, *args, **kwargs)
```

