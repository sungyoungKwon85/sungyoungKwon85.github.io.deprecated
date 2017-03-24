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








# Django Admin Excel exporting  
https://django-import-export.readthedocs.io/en/latest/installation.html  
{% highlight ruby %}
pip install django-import-export

# settings.py
INSTALLED_APPS = (
    ...
    'import_export',
)
{% endhighlight %}






<br><br><br>






# Tags + autocomplte  
https://godjango.com/33-tagging-with-django-taggit/  
http://django-taggit.readthedocs.io/en/latest/forms.html  
http://django-autocomplete-light.readthedocs.io/en/3.1.3/tutorial.html#using-autocompletes-in-the-admin  
{% highlight ruby %}
pip install django-taggit

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
