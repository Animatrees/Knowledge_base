Описание

Наследование

ModelFormMixin

BaseCreateView

Краткое summery

### Описание

**CreateView** - это класс-представление в Django, который отображает форму для создания объекта, повторно отображает форму с ошибками валидации (если таковые имеются) и сохраняет объект. Он наследуется от нескольких миксинов и базовых классов, таких как SingleObjectTemplateResponseMixin, BaseCreateView, ModelFormMixin и других.

**Преимущества использования CreateView:**

- **Уменьшение количества кода:** CreateView берет на себя большую часть работы по созданию и обработке форм, что позволяет писать меньше кода.
- **Встроенная проверка данных:** CreateView автоматически проверяет данные формы перед сохранением объекта в базе данных.
- **Легкая настройка:** Вы можете легко настроить поведение CreateView, переопределив его методы и атрибуты.

**Пример использования CreateView:**

Допустим, у нас есть модель `Author`:

```Python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
```

Чтобы создать страницу с формой для добавления новых авторов, мы можем использовать CreateView:

```Python
from django.views.generic import CreateView
from .models import Author

class AuthorCreateView(CreateView):
    model = Author
    fields = ['name']
```

Здесь мы определяем класс `AuthorCreateView`, наследующийся от `CreateView`. Мы указываем модель `Author` и поля формы `fields`. Мы можем указать `'__all__'` для параметра `fields`, тогда будут отображены все поля.

По умолчанию CreateView использует шаблон с именем `myapp/author_form.html`. Мы можем изменить это поведение, переопределив атрибут `template_name_suffix` (или указав вместо этого атрибут `template_name`)

```Python
class AuthorCreateView(CreateView):
    model = Author
    fields = ['name']
    template_name_suffix = '_create_form'
```

Теперь представление будет использовать шаблон `myapp/author_create_form.html`.

В шаблоне мы можем отобразить форму:

```HTML
<form method="post">{% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="Save">
</form>
```

При использовании CreateView мы имеем доступ к `self.object`, который является создаваемым объектом. Если объект еще не создан, то значение будет `None`. Мы также имеем доступ к самой форме в шаблоне через переменную `form`.

### Наследование

CreateView наследует функциональность от множества миксинов и базовых классов, каждый из которых добавляет свои методы и атрибуты. Это позволяет CreateView быть гибким и настраиваемым для различных сценариев создания объектов через веб-формы.

- [[i-d. Класс DetailView (+ SingleObjectTemplateResponseMixin, SingleObjectMixin, BaseDetailView)]]:
    - Этот миксин расширяет функциональность `TemplateResponseMixin` для отображения одиночного объекта.
    - Он предоставляет метод `get_template_names()`, который возвращает список имен шаблонов для рендеринга страницы.
    - Наследуется от [[i-b. Классы TemplateView, ContextMixin, TemplateResponseMixin]]:
        - Этот миксин предоставляет методы для рендеринга шаблонов и работы с контекстом.
        - Например, метод `render_to_response()` отвечает за формирование и возврат HTTP-ответа с отрендеренным шаблоном.
- `BaseCreateView`:
    - Этот базовый класс предоставляет основную функциональность для создания объектов.
    - Он наследуется от `ModelFormMixin` и `ProcessFormView`.
        - `ModelFormMixin`:
            - Он добавляет методы `get_form_class()`, `get_form_kwargs()`, `get_success_url()` и другие.
                - Этот миксин расширяет `FormMixin` и `SingleObjectMixin`.
                - [[i-e. Класс FormView (+ FormMixin, ProcessFormView, BaseFormView)]]:
                    - Этот миксин предоставляет дополнительные методы и атрибуты для работы с формами, такие как `get_form()`, `form_valid()`, `form_invalid()` и другие.
                    - Он также наследуется от [[i-b. Классы TemplateView, ContextMixin, TemplateResponseMixin]], который предоставляет методы для работы с контекстом шаблона.
                - [[i-d. Класс DetailView (+ SingleObjectTemplateResponseMixin, SingleObjectMixin, BaseDetailView)]]:
                    - Этот миксин предоставляет методы и атрибуты для работы с одиночным объектом модели.
                    - Он добавляет методы `get_object()`, `get_queryset()`, `get_context_object_name()` и другие.
        - [[i-e. Класс FormView (+ FormMixin, ProcessFormView, BaseFormView)]]:
            - Этот класс определяет методы `get()` и `post()` для обработки HTTP-запросов GET и POST.
            - Он также предоставляет метод `put()` для обработки PUT-запросов.
            - Наследуется от [[i-a. Базовый класс представлений - View]]:
                - Это базовый класс для всех классов-представлений в Django.
                - Он предоставляет основные методы, такие как `dispatch()`, `http_method_not_allowed()`, `options()` и другие.

### ModelFormMixin

`**ModelFormMixin**` - это миксин, который расширяет функциональность `FormMixin` и `SingleObjectMixin` для работы с формами, связанными с моделями Django.

**Основные аспекты его влияния:**

1. **Автоматическое создание класса формы на основе модели**:
    - `ModelFormMixin` предоставляет метод `get_form_class()`, который автоматически генерирует класс формы на основе указанной модели и списка полей (`fields`).
    - Это избавляет от необходимости вручную определять класс формы, если он напрямую связан с моделью.
2. **Предоставление экземпляра модели в форму**:
    - Метод `get_form_kwargs()` в `ModelFormMixin` добавляет экземпляр модели (`instance`) в аргументы, передаваемые в форму.
    - Это позволяет форме работать с существующим объектом модели при редактировании или обновлении данных.
3. **Обработка успешного сохранения формы**:
    - `ModelFormMixin` переопределяет метод `form_valid()`, который вызывается при успешной валидации формы.
    - В этом методе происходит сохранение связанного объекта модели с помощью `form.save()`.
4. **Определение URL перенаправления после успешной обработки формы**:
    - Миксин предоставляет метод `get_success_url()`, который возвращает URL для перенаправления после успешной обработки формы.
    - По умолчанию он использует `success_url` или `get_absolute_url()` модели, но может быть переопределен для получения пользовательского URL.
5. **Интеграция с SingleObjectMixin**:
    - `ModelFormMixin` наследуется от `SingleObjectMixin`, что позволяет работать с единичным объектом модели.
    - Это дает возможность использовать атрибуты и методы, предоставляемые `SingleObjectMixin`, такие как `get_object()` и `get_queryset()`.

Рассмотрим `ModelFormMixin` подробнее.

**Атрибуты:**

- `model` - модель Django, связанная с формой.
- `fields` - список полей модели, которые должны быть включены в форму. Если `None`, то все поля модели будут использованы.
- `success_url` - URL, на который будет выполнено перенаправление после успешной обработки формы. Может содержать подстановочные параметры, которые будут заменены значениями из объекта модели.

**Методы:**

- `get_form_class()` - возвращает класс формы, который будет использоваться для создания экземпляра формы. Если `form_class` указан, то используется он. В противном случае, класс формы генерируется автоматически на основе модели и списка полей (`fields`). 
    
    Пример переопределения `get_form_class()`:
    
    В этом примере, если текущий пользователь является суперпользователем, то используется специальный класс формы `AdminBookForm`. В противном случае, используется базовый класс формы `BookForm`.
    
    ```Python
    from django.views.generic import UpdateView
    from .models import Book
    from .forms import BookForm
    
    class BookUpdateView(UpdateView):
        model = Book
        form_class = BookForm
    
        def get_form_class(self):
            if self.request.user.is_superuser:
                return AdminBookForm
            return super().get_form_class()
    ```
    
- `get_form_kwargs()` - возвращает словарь аргументов для создания экземпляра формы. Добавляет `instance` в словарь, если объект модели доступен. 
    
    Пример переопределения `get_form_kwargs()`:
    
    В этом примере, в словарь аргументов добавляется начальное значение для поля `author`, которое устанавливается равным текущему пользователю.
    
    ```Python
    from django.views.generic import CreateView
    from .models import Book
    
    class BookCreateView(CreateView):
        model = Book
        fields = ['title', 'author', 'description']
    
        def get_form_kwargs(self):
            kwargs = super().get_form_kwargs()
            kwargs['initial'] = {'author': self.request.user}
            return kwargs
    ```
    
- `get_success_url()` - возвращает URL для перенаправления после успешной обработки формы. Если `success_url` указан, то он используется с подстановкой значений из объекта модели. В противном случае, вызывается `get_absolute_url()` модели. 
    
    Пример переопределения `get_success_url()`:
    
    В этом примере, после успешного создания книги, пользователь перенаправляется на страницу деталей этой книги, используя URL-имя `book_detail` и первичный ключ созданной книги.
    
    ```Python
    from django.views.generic import CreateView
    from django.urls import reverse_lazy
    from .models import Book
    
    class BookCreateView(CreateView):
        model = Book
        fields = ['title', 'author', 'description']
    
        def get_success_url(self):
            return reverse_lazy('book_detail', kwargs={'pk': self.object.pk})
    ```
    
- `form_valid(form)` - вызывается, если форма прошла валидацию. Сохраняет объект модели, связанный с формой, используя `form.save()`, и возвращает результат вызова базового метода. 
    
    Пример переопределения `form_valid()`:
    
    В этом примере, перед сохранением формы, в объект модели добавляется информация о пользователе, который выполнил обновление.
    
    ```Python
    from django.views.generic import UpdateView
    from .models import Book
    
    class BookUpdateView(UpdateView):
        model = Book
        fields = ['title', 'description']
    
        def form_valid(self, form):
    # Дополнительная логика перед сохранением формы
            form.instance.updated_by = self.request.user
            return super().form_valid(form)
    ```
    

Таким образом, `ModelFormMixin` предоставляет удобный способ работы с формами, связанными с моделями, в классах-представлениях Django. Он упрощает процесс создания и обработки форм, автоматизирует генерацию класса формы на основе модели и обеспечивает интеграцию с `SingleObjectMixin` для работы с единичными объектами модели.

### BaseCreateView

`**BaseCreateView**` - это базовый класс для создания представлений, которые позволяют создавать новые экземпляры моделей через формы. Он наследуется от `ModelFormMixin` и `ProcessFormView`.

**Основные аспекты его функциональности:**

1. **Обработка GET-запросов**:
    - Метод `get()` в `BaseCreateView` устанавливает `self.object` в `None`, чтобы указать, что новый объект еще не создан.
    - Затем он вызывает метод `get()` родительского класса (`ProcessFormView`) для дальнейшей обработки запроса.
2. **Обработка POST-запросов**:
    - Метод `post()` также устанавливает `self.object` в `None`, чтобы указать, что новый объект еще не создан.
    - Затем он вызывает метод `post()` родительского класса (`ProcessFormView`) для обработки отправленных данных формы и создания нового объекта.
3. **Использование ModelFormMixin**:
    - `BaseCreateView` наследуется от `ModelFormMixin`, который предоставляет функциональность для работы с формами, связанными с моделями.
    - `ModelFormMixin` автоматически генерирует форму на основе указанной модели и полей, а также обрабатывает сохранение объекта при успешной валидации формы.
4. **Требование предоставления миксина отображения (response mixin)**:
    - `BaseCreateView` не предоставляет реализацию для отображения ответа, поэтому для использования этого базового класса необходимо наследоваться от него и предоставить соответствующий миксин отображения, такой как `TemplateResponseMixin`.

Рассмотрим подробнее методы `BaseCreateView`:

- `get(request, *args, **kwargs)`:
    
    Пример использования:
    
    ```Python
    from django.views.generic import CreateView
    from .models import Book
    
    class BookCreateView(CreateView):
        model = Book
        fields = ['title', 'author', 'description']
        template_name = 'book_form.html'
    
        def get(self, request, *args, **kwargs):
    # Дополнительная логика перед обработкой GET-запроса
    # Например, проверка прав доступа
            if not request.user.is_authenticated:
                return redirect('login')
            return super().get(request, *args, **kwargs)
    ```
    
    В этом примере, представление `BookCreateView` наследуется от `CreateView`, которое, в свою очередь, наследуется от `BaseCreateView`. Перед обработкой GET-запроса выполняется проверка аутентификации пользователя, и если пользователь не аутентифицирован, происходит перенаправление на страницу входа.
    
- `post(request, *args, **kwargs)`:
    
    Пример использования:
    
    ```Python
    from django.views.generic import CreateView
    from .models import Book
    
    class BookCreateView(CreateView):
        model = Book
        fields = ['title', 'author', 'description']
        template_name = 'book_form.html'
    
        def post(self, request, *args, **kwargs):
    # Дополнительная логика перед обработкой POST-запроса
    # Например, модификация данных формы перед сохранением
            form = self.get_form()
            if form.is_valid():
                form.instance.created_by = request.user
            return super().post(request, *args, **kwargs)
    ```
    
    В этом примере, перед обработкой POST-запроса и сохранением нового объекта книги, в форму добавляется информация о пользователе, который создал книгу (`created_by`).
    

Таким образом, `BaseCreateView` является базовым классом для создания представлений, которые позволяют создавать новые объекты через формы. Он предоставляет основную функциональность для обработки GET и POST запросов, а также использует `ModelFormMixin` для работы с формами, связанными с моделями. Однако, для полноценного использования `BaseCreateView` необходимо наследоваться от него и предоставить соответствующий миксин отображения.

### Краткое summery

1. `model` - указывает модель Django, которая будет использоваться для создания нового объекта. Этот атрибут является обязательным, если не указан атрибут `form_class`.
2. `fields` - список полей модели, которые должны быть включены в форму. Если `form_class` не указан, Django автоматически создаст форму на основе указанных полей. Если `fields` равен `None`, то будут использованы все поля модели.
3. `form_class` - указывает пользовательский класс формы, который будет использоваться вместо автоматически сгенерированной формы на основе модели. Нельзя использовать одновременно `form_class` и `model` / `fields`.
4. `template_name` - имя шаблона, который будет использоваться для отображения формы. Если не указано, Django будет использовать шаблон по умолчанию на основе имени модели и суффикса `_form.html`.
5. `success_url` - URL, на который будет выполнено перенаправление после успешного создания объекта. Может быть указан явно или определен путем переопределения метода `get_success_url()`.

<div class="page-break" style="page-break-before: always;"></div>
