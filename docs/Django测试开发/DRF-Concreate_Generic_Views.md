## 拓展类

- RetrieveApIView

  提供：get方法

  继承：RetrieveModelMixin、GenericAPIView

- UpdateApIView

   提供：put、patch方法

   继承：UpdateModelMixin、GenericAPIView

- DestoryApIView

   提供：delete方法

   继承：DestoryModelMixin、GenericAPIView

- ListAPIView

   提供：get方法

   继承：ListModelMixin、GenericAPIView

- CreateAPIView

   提供：post方法

   继承：CreateModelMixin、GenericAPIView

- ListCreateApIView

   提供：post、get方法

   继承：ListModelMixin、CreateModelMixin、GenericAPIView

- RetrieveUpdateApIView

   提供：get、put、patch方法

   继承：ListModelMixin、CreateModelMixin、GenericAPIView

- ListCreateApIView

   提供：ge、createt方法

   继承：ListModelMixin、CreateModelMixin、GenericAPIView

- RetrieveUpdateApIView

   提供get、put、patch方法

   继承：RetrieveModelMixin、UpdateModelMixin、GenericAPIView

- RetrieveDestoryApIView

   提供get、delete方法

   继承：RetrieveModelMixin、DestoryModelMixin、GenericAPIView

- RetrieveUpdateDestoryApIView

   提供get、put、patch、delete方法

   继承：RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin、GenericAPIView

## 示例

views.py

```python
from rest_framework import generics

class ProjectsList(generics.ListCreateAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer

    ordering_fields = ['name', 'leader']
    filter_fields = ['name', 'leader', 'tester']



class ProjectsDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Projects.objects.all()
    serializer_class = ProjectModelSerializer
```



## 痛点

列表视图和详情视图无法合并

两个类视图中，有相同的get方法，会冲突

两个类视图所对应的url地址不一致