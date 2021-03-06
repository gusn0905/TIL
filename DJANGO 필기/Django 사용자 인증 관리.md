# Django 사용자 인증 관리

2020.04.13

>  오늘 수업 
>
> - 회원가입, 로그인
>
> - 지금까지 배운 것  + 오늘 수업 => instagram 클론 코딩

### CRUD 기본 흐름 코딩

#### 1. 프로젝트 생성 (프로젝트 명:  `instagram`)

터미널에 `django-admin startproject instagram` 명령어 입력

- 프로젝트 초기 설정

  1. git

     `git init`으로 설정

     gitignore 생성

     - gitignore 사이트에서 django를 검색하여 파일들 복사하기

     - 터미널에 명령어 입력 (파일 복사한거 붙여넣는 명령어)

       ```bash
       ~/instagram/ $ vi.gitignore
       ```

  2. django

     setting.py에서 기본 설정해주기

     이후 로켓 확인하기

     (`git status` 하게 되면 .gitignore 들 제외됨!)

#### 2. 앱만들기 

터미널에 `python manage.py startapp articles` 입력 후 앱 등록하기

- Naming 설정시 
  - 앱 : 복수형 ★
  - 모델 : 단수형

#### 3. Model 작성하기

```python
# models.py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

1. `python manage.py makemigrations` 
2.  `python manage.py showmigrations` 을 통해 확인!! 
3.  `python manage.py migrate`을 통해 migrate 완료하기

#### 4. Admin

어떠한 모델에 대한 내용을 관리할 지이기에 model 파일 가져와야 한다.

```python
from django.contrib import admin
from .models import Article

# 작성 순서
# 1. python random, os, ..
# 2. django ~
# 3. 내가 만든 파일(model, url,...)

class ArticleAdmin(admin.ModelAdmin):
    list_display = ['id', 'title', 'created_at', 'updated_at'] # admin 사이트에 보여질 목록 설정
    list_display_link = ['title'] # link 설정 (설정안할 경우 처음값에 링크생김.)
    list_filter = ['created_at'] # 필터 기능(관리 편함)
    
admin.site.register(Article, ArticleAdmin)
```

`python manage.py createsuperuser` 입력 후 admin 사이트로 들어가 추가, 삭제, 수정 동작 확인하기

#### 5. forms.py

앱안에 forms.py 생성하기

```python
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'        
```

#### 6. 프로젝트/urls.py

```python
from django.urls import path, include

from articles import views

urlpatterns = [
    ..,
    path('', views.index),
    path('articles/', include('articles.urls')), # include 이용하여 추가해주기
]
```

#### 7. 앱/urls.py

```python
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
	path('', views.index, name='index'), 
    path('create/', views.create, name='create'),
    path('<int:pk>/', views.detail, name='detail'),
]
```

#### 8. views.py

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from .models import Article
from .forms import ArticleForm

def index(request):
    articles = Article.objects.order_by('-pk') # 최신순 정렬
    context = {
        'articles': articles, 
    }
    return render(request, 'articles/index.html', context)

def create(request):
    if request.method == 'POST': # 사용자 값을 받고 검증 후 저장 or Form으로
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('articles:detail', article.pk)
        messages.warning(reqeust, '폼을 다시 확인 후 제출해주세요.')
    else:    
    	form = ArticleForm()
    context = {
        'form': form
    }
    return render(request, 'articles/forms.html', context)

def detail(request, pk):
    article = Article.object.get(pk=pk)
    get_object_or_404(Article, pk=pk)
    context = {
        'article': article
    }
    return render(request, 'articles/detail.html', context)
    
```

- message 만들기

  `from django.contrib import messages` 로 불러와야 사용 가능

  base.html에 templates 사용하는 것을 추천한다!!

#### 9. templates

##### instagram/templates/base.html

- setting.py 에서 TEMPLATES 에 `DIRS: [os.path.join(BASE_DIRS, 'templates')]` 설정 추가하기
- bootstrap css, js 복사하여 붙이기

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Instagram - {% block title %}{% endblock %}</title>
    {% block css %}
    {% endblock %}
</head>
<body>
    {% if messages %}
    <ul class="messages">
        {% for message in messages %}
        	<li>{{ message }}</li>
        {% endfor %}
    </ul>
    {% block content %}
    {% endblock %}
</body>
</html>
```

##### articles/templates/index.html

```html
{% extends 'base.html' %}

{% block title %} 목록 {% endblock %}

{% block content%}
    {% for article in articles %}
        <p>{{ article.pk }}</p>
		<a href="{% url 'articles:detail' article.pk %}">글 보러가기</a>
		<hr>
    {% endfor %}
{% endblock %}
```

##### articles/templates/forms.html

```html
{% extends 'base.html' %}

{% block title %} 새 글쓰기 {% endblock %}

{% block content%}
    <form action="" method="POST">
        {% csfr_token %}
        {{ form.as_p }}
        <button>작성</button>
	</form>
{% endblock %}
```

##### articles/templates/detail.html

```html
{% extends 'base.html' %}

{% block title %}{{ article.pk }}번 글{% endblock %}

{% block content%}
<h1>{{ article.pk}}: {{ article.title }}</h1>
<p>{{ article.content }}</p>
{% endblock %}
```

### 회원가입 

> User objects 관련 공식문서
>
> https://docs.djangoproject.com/en/3.0/topics/auth/default/

- Django 내부의 `User 클래스`, `모델폼` 활용하여 구현!!

- 고려사항 

  -  user DB/password 암호화

  - pass 일치하는지 검증, 저장

    ※ 이 모든 고려사항을 `Django 내부적 기능 활용 `하여 해결!!

- 비밀번호 그래도 저장 X , `해시함수`이용하여 DB에 저장

  - 대표적인 해시 함수 : SHA256 (단방향 알고리즘, 역으로 연산 불가능, 블록체인/비트코인에서 많이 사용)

  - `salting` 을 통해 같은 비밀번호가 똑같은 해시함수의 결과가 나오지 않도록 해준다.

  - `PBKD2` : 해시 함수의 컨테이너, 솔트를 적용한 후 해시 함수의 반복 함수를 임의로 선택 가능

    cf) `MD5 ` : 복구화 가능

#### 1. (계정 관리) 앱 생성 

`python manage.py startapp accounts` (앱이름 : accounts)

setting.py에 앱 등록하기

#### 2. Models.py

```python
# models.py

from django.db import models
```

`python manage.py migrate` 만 해주자!

#### 3. 프로젝트/urls.py

```python
from django.urls import path, include

from articles import views

urlpatterns = [
    ..,
    path('', views.index),
    path('articles/', include('articles.urls')), 
    path('accounts/', include('accounts')),
]
```

#### 4. 앱/urls.py

```python
from django.urls import path

from . import views

app_name = 'accounts'
urlpatterns = [
	path('signup/', views.signup, name='signup'),
]
```

#### 5. views.py

- `from django.contrib.auth.forms import UserCreationForm` 를 통해 django 내부 기능을 이용!!

  - `class UserCreationForm(forms.ModelForm): ..` 

     **ModelForm 상속**받는다.

    `createuser` 를 이용하여 입력하면 password 암호화!

  - 참고)

    ```python
    class Meta:
        model = User
        fields = ("username",)
        field_classes = {'username': UsernameField}
        ...
    ```

```python
from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm

def signup(request): # articles와 흐름 비슷!
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            # 게시물 목록 페이지
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

#### 6. 앱/templates/signup.html

```html
{% extends 'base.html' %}

{% block content%}
    <form action="" method="POST">
        {% csfr_token %}
        {{ form.as_p }}
        <button>회원가입</button>
	</form>
{% endblock %}
```

- 아직 로그인 폼이 없기에 만들어진 user를 확인하려면 admin 사이트 들어가서 확인 가능하다.
- password 변경시 `set_password()` 함수를 통해 가능하다.

---

#### 상세 보기 페이지

##### 1.  urls.py

```python
from django.urls import path

from . import views

app_name = 'accounts'
urlpatterns = [
	path('signup/', views.signup, name='signup'),
    path('<int:pk>/', views.detail, name='detail'),
]
```

##### 2. views.py

- solution 1 : `User class` 이용

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)

def detail(request, pk):
    user = get_object_or_404(User, pk=pk) # User class 사용, import User 필요
    context = {
        'user': user,
    }
    return render(request, 'accounts/detail.html', context)
```

- solution 2:  `get_user_model` 이용 "추천"

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm # 회원관련 기능 auth (authentication, 인증)

def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = UserCreationForm() # 차이점!!
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)

def detail(request, pk):
    User = get_user_model()
    user = get_object_or_404(User, pk=pk) 
    context = {
        'user': user,
    }
    return render(request, 'accounts/detail.html', context)
```

##### 3. templates/accounts/detail.html

```html
{% extends 'base.html' %}

{% block content%}
<h1>{{ user.pk }} : {{ user.username }} </h1>
<hr>
{% endblock %}
```

---

### 참고용 - update

#### 1. urls.py

```python
from django.urls import path

from . import views

app_name = 'accounts'
urlpatterns = [
	path('signup/', views.signup, name='signup'),
    path('<int:pk>/', views.detail, name='detail'),
    path('<int:pk>/update/', views.update, name='update'),
]
```

#### 2. views.py

- `UserChangeForm ` 추가 : user 변경을 위한 form

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserCreationForm, UserChangeForm
from .forms import CustomUserChangeForm # form을 통해 customize, 작성안할 시에 django 내부 형식(복잡)에 따름

def signup(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)

def detail(request, pk):
    User = get_user_model()
    user = get_object_or_404(User, pk=pk) 
    context = {
        'user': user,
    }
    return render(request, 'accounts/detail.html', context)

def update(request, pk):
    form = UserChangeForm()
    context = {
        'form': form,
    }
    return render(request, 'account/update.html', context)
```

#### 3. templates/accounts/update.html

```html
{% extends 'base.html' %}

{% block content%}
    <form action="" method="POST">
        {% csfr_token %}
        {{ form.as_p }}
        <button>회원 정보 수정</button>
	</form>
{% endblock %}
```

#### 4. forms.py

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.forms import UserChangeForm

class CustomUserChangeForm(UserChangeForm): # 상속 이용!!
    class Meta:
        model = get_user_model()
        fields = ['username', 'first_name', 'last_name', 'email']   
```