Тип связи "Один к одному" (или OneToOneField в Django) представляет отношение, при котором каждый объект одной модели связан только с одним объектом другой модели, и наоборот. Данный тип связи чаще всего используется для расширения базовой модели (например, для модели юзера добавляется модель личного кабинета и т.д.).

Концептуально эта связь похожа на ForeignKey с `unique=True`, но "обратная" сторона отношения будет напрямую возвращать один объект (здесь также, как и в FK за это отвечает атрибут `related_name`, но там по умолчанию возвращался set, а здесь только один объект; если не задать related_name, то по умолчанию будет использоваться имя текущей модели с маленькой буквы).

Синтаксис:

```Python
OneToOneField(to, on_delete, parent_link=False, **options)
```

`OneToOneField` принимает все дополнительные аргументы, которые принимает `ForeignKey`, плюс один дополнительный аргумент:

- `parent_link` - когда мы наследуем одну модель от другой, то Django неявно создаёт поле `OneToOneField` для этой связи. Вот в таком случае, чтобы явно указать Django, что нужно использовать именно данное поле и используется данный параметр.
    
    Пример:
    
    ```Python
    from django.db import models
    
    class Women(models.Model):
        name = models.CharField(max_length=100)
    		...
    
    class Husband(Women):
        age = models.IntegerField(null=True)
    ```
    
    Здесь Django автоматически создаст скрытое поле `OneToOneField` для связи между `Women` и `Husband`.
    
    ```Python
    from django.db import models
    
    class Women(models.Model):
        name = models.CharField(max_length=100)
    		...
    
    class Husband(Women):
        age = models.IntegerField(null=True)
    		wife = models.OneToOneField('Women', on_delete=models.SET_NULL, parent_link=True, null=True, blank=True, related_name='husband')
    ```
    
    В этом случае мы явно указываем `OneToOneField` с параметром `parent_link=True` для поля `wife`. Это означает, что это поле будет использоваться в качестве обратной связи с родительским классом (`Women`), а не создастся скрытое поле `OneToOneField`.
    

**Для добавления связи используем shell:**

```Shell
w = Women.objects.get(pk=1)
h = Husband.objects.create('Jack', 32)
w.husband = h
w.save()
```

После операции добавления связи обязательно сохраняем изменения с помощью метода `save()`.

<div class="page-break" style="page-break-before: always;"></div>
