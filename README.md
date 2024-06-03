# rest-web-employ

First, ensure you have Django installed. You can install it via pip:

```bash
pip install django
```

Now, let's create a new Django project:

```bash
django-admin startproject employee_management_project
```

Navigate into the project directory:

```bash
cd employee_management_project
```

Create a new Django app for employee management:

```bash
python manage.py startapp employees
```

Now, let's define our models, views, serializers, and URLs.

1. Models (`employees/models.py`):

```python
from django.contrib.auth.models import User
from django.db import models

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email = models.EmailField()

class Employee(models.Model):
    name = models.CharField(max_length=100)
    department = models.CharField(max_length=100)
    designation = models.CharField(max_length=100)
    salary = models.DecimalField(max_digits=10, decimal_places=2)
    joining_date = models.DateField()

    def __str__(self):
        return self.name
```

2. Serializers (`employees/serializers.py`):

```python
from rest_framework import serializers
from .models import Profile, Employee

class ProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = '__all__'

class EmployeeSerializer(serializers.ModelSerializer):
    class Meta:
        model = Employee
        fields = '__all__'
```

3. Views (`employees/views.py`):

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated
from .models import Profile, Employee
from .serializers import ProfileSerializer, EmployeeSerializer
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token
from rest_framework.authtoken.views import ObtainAuthToken
from django.contrib.auth import authenticate, login as auth_login

@api_view(['POST'])
def signup(request):
    username = request.data.get('username')
    password = request.data.get('password')
    email = request.data.get('email')

    if not username or not password:
        return Response({"message": "Username and password are required."}, status=status.HTTP_400_BAD_REQUEST)

    if User.objects.filter(username=username).exists():
        return Response({"message": "Username already exists."}, status=status.HTTP_400_BAD_REQUEST)

    user = User.objects.create_user(username=username, password=password, email=email)
    if user:
        auth_login(request, user)
        token, created = Token.objects.get_or_create(user=user)
        return Response({"token": token.key}, status=status.HTTP_201_CREATED)
    else:
        return Response({"message": "Signup failed."}, status=status.HTTP_400_BAD_REQUEST)

@api_view(['POST'])
def login(request):
    username = request.data.get('username')
    password = request.data.get('password')

    user = authenticate(request, username=username, password=password)
    if user is not None:
        auth_login(request, user)
        token, created = Token.objects.get_or_create(user=user)
        return Response({"token": token.key}, status=status.HTTP_200_OK)
    else:
        return Response({"message": "Invalid credentials."}, status=status.HTTP_401_UNAUTHORIZED)

@api_view(['POST'])
def create_profile(request):
    if request.user.is_authenticated:
        serializer = ProfileSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(user=request.user)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    else:
        return Response({"message": "Authentication credentials were not provided."}, status=status.HTTP_401_UNAUTHORIZED)

@api_view(['POST'])
@permission_classes([IsAuthenticated])
def update_profile(request):
    try:
        profile = Profile.objects.get(user=request.user)
    except Profile.DoesNotExist:
        return Response({"message": "Profile does not exist."}, status=status.HTTP_404_NOT_FOUND)

    serializer = ProfileSerializer(profile, data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def list_employees(request):
    employees = Employee.objects.all()
    serializer = EmployeeSerializer(employees, many=True)
    return Response(serializer.data)

@api_view(['PUT', 'PATCH'])
@permission_classes([IsAuthenticated])
def update_own_profile(request):
    try:
        profile = Profile.objects.get(user=request.user)
    except Profile.DoesNotExist:
        return Response({"message": "Profile does not exist."}, status=status.HTTP_404_NOT_FOUND)

    serializer = ProfileSerializer(profile, data=request.data, partial=True)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

4. URLs (`employees/urls.py`):

```python
from django.urls import path
from . import views

urlpatterns = [
    path('signup/', views.signup, name='signup'),
    path('login/', views.login, name='login'),
    path('create_profile/', views.create_profile, name='create_profile'),
    path('update_profile/', views.update_profile, name='update_profile'),
    path('list_employees/', views.list_employees, name='list_employees'),
    path('update_own_profile/', views.update_own_profile, name='update_own_profile'),
]
```

5. Settings (`employee_management_project/settings.py`):

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'employees',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
```

6. Ensure to include the URLs of your app in the main project's `urls.py`.

That's the complete code for your Django project from scratch, with user authentication, login, signup, profile creation and update, and employee management features. Let me know if you need further assistance!
