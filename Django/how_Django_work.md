# How Django works?

## MVT(Model,View,Template)
![MVT](https://github.com/SEONGHUN-95/TIL/blob/main/MVT.jpeg)


> 장고 프레임워크의 요소들을 알아보자.

## URL - View 작동 방식

1. 브라우저에서 서버로 url 페이지 요청.
2. urls.py 파일에서 URL 매핑 확인하여, views.py 파일에서 해당 함수 호출.
3. view는 model 통해 필요한 DB 불러오고, Template 통해 HTML 파일 생성.
4. 함수 작동 결과를 브라우저에 반영.
 

```python
#urls.py

from . import views
from django.urls import path
from .views import base_views, question_views, answer_views, comment_views, vote_views

app_name = 'cash'

urlpatterns = [
    #base_views.py
    path('', base_views.index, name = 'index'),
    path('<int:question_id>/', base_views.detail, name = 'detail'), 
]
```
path(' ', base_views.index, name = 'index')
-> ' '일 경우 base_views.index 호출, 결과 브라우저에 반영.

## View
> View에서 model과 template에 접근한다.
```python
#base_views.py

def detail(request, question_id):
    """
    질문 내용
    """
    question = get_object_or_404(Question, pk=question_id)

    #입력 파라미터
    page = request.GET.get('page','1')
    
    #조회하기
    question_list = Question.objects.order_by('-create_date')    
    #페이징처리
    paginator = Paginator(question_list, 10)
    page_obj = paginator.get_page(page)

    context = {'question': question, 'question_list' : page_obj}
    return render(request, 'cash/question_detail.html', context)

```

## Model 
> 장고에서는 SQL문 대신 모델을 통해 데이터베이스를 처리한다.(ORM)
<br>

```python
from django.db import models
from django.contrib.auth.models import User

class Question(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name = 'author_question')
    subject = models.CharField(max_length=200)
    content = models.TextField()
    modify_date = models.DateTimeField(null=True, blank=True)
    create_date = models.DateTimeField()
    voter = models.ManyToManyField(User, related_name = 'voter_question')

    def __str__(self):
        return self.subject
```
Model을 수정하고 나서는 makemigrations 명령어를, 적용되지 않은 설정값들을 적용시키려면 migrate 명령어 사용.

<br>

##  Template
> template은 클라이언트에게 반환할 틀이라고 할 수 있다.

```python
{% extends 'base.html' %}
{% block content %}
<div class="container my-3">
    {% if messages %}
    <div class="alert alert-danger my-3" role="alert">
    {% for message in messages%}
        <strong>{{ message.tags }}</strong>
        <ul><li>{{ message.message}}</li></ul>
    {% endfor %}
    </div>
    {% endif %}
    <!-- 질문 -->
    <h2 class="border-bottom py-2">{{ question.subject }}</h2>
    <div class="card my-3">
    <div class="card">
        <div class="card-body">
            <div class="card-text" style="white-space: pre-line;">{{ question.content }}</div>
            <div class="d-flex justify-content-end">
                {% if question.modify_date %}
                <div class="badge badge-light p-2 text-left mx-3">
                <div class="mb-2">modified at</div>
                <div> {{ question.modify_date }}</div>
                </div>
                {% endif %}                
                <div class="badge badge-light p-2 text-left">
                    <div class="mb-2"> {{ question.author.username}}</div>
                    <div>{{ question.create_date }}</div>
                </div>
            </div>
</script>
{% endblock %}
```
