---
layout: post
title:  "python-django-admin-issues"
date:   2017-02-28 20:10:53 +0900
categories: python-django
---

어드민 개발하면서 배운 내용 정리  



<br><br><br>








# How to show model list to Admin  
http://djangobook.com/customizing-change-lists-forms/  
{% highlight ruby %}
class SAdmin(admin.ModelAdmin):
    list_display = ('s_id', 'r_id', 'question', 'answer', 'staff', 'updated')
    list_display_links = ('question',)
    ordering = ('-updated',)
    search_fields = ('question',)

    def has_add_permission(self, request):
        return False


admin.site.register(SData, SAdmin)
{% endhighlight %}

<br><br><br>








# How to show join results to detail view(edit view)  
http://brownbears.tistory.com/101  
{% highlight ruby %}
# models.py
class RData(models.Model):
    r_id = models.CharField(primary_key=True, max_length=32)
    ...


class Comment(models.Model):
    comment_id = models.CharField(primary_key=True, max_length=32, editable=False)
    r_id = models.ForeignKey(RawData, db_column='r_id')
    ...


# admin.py
class CommentInline(admin.TabularInline):
    model = Comment
    fk_name = 'r_id'
    extra = 0
    ...


class RAdmin(admin.ModelAdmin):
    ...
    inlines = [CommentInline, SDataInline, ]
    ...

{% endhighlight %}


<br><br><br>










# 장고 디버깅 방법(아래가 영문)   
http://raccoonyy.github.io/django-debug-toolbar/  
http://django-debug-toolbar.readthedocs.io/en/stable/installation.html  
{% highlight ruby %}
# settings.py
INSTALLED_APPS = [
    ...
    'django.contrib.staticfiles',
    ...
    'debug_toolbar', # debug_toolbar
]
MIDDLEWARE = [
    ...
    'debug_toolbar.middleware.DebugToolbarMiddleware', # debug_toolbar
]
INTERNAL_IPS = ('127.0.0.1', )

# urls.py
from django.conf import settings
from django.conf.urls import include
if settings.DEBUG:
    import debug_toolbar
    urlpatterns += [
        url(r'^__debug__/', include(debug_toolbar.urls)),
    ]
{% endhighlight %}

<br><br><br>







# MultiValueDictKeyError  
`comment_uid = models.CharField(primary_key=True, max_length=32, editable=False)`  


<br><br><br>









# Group by and Count between two models to ModelAdmin  
{% highlight ruby %}
    def get_queryset(self, request):
        qs = super(RAdmin, self).get_queryset(request)
        return qs.annotate(comment_count=Count('comment'), s_count=Count('sdata'))

    def get_comment_count(self, obj):
        return obj.comment_count
    get_comment_count.short_description = 'comment count'
{% endhighlight %}


<br><br><br>







# Extend chage_view  
http://rahmonov.me/posts/customize-django-admin-templates/  


<br><br><br>







# django admin, Make a link to a list field
https://eureka.ykyuen.info/2014/12/12/django-sort-the-djano-admin-list-table-by-specific-field/  








# Make a query in select column  
{% highlight ruby %}
def get_queryset(self, request):
    qs = super(CustomAdmin, self).get_queryset(request).extra(
        select={
            'comment_count': 'SELECT COUNT(table2.id) '
                             'FROM table2 '
                             'WHERE table2.f_id = table1.id'
        },
    )
{% endhighlight %}


<br><br><br>







# 데이터베이스 연결 시간 제한?  
https://docs.djangoproject.com/en/1.10/ref/databases/#mysql-db-api-drivers  
https://docs.djangoproject.com/en/1.8/ref/settings/#std:setting-CONN_MAX_AGE  
MySQLdb는 python3 지원 안된다고 함, 대신 mysqlclient가 MySQLdb based이고 python3를 지원하여 recommended 이다.  
slow query가 없는 듯 하여 원인 파악이 어려운 상태지만 아래 코드는 connection timeout를 건 것.
{% highlight ruby %}
'CONN_MAX_AGE': 60,
{% endhighlight %}


<br><br><br>








# date field, 편집은 안되고 자동 현재 시간을 하고 싶다.  
{% highlight ruby %}
auto_now=True
{% endhighlight %}

<br><br><br>








# [Django Admin Excel exporting]  
[Django Admin Excel exporting]: https://django-import-export.readthedocs.io/en/latest/installation.html  
{% highlight ruby %}
pip install django-import-export

# settings.py
INSTALLED_APPS = (
    ...
    'import_export',
)

# admin.py
class MyResource(resources.ModelResource):
    class Meta:
        model = MyData
        import_id_fields = ('my_uid',)
        exclude = ('tags',)

class MyAdmin(ImportExportModelAdmin):
    ...
    resource_class = SeedDataResource
{% endhighlight %}

<br>

**2017-04-10**  
Mac에서 잘 됐으나 개발 CentOS 서버에서는 import error가 발생했다.  
여러가지를 시도 해 보았으나 아래의 코드가 적절하게 동작했다.  
가상환경에서 python3와 python2에 대해서 왔다갔다 동작하는 듯?  
Mac, CentOS 모두 Python3.5.2이고, package도 똑같이 설치했는데.....   
parse는 /Library/Frameworks/Python.framework/Versions/3.5/lib/python3.5/urllib/parse.py 경로에 위치한다.  

{% highlight ruby %}
# diff_match_patch.py
try:
    from urllib.parse import urlparse
except ImportError:
     from urlparse import urlparse
{% endhighlight %}




<br><br><br>






# Tags + autocomplte  
https://godjango.com/33-tagging-with-django-taggit/  
http://django-taggit.readthedocs.io/en/latest/forms.html  
http://django-autocomplete-light.readthedocs.io/en/3.1.3/tutorial.html#using-autocompletes-in-the-admin  
{% highlight ruby %}
pip install django-taggit
pip install django-autocomplete-light

# settings.py
INSTALLED_APPS = [
  'dal',
  'dal_select2',
  'taggit',
  'django.contrib.admin',
  ...
]


# models.my
from taggit.managers import TaggableManager

class AModel:
  ...
  tags = TaggableManager(blank=True)


# admin.py
from adminapp.forms import TagModelForm

class AInline(TabularInline):
  fieldsets = (
    (None, {'fieldls': ('title', ('tags'), ... )})
    )
    ...
    form = TagModelForm

class AAdmin(ModelAmdin):
  ...
  form = TagModelForm
  ...

# view.py
from dal import autocomplete
from taggit.models import Tag

class TagAutocomplete(autocomplete.Select2QuerySetView):

    def get_queryset(self):
        # Don't forget to filter out results depending on the visitor !
        if not self.request.user.is_authenticated():
            return Tag.objects.none()

        qs = Tag.objects.all()

        if self.q:
            qs = qs.filter(name__istartswith=self.q)

        return qs  


# forms.py
from django import forms
from dal import autocomplete
from adminapp.models import AModel

class TagModelForm(forms.ModelForm):
    class Meta:
        model = AModel
        fields = ('__all__')
        widgets = {
            'tags': autocomplete.TaggitSelect2(url='tag-autocomplete', attrs={'size': 80, 'class': 'form-control focused'})
        }


# urls.py
from adminapp.views import TagAutocomplete
...

url(r'^tag-autocomplete/$', TagAutocomplete.as_view(), name='tag-autocomplete',)

{% endhighlight %}
<br>
**Slug?**  
<br>
Slugs are used in permalinks structure.  
It is a simple id of posts and pages or tag.  

<br><br><br>
# [admin menu bar]   
[admin menu bar]: https://github.com/cdrx/django-admin-menu
{% highlight ruby %}
pip install django-admin-menu

INSTALLED_APPS = [
    'admin_menu',

ADMIN_LOGO = 'logo.png' # option

# option
MENU_WEIGHT = {
    'World': 20,
    'Auth': 4,
    'Sample': 5
}

class MyAdmin(admin.ModelAdmin):
    menu_title = "Users"
    menu_group = "Staff"

# changelists.css

{% endhighlight %}

<br>
**20170405**  
* django-admin-menu package적용하려함
* MAC환경에서 잘 동작하지만 Linux 환경에서 문제발생
* libsass, sass import 문제
* gcc compiler version/centos 일 수 있음
* 결국 [suit package]를 적용
    * pip uninstall django-admin-menu
    * pip install django-suit==0.2.25
    * settings.py > ‘suit’ 추가

[suit package]: https://django-suit.readthedocs.io/en/develop/sortables.html

<br><br>
**20170406**   
* [Django Jet]
* [sass]
* [luby-sass-install]
{% highlight ruby %}
# suit의 경우 dom 구조가 기존에서 변경이 되어 다른 interface로 변경을 했다.  
# [Django Jet]가 적합해 보였다.
# 참고 : ./manage.py collectstatic 은 DEBUG=False 일 때 사용한다. static폴더를 설정한 경로로 static file이 모인다.  
$ pip install django-jet


# settings.py
INSTALLED_APPS = (
    ...
    'jet',

# urls.py
urlpatterns = patterns(
    '',
    url(r'^jet/', include('jet.urls', 'jet')),  # Django JET URLS

$ ./manage.py migrate jet
$ ./manage.py collectstatic
{% endhighlight %}

<br><br>

css 변경이 필요해 살펴보니 .scss 확장자명을 가진 파일이 보였다.  
sass라는 CSS를 더욱 효율적으로 사용토록 해주는 녀석이다.  
이 녀석을 쓰려면 sass를 설치해야 하고, ruby로 만들어졌기 때문에 linux에 luby 설치가 필요했다.  
{% highlight ruby %}
$ yum install rubygems
$ gem install sass

# scss 파일 수정후
$ sass --watch a.scss:a.css
{% endhighlight %}

<br><br>

local과 dev, prd 서버가 각기 다른 migration 파일을 가지고 있어서 혼란스러운 상황.  
jenkins이 migration 폴더까지 덮어씌우고 있기 때문에 **앞으로는 migration 폴더까지 정리해서 반영할 필요가 있다.**  
이미 지워진 table을 지우려고 하는가 하면, 중요 테이블관련하여 foeignkey 연관된 것을 삭제할까 물어보기도 한다. 일단 no..  
**이미 생성된 table때문에 migrate가 되지 않는 경우에는 ./manage.py migrate --fake 를 사용하면 된다.**  
{% highlight ruby %}
# migrate 디렉토리가 jenkins소유로 되어 있어 makemigrations하는데 유저 변경이 필요했다.
sudo chown -R username:group directory

# 배포할때 유저를 원복해야 한다.
{% endhighlight %}

<br><br>

[Django Jet]: http://jet.readthedocs.io/en/latest/install.html
[sass]: http://sass-lang.com/
[luby-sass-install]: https://www.cmsfactory.net/node/11750

{% highlight ruby %}
# jet의 경우 edit page에서 previous/next item으로 넘어가는 기능을 제공한다.
# 이 때 full scan이 일어나 60만건이 넘는 데이터를 읽는데 성능이 매우 안좋아진다.
# 해당 내용 부분을 삭제했다.

# myvenv/lib/python3.5/site-packages/jet/templates/admin/base.html
   if change and show_siblings
        <div class="changeform-navigation">
            ...
        </div>
   endif
{% endhighlight %}


<br><br><br>


장고 튜토리얼 잘된 번역 사이트  

<br><br><br>

# 장고 튜토리얼 사이트  
https://tutorial.djangogirls.org/ko/django_orm/  
http://djangogo.tistory.com/entry/Djangogo-6-%EB%AA%A8%EB%8D%B8%EC%97%90-%EB%A7%9E%EB%8A%94-Django%EA%B4%80%EB%A6%AC%EC%82%AC%EC%9D%B4%ED%8A%B8-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0  
https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Admin_site  

<br><br><br>

# 장고 packages  
https://djangopackages.org/packages/p/django-autocomplete-light/


<br><br><br>

{% highlight ruby %}
{% endhighlight %}















.
