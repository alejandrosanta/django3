## Data y Modelos

### 1. Crear Modelo
> app_name/models.py
```python
class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField()
```

### 2. Correr Migracion
`$ python3 manage.py makemigrations`
`$ python3 manage.py migrate`

### 3. Insertar Data
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book`
`>>> harry_potter = Book(title="Harry Potter 1", rating=5)`
`>>> harry_potter.save()`
`>>> lord_of_the_rings = Book(title="Lord of the Rings", rating=4)`
`>>> lord_of_the_rings.save()`

### 4. Actualizar Modelo y Migrar
> app_name/models.py
```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.urls import reverse

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1),MaxValueValidator(5)])
    author = models.CharField(null=True, max_length=100)
    is_bestselling = models.BooleanField(default=False)

    def get_absolute_url(self):
        return reverse("book-detail", args=[self.id])

    def __str__(self):
        return f"{self.title} ({self.rating})"
```
`$ python3 manage.py makemigrations`
`$ python3 manage.py migrate`

### 5. Ver Data
* Add slug and overwrite save
> app_name/models.py
```python
from django.core import validators
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.urls import reverse
from django.utils.text import slugify

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1),MaxValueValidator(5)])
    author = models.CharField(null=True, max_length=100)
    is_bestselling = models.BooleanField(default=False)
    slug = models.SlugField(default="", null=False, db_index=True)

    def get_absolute_url(self):
        return reverse("book-detail", args=[self.id])

    def save(self, *args, **kwargs):
        self.slug = slugify(self.title)
        super().save(*args, **kwargs)

    def __str__(self):
        return f"{self.title} ({self.rating})"
```
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book`
`>>> Book.objects.get(title="Lord of the Rings").slug`
`>>> Book.objects.get(title="Lord of the Rings").save()`
`$ python3 manage.py shell`

* Ver Data
`>>> from book_outlet.models import Book`
`>>> Book.objects.all()`
> app_name/templates/app_name/index.html
```html
{% extends "book_outlet/base.html" %}

{% block title %}
    All Books
{% endblock %}
{% block content %}
    <ul>
        {% for book in books %}
            <li><a href="{{ book.get_absolute_url }}">{{ book.title }}</a> (Rating: {{ book.rating }})</li>
        {% endfor %}
    </ul>
{% endblock %}
```
> app_name/views.py
```python
from django.shortcuts import get_object_or_404, render
from django.http import Http404

from .models import Book

def index(request):
    books = Book.objects.all()
    return render(request, "book_outlet/index.html", {
        "books": books
    })

def book_detail(request, slug):
    # try:
    #     book = Book.objects.get(pk=id)
    # except:
    #     raise Http404()
    book = get_object_or_404(Book, slug=slug)
    return render(request, "book_outlet/book_detail.html", {
        "title":book.title,
        "author":book.author,
        "rating":book.rating,
        "is_bestseller":book.is_bestselling
    })
```
> app_name/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index),
    path("<slug:slug>", views.book_detail, name="book-detail")
]
```
> app_name/templates/app_name/book_detail.html
```html
{% extends "book_outlet/base.html" %}

{% block title %}
    {{ title }}
{% endblock %}
{% block content %}
    <h1>{{ title }}</h1>
    <h2>{{ author }}</h2>
    <p>The book has a rating of {{ rating }}
        {% if is_bestseller %}
            and is a bestseller.
        {% else %}
            but is not a bestseller.
        {% endif %}
    </p>
{% endblock %}
```

### 6. Actualizar Data
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book`
`>>> harry_potter = Book.objects.all()[0]`
`>>> harry_potter.author = "J. K. Rowling"`
`>>> harry_potter.save()`
`>>> Book.objects.all()[0].author`

### 7. Eliminar Data
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book`
`>>> harry_potter = Book.objects.all()[0]`
`>>> harry_potter.delete()`
* Eliminar todo
`>>> Book.objects.all().delete()`

### 8. Querying y Filtrando Data
`>>> Book.objects.get(rating=4)`
`>>> Book.objects.get(title="Hola")`
Cuando se utiliza get se debe utilizar condiciones que solo match un solo registro

`>>> Book.objects.filter(is_bestselling=False)`
* Condition and
`>>> Book.objects.filter(is_bestselling=True, rating=2)`
* Condition or
`>>> from django.db.models import Q`
`>>> Book.objects.filter(Q(rating__lt=5) | Q(is_bestselling=True))`

`>>> Book.objects.filter(rating__lt=5)`
`>>> Book.objects.filter(title__contains="Lord")`
Cuando se utiliza filter puede encontrar varios registros

Para mejorar el performance almacenar las consultas en una variable


## Admin
### 1. Crear super usuario
`python3 manage.py createsuperuser`
> http://127.0.0.1:8000/admin/

### 2. Configurar las opciones de Administracion
```python
from django.contrib import admin
from .models import Book

class BookAdmin(admin.ModelAdmin):
    prepopulated_fields = {"slug":("title",)}
    list_filter = ("author", "rating",)
    list_display = ("title", "author",)

admin.site.register(Book, BookAdmin)
```
## Relationships
### 1. Una a Muchos
```python
class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1),MaxValueValidator(5)])
    author = models.ForeignKey(Author, on_delete=models.CASCADE, null=True, related_name="libros")
    is_bestselling = models.BooleanField(default=False)
    slug = models.SlugField(default="", blank=True, null=False, db_index=True)
```
* Creando la relacion
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book, Author`
`>>> alejandro = Author(first_name="Alejandro", last_name="Santa Arciniegas")`
`>>> alejandro.save()`
`>>> ml= Book(title="My Life", rating= 5, is_bestselling=True, slug="my_life", author=alejandro)`
`>>> ml.save()`

* Filtrando
`>>> ab = Book.objects.filter(author__first_name="Alejandro")`
`>>> ab = Book.objects.filter(author__first_name__contains="Aleja")`

* Obteniendo todos los libros de un autor
`>>> ale = Author.objects.get(first_name="Alejandro")`
`>>> ale.libros.all()`
`>>> ale.libros.filter(rating__gt=3)`

### 2. Una a una

```python
class Address(models.Model):
    street  = models.CharField(max_length=80)
    postal_code = models.CharField(max_length=5)
    city = models.CharField(max_length=50)

class Author(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    address = models.OneToOneField(Address, on_delete=models.CASCADE, null= True)
```

### 3. Muchos a muchos

```python
class Country(models.Model): 
    name  = models.CharField(max_length=80)
    code  = models.CharField(max_length=2)

class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField(
        validators=[MinValueValidator(1),MaxValueValidator(5)])
    author = models.ForeignKey(Author, on_delete=models.CASCADE, null=True, related_name="libros")
    is_bestselling = models.BooleanField(default=False)
    slug = models.SlugField(default="", blank=True, null=False, db_index=True)
    published_countries = models.ManyToManyField(Country, null=False, related_name="book_country")
```
`$ python3 manage.py shell`
`>>> from book_outlet.models import Book, Country`
`>>> germany = Country(name="Germany", code="DE")`
`>>> germany.save()`
`>>> ml.published_countries.add(germany)`
`>>> germany.book_set.all()`
`>>> germany.book_country.all()`
