# python-web-app

First, make sure you have Django installed. You can install it using pip:

```
pip install django
```

Once Django is installed, you can create a new Django project:

```
django-admin startproject medicine_tracking
```

Change to the project directory:

```
cd medicine_tracking
```

Now, let's create a Django app for our web application:

```
python manage.py startapp medicine_app
```

Next, open the `medicine_tracking/settings.py` file and add `'medicine_app'` to the `INSTALLED_APPS` list.

```python
INSTALLED_APPS = [
    ...
    'medicine_app',
]
```

Now, let's define the models for our web application. Create a file called `medicine_app/models.py` and add the following code:

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)

class MedicineDispatch(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    details = models.TextField()
    date = models.DateField(auto_now_add=True)
```

Next, we need to create the database tables for our models:

```
python manage.py makemigrations
python manage.py migrate
```

Now, let's create the views for our web application. Create a file called `medicine_app/views.py` and add the following code:

```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required
from .models import User, MedicineDispatch

def register(request):
    if request.method == 'POST':
        username = request.POST['username']
        email = request.POST['email']
        password = request.POST['password']
        User.objects.create_user(username=username, email=email, password=password)
        return redirect('login')
    return render(request, 'register.html')

def user_login(request):
    if request.method == 'POST':
        email = request.POST['email']
        password = request.POST['password']
        user = authenticate(request, email=email, password=password)
        if user is not None:
            login(request, user)
            return redirect('dashboard')
        else:
            return redirect('login')
    return render(request, 'login.html')

@login_required
def dashboard(request):
    if request.user.is_superuser:
        users = User.objects.all()
        return render(request, 'admin_dashboard.html', {'users': users})
    else:
        dispatches = MedicineDispatch.objects.filter(user=request.user)
        return render(request, 'user_dashboard.html', {'dispatches': dispatches})

@login_required
def logout_view(request):
    logout(request)
    return redirect('login')

@login_required
def post_dispatch(request):
    if request.method == 'POST':
        details = request.POST['details']
        MedicineDispatch.objects.create(user=request.user, details=details)
        return redirect('dashboard')
    return render(request, 'post_dispatch.html')
```

Next, let's create the URL patterns for our views. Open the `medicine_app/urls.py` file and add the following code:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('register/', views.register, name='register'),
    path('login/', views.user_login, name='login'),
    path('dashboard/', views.dashboard, name='dashboard'),
    path('logout/', views.logout_view, name='logout'),
    path('post_dispatch/', views.post_dispatch, name='post_dispatch'),
]
```

Now, let's create the HTML templates for our web application. Create a directory called `medicine_app/templates` and inside it, create the following files:

1. `register.html`:

```html
<h1>Register</h1>
<form method="post">
  {% csrf_token %}
  <input type="text" name="username" placeholder="Username" required><br>
  <input type="email" name="email" placeholder="Email" required><br>
  <input type="password" name="password" placeholder="Password" required><br>
  <button type="submit">Register</button>
</form>
```

2. `login.html`:

```html
<h1>Login</h1>
<form method="post">
  {% csrf_token %}
  <input type="email" name="email" placeholder="Email" required><br>
  <input type="password" name="password" placeholder="Password" required><br>
  <button type="submit">Login</button>
</form>
```

3. `admin_dashboard.html`:

```html
<h1>Admin Dashboard</h1>
<ul>
  {% for user in users %}
    <li>{{ user.username }} - {{ user.email }}</li>
  {% endfor %}
</ul>
```

4. `user_dashboard.html`:

```html
<h1>User Dashboard</h1>
<ul>
  {% for dispatch in dispatches %}
    <li>{{ dispatch.details }} - {{ dispatch.date }}</li>
  {% endfor %}
</ul>
```

5. `post_dispatch.html`:

```html
<h1>Post Dispatch</h1>
<form method="post">
  {% csrf_token %}
  <textarea name="details" placeholder="Details" required></textarea><br>
  <button type="submit">Post Dispatch</button>
</form>
```

Finally, open the `medicine_tracking/urls.py` file and add the following code:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('medicine_app.urls')),
]
```

That's it! You have now created a web application in Python using the Django framework. You can run the application using the following command:

```
python manage.py runserver
```

You can access the web application in your browser at `http://localhost:8000/`.
