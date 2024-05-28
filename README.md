### 1. Crear Modelo
> app_name/models.py
from django.db import models

```python
class Book(models.Model):
    title = models.CharField(max_length=50)
    rating = models.IntegerField()
```

### 2. Correr Migracion
`$ python3 manage.py makemigrations`
`$ python3 manage.py migrate`

### 3. Insertar Data





