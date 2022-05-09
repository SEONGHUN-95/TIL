# Sorting questions

> 게시글들을 최신순, 추천순, 인기순으로 정렬해보자.

```python
   #template file
            # 템플릿 파일에 콤보박스로 정렬 조건을 추가. 선택된 정렬 기준 확인하기 위해 class 설정
            <select class="form-control so">
                <option value="recent" {% if so == 'recent' %}selected{% endif %}>최신순
                </option>
                <option value="recommend" {% if so == 'recommend' %}selected{% endif %}>추천순</option>
                <option value="popular" {% if so == 'popular' %}selected{% endif %}>인기순</option>
            </select>

            # searchForm에 정렬 기준을 입력할 수 있도록 엘리먼트를 추가.
            <form id="searchForm" method="get" action="{% url 'index' %}">
                <input type="hidden" id="so" name="so" value="{{ so }}">
            </form>
        
        #JQuery 추가(select 값 변경 시 해당 값 so 필드에 저장 후 searchForm submit)
        $(document).ready(function(){
            $(".so").on('change', function(){
                $("#so").val($(this).val());
                $("#page").val(1);
                $("#searchForm").submit();
            });
        });
        </script>
```
```python
    #view file
    from django.db.models import Q, Count

    def index(request):
        #so
        so = request.GET.get('so','recent')
        
        #so 값이 변화될 때마다 정렬 기준 변동.
        #annotate는 필드 임시 추가 함수로서 Count 함수 등에서 사용 가능한 필드를 추가함.
        if so == "recommend":
        question_list = Question.objects.annotate(num_voter=Count('voter')).order_by('-num_voter', '-create_date')
        elif so == 'popular':
        question_list = Question.objects.annotate(num_answer=Count('answer')).order_by('-num_answer', '-create_date')
        else:
        question_list = Question.objects.order_by('-create_date')

        context = {'question_list': page_obj, 'page' :page, 'kw':kw, 'so':so}

        return render(request, 'cash/question_list.html', context)
```
