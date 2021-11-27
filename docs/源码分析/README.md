[toc]





# 源码分析



## 版本：v4.0_创建增删改查五个接口(111-112)

### 代码

projects/urls.py

```python
from django.urls import path
from projects import views


urlpatterns = [
    path('', views.ProjectsList.as_view()),
    path('<int:pk>/', views.ProjectsDetail.as_view())
]

```

projects/views.py

```python
from django.http import HttpResponse, JsonResponse
from django.views import View
from projects.models import Projects
import json

class ProjectsList(View):

    def get(self, request):
        #1.从数据库中获取所有的项目信息
        project_qs = Projects.objects.all()

        #2.将数据模型实例转化为字典类型(嵌套字典的列表)
        project_list = []
        for project in project_qs:
            # one_dict = {
            #     'name': project.name,
            #     'leader': project.leader,
            #     'tester': project.tester,
            #     'publish_app': project.publish_app,
            #     'desc': project.desc
            # }
            # project_list.append(one_dict)

            project_list.append({
                'name': project.name,
                'leader': project.leader,
                'tester': project.tester,
                'publish_app': project.publish_app,
                'desc': project.desc
            })

        #JsonResponse第一个参数默认只能为dict字典，如果要设置为其他类型，需要将safe=False
        #序列化
        return JsonResponse(project_list, safe=False)


    def post(self, request):
        '''
        新增项目
        :param request:
        :return:
        '''
        #1.从前端获取json格式的数据，转化为Python中的类型
        #为了严谨性，这里需要做各种复杂的校验
        #比如：是否为json，传递的项目数据是否符合要求，有些必传参数是否携带等
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')

        #2.向数据库中新增项目
        # project = Projects.objects.create(name=python_dict['name'],
        #                         leader=python_dict['leader'],
        #                         tester=python_dict['tester'],
        #                         programer=python_dict['programer'],
        #                         publish_app=python_dict['publish_app'],
        #                         desc=python_dict['desc'])
        project = Projects.objects.create(**python_data)

        #3.将模型类对象转化为字典返回
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict, status=201)



class ProjectsDetail(View):

    def get(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #省略
        #2.获取指定pk值的项目
        project = Projects.objects.get(id=pk)

        #3.将模型类对象转化为字典
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict)


    #全部更新，patch为部分更新
    def put(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = Projects.objects.get(id=pk)

        #3.从前端获取json格式的数据
        #反序列化
        json_data = request.body.decode('utf-8')
        python_data = json.loads(json_data, encoding='utf-8')


        #4.更新项目
        project.name = python_data['name']
        project.leader = python_data['leader']
        project.tester = python_data['tester']
        project.programer = python_data['programer']
        project.publish_app = python_data['publish_app']
        project.desc = python_data['desc']
        project.save()

        #5.将模型类对象转化为字典
        #序列化
        one_dict = {
            'name': project.name,
            'leader': project.leader,
            'tester': project.tester,
            'publish_app': project.publish_app,
            'desc': project.desc
        }

        return JsonResponse(one_dict, status=201)



    def detele(self, request, pk):
        #1.校验前端传递的pk(项目id)值，类型是否正确(正整数)，在数据中是否存在
        #2.获取指定id为pk值的项目
        project = Projects.objects.get(id=pk)
        project.delete()

        return JsonResponse(None, safe=False, status=204)
```

### 源码分析

[django类视图as_view()方法解析](https://www.cnblogs.com/olivertian/p/11072528.html)

