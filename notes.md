# First Django Blog

## Virtual Environment

`python3 -m venv myvenv` -> Creates a Virtual Environment
<br>

`source myvenv/bin/activate` -> To start the Virtual Environment

### Changing Settings

TO make Changes in the settings of your Django project head over to: `mysite/settings.py`
<br>
Like....

```py
TIME_ZONE = 'India/Kolkata'
```

The following code snippet is for a 'path' for static files.

```py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'static'
```

When `DEBUG` is `True` and `ALLOWED_HOSTS` is empty, the host is validated against `['localhost'], '127.0.0.1', '[::1]'`, but this won't match our hostname on "PythonAnywhere" once our application is deployed so we are gonna change the following setting:

```py
ALLOWED_HOST = ['127.0.0.1', '.pythonanywhere.com']
```

We can also set database of our preference by changing:

```py
DATABASE = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

### Starting the web server

`python3 manage.py runserver` -> Runs the server

`https://127.0.0.1:8000` -> Run it on browser

## Django Model

### Creating an application

To keep everything tidy, we should create a separate application inside our project.

```sh
python3 manage.py strtapp blog
```

### Creating a blog post model

In `blog/models.py` we will add a `Model`

```py
from django.conf import settings
from django.db import models
from django.utils import timezone

class Post(models.Model):
    author = models.ForeginKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```

Django's documentation: https://docs.djangoproject.com/en/3.2/ref/models/fields/#field-types
<br>

### Create table for models in our database

```sh
manage.py makemigrations blog
```

## Django Admin

In `blog/admin.py` add:

```py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

We need to register the model with `admin.site.register(Post)` to make our code visible on the admin page.
<br>

Now run `python3 manage.py runserver` in the console and go to this address on the browser: http://127.0.0.1:8000/admin/
<br>

Now to log in we need to create a superuser account.
<br>

To create SU: `python manage.py createsuperuser`
<br>

Let's return to our browser.
We will now add some posts to our blog.. Try add multiple of them, Some with published dates and others without published dates. It will be helpful later.
<br>

Django admin: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/

## Django URLs

### How do URLs work in Django?

Open `mysite/urls.py` and this would be there:

```py
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]
```

The admin URL, which we visited in the previous chapter, is already here:

```py
path('admin/', admin.site.urls),
```

This line means that for every URL that starts with `admin/`, Django will find a corresponding _view_. In this case, we're including a lot of admin URLs so it isn't all packed into this small file - it's more readable and cleaner.

### Your first Django URL!

We want 'https://127.0.0.1:8000' to be the home page of our blog and to display a list of posts.
<br>

We also want to keep the `mysite/urls.py` file clean, so we will import URLs from our `blog` application to the main `mysite/urls.py` file.

So to do that.. Let's make some changes to `mysite/urls.py`

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls'))
]
```

Django will now redirect everything that comes into 'https://127.0.0.1:8000' to `blog.urls` and looks for further instructions there.

### blog.urls

Create a new empty file named `urls.py` in the `blog` directory, and open it in the code editor.
<br>

Add these lines:

```py
from django.urls import path
from . import views

urlpatterns - [
    path('', views.post_list, name = 'post_list'),
]
```

Now, We are assigning a `view` called `post_list` to the root URL. This URL pattern will match an empty string and the Django URL resolver will ignore the domain name (i.e., https://127.0.0.1:8000/)that prefixes the full URL path. This pattern will tell Django that `views.port_list` is the right place to go if someone enters your website at the 'https://127.0.0.1:8000/'
<br>

The last part, `name='post_list'`, is the name of the URL that will bw used to identify the view. This can be the same as the name of the view but it can also be something completely different. We will be using the named URLs later in the project, so it is important to name each URL in the app. We should also try to keep the names of URLs unique and easy to remember.

## Django views

A _view_ is a place where we put lhe "logic" of our application. It will request information from the `model` you created before and pass it to a `template`. We'll create a template in the next chapter. Views are just Python function.

Go to `blog/views.py`

It should look like this..

```py
from django.shortcuts import render

# Create your views here.
```

Add.. The following code snippet here..

```py
def post_list(request):
    return render(request, 'blog/post_list.html', {})
```

We created a function called `post_list` that takes `request` and will `return` the value it gets from calling another function `render` that will render out template `blog/post_list.html`.

## Create a template

Templates are stored in `blog/templates/blog`.
<br>

Well there is something weird above.. As we can see.. There are two `blog`, well it's a useful naming convention that makes life easier when things start to get more complicated.
<br>

And now make a `post_list.html` file in `blog/templates/blog` directory.
<br>

Now add some html and possibly some css as well.

## Django ORM and QuerySets

### What is a QuerySet?

A QuerySet is, in essence, a list of objects of a given Model. QuerySets allow you to read the data from the database, filter it and order it.

### Django Shell

Type this command in the terminal:

```sh
python3 manage.py shell
```

### All objects

Show all our posts:

```py
Post.objects.all()
```

It would show an error message, as we have not imported anything.

```py
from blog.models import Post
```

Now the above command will run without any errors.

### Create Object

Create a new Post object in Database:

```py
Post.objects.create(author=me, title='Sample title', text='Test')
```

Now to define the `me` there..

```py
from django.contrib.auth.models import User
```

What users do we have in our database? Try this:

```py
User.objects.all()
```

Let's make an instance of the user.

```py
me = User.objects.get(username='himanshu')
```

Now to create the post itself:

```py
Post.objects.create(author=me, title='Simple Title', text='Test')
```

Now to check if it works..

```py
Post.object.all()
```

YEA.. It should be there..

### Add more posts

If you want to..

### Filter objects

A big part of QuerySets is the ability to filter them. Let's say we want to find a posts that user himanshu authored. We will use `filter` instead of `all` in `Post.objects.all()`. In parentheses we state what condition(s) a blog post needs to meet to end up in our queyset. In our case, the condition is that `author` should be equal to `me`. The way to write it in Django is `author=me`.

```py
Post.objects.filter(author=me)
```

Or maybe...

```py
Post.objects.filter(title__contains='title')
```

**Note:** There is two underscore characters between `title` and `contains`. Django's ORM uses this rule to separate filed names ("title") and operations or filters ("contains"). If we use only one underscore, you'll get an error like "FieldError: Cannot resolve keyword title_contains".
<br>

We can also get a list of all published posts. We will do this by filtering all the posts that have `published_data` set in the past:

```py
from django.utils import timezone
Post.objects.filter(published_date__lte=timezone.now())
```

The post that we added right now in Python Console is not published yet, but we can change that...
<br>

First get an instance of a post we want to publish:

```py
post = Post.objects.get(title='Sample Title')
```

And then publish it with our `publish` method:

```py
post.publish()
```

### Ordering objects

QuerySet also allows you to order the list of objects. Let's try to order them by `created_date` field:

```py
Post.objects.order_by('created_date')
```

We can also reverse the ordering by adding `-` at the beginning:

```py
Post.objects.order_by('-created_date')
```

### Complex queries through method-chaining

We can even chain multiple queries.

## Dynamic data in templates

We have everything now.. `Post` model is defined in `models.py`, we have `post_list` in `views.py` and the template added. But how to make them appear in our HTML template...
<br>

Views are used just for this..Connect models and templates. In our `post_list` _view_ we will need to take the models we want to display and pass them to the template. In a _view_ we decide what (model) will be displayed in a template.

Head over to `blog/views.py` and open `post_list`:

```py
from django.shortcuts import render

def post_list(requests):
    return render(request, 'blog/post_list.html', {})
```

Now to add the model.. `models.py`

```py
from django.shortcuts import render
from .models import Post
```

### QuerySet

Now adding the Publishing command to the file `blog/views.py`:

```py
from django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {})
```

Now to pass the `posts` QuerySet to the template context, by changing the `render` function call.

```py
rom django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

QuerySet Documentation: https://docs.djangoproject.com/en/3.2/ref/models/querysets/

## Django Templates

To print a variable in Django templates, we use double curly brackets with the variable's name inside like in `blog/templates/blog/post_list.html`:

```py
{{ posts }}
```

Everything we put in between `{% for %}` and `{5 endfor %}` will be repeated for each object in the list.

```html
<header>
  <h1><a href="/">Django Girls Blog</a></h1>
</header>

{% for post in posts %}
<article>
  <time>published: {{ post.published_date }}</time>
  <h2><a href="">{{ post.title }}</a></h2>
  <p>{{ post.text|linebreaksbr }}</p>
</article>
{% endfor %}
```

Have you noticed that we used a slightly different notation this time (`{{ post.title }}` or `{{ post.text }}`)? We are accessing data in each of the fields defined in our `Post` model. Also, the `|linebreaksbr` is piping the posts' text through a filter to convert line-breaks into paragraphs.

## CSS

### Bootstrap

```html
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
  integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
  crossorigin="anonymous"
/>
```
