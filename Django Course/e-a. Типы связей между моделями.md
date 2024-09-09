Один ко многим - ForeignKey

Многие ко многим - ManyToManyField

Один к одному - OneToOneField

### Один ко многим - ForeignKey

Тип связи "Один ко многим" (или ForeignKey в Django) представляет отношение, при котором каждый объект одной модели связан с одним объектом другой модели, но объекты второй модели могут быть связаны с несколькими объектами первой модели.

1. **Пример с двумя моделями: Customers и Orders**.
    
    У каждого пользователя может быть несколько заказов, но каждый заказ принадлежит только одному пользователю.
    
    [![](https://res.cloudinary.com/practicaldev/image/fetch/s--ZGfb3_y5--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://i.postimg.cc/NM7wPYx6/one-to-many-erd.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--ZGfb3_y5--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://i.postimg.cc/NM7wPYx6/one-to-many-erd.png)
    
2. **Реализация:**
    
    ```Python
    from django.db import models
    
    class Customers(models.Model):
        name = models.CharField(max_length=100)
    
    class Orders(models.Model):
        customer = models.ForeignKey(User, on_delete=models.CASCADE)
    ```
    
    - В модели Orders определено поле ForeignKey, которое связывает каждый заказ с объектом пользователя. Это поле указывает на модель Customers и определяет тип связи.
    - Параметр `on_delete=models.CASCADE` указывает, что при удалении пользователя все связанные с ним заказы также должны быть удалены.
    - За кулисами Django добавляет "_id" к имени поля, чтобы создать имя столбца базы данных. В данном примере поле ForeignKey создает в базе данных столбец с именем `customer_id`, который содержит идентификатор пользователя, связанного с каждым заказом.
3. **Преимущества:**
    - Позволяет устанавливать однозначное отношение между объектами разных моделей.
    - Обеспечивает простоту и эффективность доступа к связанным объектам.
    - Упрощает структурирование данных и выполнение запросов к базе данных.

### Многие ко многим - **ManyToManyField**

Тип связи "Многие ко многим" (или ManyToManyField в Django) представляет отношение, при котором каждый объект одной модели может быть связан с несколькими объектами другой модели, и наоборот.

1. **Пример с двумя моделями: Movie и Actor:**
    
    У каждого фильма может быть несколько актеров, и каждый актер может участвовать в нескольких фильмах.
    
    [![](https://miro.medium.com/v2/resize:fit:1400/1*edu3GQjYMIOGxlowoOVuUQ.png)](https://miro.medium.com/v2/resize:fit:1400/1*edu3GQjYMIOGxlowoOVuUQ.png)
    
2. **Реализация:**
    
    ```Python
    from django.db import models
    
    class Movie(models.Model):
        movie_title = models.CharField(max_length=100)
        actors = models.ManyToManyField('Actor')
    
    class Actor(models.Model):
        name = models.CharField(max_length=100)
    ```
    
    - В модели Movie определено поле ManyToManyField, которое связывает каждый фильм с несколькими актерами (ссылается на модель `‘Actor’`)
    - Взаимосвязь между моделями Movie и Actor обрабатывается в Django автоматически с использованием промежуточной таблицы в базе данных, которая отслеживает связи между объектами двух моделей.
3. **Преимущества:**
    - Позволяет описывать сложные отношения между объектами различных моделей.
    - Обеспечивает гибкость при работе с данными, так как позволяет добавлять и удалять связанные объекты без изменения схемы базы данных.
    - Упрощает выполнение запросов, связанных с данными, которые имеют многие ко многим отношения.

### Один к одному - **OneToOneField**

Тип связи "Один к одному" (или OneToOneField в Django) представляет отношение, при котором каждый объект одной модели связан только с одним объектом другой модели, и наоборот.

1. **Пример с двумя моделями: UserProfile и User:**
    
    Каждый пользователь может иметь только один профиль, и каждый профиль связан только с одним пользователем.
    
    [![](https://i.stack.imgur.com/XHygZ.png)](https://i.stack.imgur.com/XHygZ.png)
    
2. **Реализация:**
    
    ```Python
    from django.db import models
    from django.contrib.auth.models import User
    
    class UserProfile(models.Model):
        user = models.OneToOneField(User, on_delete=models.CASCADE)
        custom_url = models.CharField(max_length=50, blank=True)
        biography = models.CharField(max_length=50, blank=True)
    ```
    
    - В модели UserProfile определено поле OneToOneField, которое связывает каждый профиль с одним пользователем из модели User.
    - `on_delete=models.CASCADE` указывает, что при удалении пользователя также будет удален его профиль. Это может быть изменено в зависимости от требований приложения.
    - Связь "Один к одному" обычно используется для создания дополнительных профилей или расширенной информации о существующих пользователях.
3. **Преимущества:**
    - Гарантирует наличие только одного связанного объекта для каждого объекта модели.
    - Упрощает доступ к дополнительным данным о сущностях, таких как профили пользователей или дополнительные сведения о продуктах.
    - Обеспечивает эффективное использование базы данных, поскольку связь описывается через уникальные ключи и индексы.

<div class="page-break" style="page-break-before: always;"></div>
