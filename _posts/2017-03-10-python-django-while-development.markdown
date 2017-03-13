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
class ClippedSeedDataAdmin(admin.ModelAdmin):
    list_display = ('seed_uid', 'raw_uid', 'question', 'answer', 'staff_seq', 'updated')
    list_display_links = ('question',)
    ordering = ('-updated',)
    search_fields = ('question',)

    def has_add_permission(self, request):
        return False


admin.site.register(ClippedSeedData, ClippedSeedDataAdmin)
{% endhighlight %}

<br><br><br>


# How to show join results to detail view(edit view)  
http://brownbears.tistory.com/101  
{% highlight ruby %}
# models.py
class ClippedRawData(models.Model):
    raw_uid = models.CharField(primary_key=True, max_length=32)
    ...


class ClippedRawCommentData(models.Model):
    comment_uid = models.CharField(primary_key=True, max_length=32, editable=False)
    raw_uid = models.ForeignKey(ClippedRawData, db_column='raw_uid')
    ...


# admin.py
class CommentDataInline(admin.TabularInline):
    model = ClippedRawCommentData
    fk_name = 'raw_uid'
    extra = 0
    ...


class ClippedRawDataAdmin(admin.ModelAdmin):
    ...
    inlines = [CommentDataInline, SeedDataInline, ]
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


# Show realated data to ModelAdmin  
{% highlight ruby %}
    def get_queryset(self, request):
        qs = super(ClippedRawDataAdmin, self).get_queryset(request)
        return qs.annotate(comment_count=Count('clippedrawcommentdata'), seed_count=Count('clippedseeddata'))

    def get_comment_count(self, obj):
        return obj.comment_count
    get_comment_count.short_description = 'comment count'
{% endhighlight %}


<br><br><br>


# Multiple columns for FOREIGNKEY in DJANGO  
https://pypi.python.org/pypi/django-composite-foreignkey   

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



장고 튜토리얼 잘된 번역 사이트  
https://tutorial.djangogirls.org/ko/django_orm/  
http://djangogo.tistory.com/entry/Djangogo-6-%EB%AA%A8%EB%8D%B8%EC%97%90-%EB%A7%9E%EB%8A%94-Django%EA%B4%80%EB%A6%AC%EC%82%AC%EC%9D%B4%ED%8A%B8-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0  
<br><br><br>

# 장고 튜토리얼 영문 사이트  
https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Admin_site  

<br><br><br>

# 장고 어드민 커스터마이즈 좋은 사이트(한글)  
https://milooy.wordpress.com/2016/07/27/django-admin-customizing/  




<br><br><br>



{% highlight ruby %}

{% endhighlight %}















.
