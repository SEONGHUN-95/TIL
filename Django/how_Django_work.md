# How Django works?
> 장고 프레임워크의 요소들을 알아보자.
<br>

## URL - View 작동 방식

1. 브라우저에서 서버로 url 페이지 요청.
2. urls.py 파일에서 URL 매핑 확인하여, views.py 파일의 함수 호출.
3. 함수 작동 결과를 브라우저에 반영.

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
> 장고에서는 SQL문 대신 모델을 통해 데이터베이스를 처리한다.

<br>
Model을 수정하고 나서는 makemigrations 명령어를, 적용되지 않은 설정값들을 적용시키려면 migrate 명령어 사용.

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
