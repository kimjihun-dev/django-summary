
# django 주요기능정리
<br><br>

## requirement.txt 작성방법
	pip freeze > requirements.txt

<br><br>
## form 파일 작성시 Tip
<br><br>

	class QuestionForm(forms.ModelForm):
		class Meta:
		model = Question
		fields = ['subject', 'content']
		# 클래스 지정으로 꾸밀수 있음
		widgets = {
		    'subject': forms.TextInput(attrs={'class': 'form-control'}),
		    'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
		}
		# 영어에서 한글로 변경
		labels = {
		    'subject': '제목',
		    'content': '내용',
		}


	class AnswerForm(forms.ModelForm):
		class Meta:
		model = Answer
		fields = ['content']
		labels = {
		    'content': '답변내용'
		}


<br><br>
## 이미지 업로드 방법
<br><br>

**model.py**

	image = models.ImageField(blank=True, null=True)


**pip install pillow**


**settings.py**

	MEDIA_URL = '/media/'
	MEDIA_ROOT = os.path.join(BASE_DIR, 'media')


**urls.py**

	from django.conf.urls.static import static
	from django.conf import settings
	urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
		

**template** 

	{% if project.image %}
		<img src="{{ project.image.url }}">
	{% endif %}


<br><br>
<hr/>
<br><br>

## static 정적파일 이용방법
<br><br>
**config/setting.py**

	STATIC_URL = '/static/'
	STATICFILES_DIRS = os.path.join(BASE_DIR, 'static')
	

사용할 경우

	base.html 파일에서
	{% load static %}
	<link rel="stylesheet" href="{% static 'style.css' %}">
	


<br><br>
<hr/>
<br><br>


## ForeignKey(on_delete=) 옵션들
<br><br>

종류 | 동작 | 
---|:---:|
`CASCADE` | 연결된 객체가 지워지면 해당 하위 객체도 같이 삭제
`PROTECT` | 하위 객체가 남아 있다면 연결된 객체가 지워지지 않음
`SET_NULL` | 연결된 객체만 삭제하고 필드 값을 null 로 설정
`SET_DEDAULT` | 연결된 객체만 삭제하고 필드 값을 설정된 기본 값으로 변경
`SET()` | 연결된 객체만 삭제하고 지정한 값으로 변경
`DO_NOTHING` | 아무일도 하지 않음
<br>


<br><br>
<hr/>
<br><br>

## 페이징 처리
<br><br>

제네릭 뷰를 쓸 경우(paginate_by 만 넣어주면 된다)
	
	class PostList(generic.ListView):
		queryset = Post.objects.filter(status=1).order_by('-created_at')
		template_name = 'blog/index.html'
		paginate_by = 4
	
	{% if is_paginated %}
	 <nav aria-label="Page navigation conatiner"></nav>
	  <ul class="pagination justify-content-center">
	    {% if page_obj.has_previous %}
	    <li><a href="?page={{ page_obj.previous_page_number }}" class="page-link">&laquo; PREV </a></li>
	    {% endif %}
	    {% if page_obj.has_next %}
	    <li><a href="?page={{ page_obj.next_page_number }}" class="page-link"> NEXT &raquo;</a></li>

	    {% endif %}
	  </ul>
	 </nav>
	{% endif %}
	

함수형으로 쓸 경우
	
	from django.core.paginator import Paginator, PageNotAnInteger, EmptyPage
	
	def PostList(request):
		oject_list = Post.objects.filter(status=1).order_by('-created_on')
    		paginator = Paginator(object_list, 3)  # 3 posts in each page
    		page = request.GET.get('page')
	
		try:
        		post_list = paginator.page(page)
    		except PageNotAnInteger:
            	# If page is not an integer deliver the first page
        		post_list = paginator.page(1)
    		except EmptyPage:
			# If page is out of range deliver last page of results
			post_list = paginator.page(paginator.num_pages)
    		return render(request, 'index.html', {'page': page, 'post_list': post_list})
	
	# view
	
	{% if post_list.has_other_pages %}
	  <nav aria-label="Page navigation conatiner"></nav>
	  <ul class="pagination justify-content-center">
	    {% if post_list.has_previous %}
	    <li><a href="?page={{ post_list.previous_page_number }}" class="page-link">&laquo; PREV </a></li>
	    {% endif %}
	    {% if post_list.has_next %}
	    <li><a href="?page={{ post_list.next_page_number }}" class="page-link"> NEXT &raquo;</a></li>
	   {% endif %}
	  </ul>
	  </nav>
	</div>
	{% endif %}
	

<br><br>
<hr/>
<br><br>

## summernote 설치
링크 : https://github.com/summernote/django-summernote/
<br><br>

**설치**

	pip install django-summernote

**settings.py 에서 등록**

	INSTALLED_APPS += ('django_summernote', )
	
**config urls.py 에 url 등록**

	path('summernote/', include('django_summernote.urls')),
	
**settings.py media 파일 설정 해줘야 함**

	MEDIA_URL = '/media/'
	MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
	
	from django.conf import settings
 	from django.conf.urls.static import static

	urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
	
**마이그레이트**

	 python manage.py migrate
	 
**admin 사이트에서 사용할시**

	from django_summernote.admin import SummernoteModelAdmin
	from .models import Post

	class PostAdmin(SummernoteModelAdmin):
    		summernote_fields = ('content',)

	admin.site.register(Post, PostAdmin)
	


<br><br>
<hr/>
<br><br>


## .gitignore 설정
<br><br>

	*.pyc
	*~
	__pycache__
	myvenv
	db.sqlite3
	/static
	.DS_Store
	/venv
	.idea	

<hr/>
<br><br>


## SECRET_KEY 설정
<br><br>

	KEY 값은 별도의 파일로 분리하여 import 로 가져온다.

	from . import secret_key
	
	SECRET_KEY = secret_key.secret_config


<br><br>
<hr/>
<br><br>


## Slug 설정
<br>

**슬러그란?**

슬러그는 페이지나 포스트를 설명하는 핵심단어의 집합. 원래 신문이나 잡지 등에서 제목을 쓸 떄,
중요한 의미를 포함하는 단어만을 이용해 제목을 작성하는 방법을 말함.
웹 개발분야에서는 콘텐츠의 고유 주소로 사용되어, 콘텐츠의 주소가 어떤 내용인지를 쉽게 이용할 수 있도록 한다.

보통 슬러그는 페이지나 포스트의 제목에서 조사, 전치나, 쉼표, 마침표 등을 뺴고 띄어쓰기는 하이픈(-) 으로 대체해서 만들며,
URL에 사용된다. 슬러그를 URL 에 사용함으로써 검색엔진에서 더 빨리 페이지를 찾아주고 검색 엔진의 정확도를 높여준다.

slug 필드의 디폴트 길이는 50 , 해당필드에는 인덱스가 디폴트로 생성됨.

	slug = model.SlugField('SLUG', unique=True, allow_unicode=True, help_text='')
	
slug 필드에 unique 옵션을 추가해 특정 포스트를 검색시 기본 키 대신에 사용.
allow_unicode 옵션을 추가하면 한글처리가 가능하다.
help_text 는 해당 칼럼을 설명해주는 문구로 폼 화면에 나타남.

admin.py

	prepopulated_fields = {'slug': ('title',)}


어드민 페이지 등록시 위의 옵션을 넣으면 title 필드를 이용해 값이 미리 채워지도록 한다.



<br><br>
<hr/>
<br><br>

## 템플릿에서 URL 추출 함수

get_absolute_url() 메소드 호출하는 방법과, {% url %} 템플릿 태그를 사용 하는 방법이 있음.

**get_absolute_url() 사용예시**

	def get_absolute_url(self):
        	return reverse('blog:post_detail', args=(self.slug,))
		
위의 메소드를 호출하면 /blog/post/slug단어  와 같은 형식이 된다. 

<br><br>
<hr />
<br><br>

## pythonanywhere 셋팅 


**wsgi.py**

	import os
	import sys
	path = "/home/kimzod/zodlab"
	if path not in sys.path:
	    sys.path.append(path)

	from django.contrib.staticfiles.handlers import StaticFilesHandler
	from django.core.wsgi import get_wsgi_application

	os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")
	application = StaticFilesHandler(get_wsgi_application())
	

**Static files**

사용했던 js, css 등 정적파일들과 이미지 등 파일 업로드시 설정.
앞에는 URL, 뒤에는 디렉토리이다.

	/media/		/home/kimzod/zodlab/media
	/static/	/home/kimzod/zodlab/static


**데이터베이스**

파이썬애니웨어 프리계정에서는 mysql 을 사용한다.

config/settings.py

	'default': {
		'ENGINE': 'django.db.backend.mysql',
		'NAME': 'DB이름',
		'USER': 'DB관리자계정',
		'PASSWORD': 'DB관리자 비밀번호',
		"HOST': 'Database host address 이 부분을 복사해서 입력한다.
	}





