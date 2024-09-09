Атрибуты пагинации

Методы пагинации

Отображение ссылок на страницы

Нюансы и особенности

Бонус)

Django предоставляет удобный способ реализации пагинации в generic-представлениях, в частности в классе `ListView`. Пагинация позволяет разделить список объектов на страницы, чтобы пользователь мог удобно просматривать большое количество данных.

### Атрибуты пагинации

Для включения пагинации в `ListView` достаточно указать атрибут `paginate_by`:

```Python
class ContactListView(ListView):
    paginate_by = 20
    model = Contact
```

Этот атрибут определяет количество объектов, отображаемых на одной странице. Django автоматически добавит в контекст шаблона переменные `paginator` и `page_obj`, которые будут использоваться для отображения ссылок на страницы.

Помимо `paginate_by` есть и другие атрибуты, связанные с пагинацией:

- `paginate_orphans` - максимальное количество "сирот" (объектов на последней странице), которые будут добавлены к предыдущей странице. По умолчанию равно 0.
- `paginator_class` - класс пагинатора. По умолчанию используется `django.core.paginator.Paginator`.
- `page_kwarg` - имя параметра в URL, содержащего номер страницы. По умолчанию равно "page".

### Методы пагинации

Класс `ListView` содержит несколько методов, связанных с пагинацией, которые можно переопределить для более тонкой настройки:

- `paginate_queryset(queryset, page_size)` - разбивает queryset на страницы с учетом размера страницы. Возвращает кортеж (paginator, page, queryset, is_paginated).
- `get_paginate_by(queryset)` - возвращает количество объектов на странице или `None`, если пагинация не используется.
- `get_paginator(queryset, per_page, **kwargs)` - создает и возвращает экземпляр пагинатора.
- `get_paginate_orphans()` - возвращает количество "сирот" для пагинации.

Пример переопределения метода `get_paginate_by()` для динамического определения количества объектов на странице:

```Python
class ContactListView(ListView):
    model = Contact

    def get_paginate_by(self, queryset):
        if self.request.user.is_authenticated:
            return 50
            # Для авторизованных пользователей показывать по 50 объектов
        else:
            return 10
            # Для остальных - по 10 объектов
```

### Отображение ссылок на страницы

Для отображения ссылок на страницы в шаблоне можно использовать переменные `paginator` и `page_obj`. Пример шаблона с ссылками на предыдущую, следующую, первую и последнюю страницы:

```HTML
{% for contact in page_obj %}
    {{ contact.full_name|upper }}<br>
    ...
{% endfor %}

<div class="pagination">
    <span class="step-links">
        {% if page_obj.has_previous %}
            <a href="?page=1">&laquo; first</a>
            <a href="?page={{ page_obj.previous_page_number }}">previous</a>
        {% endif %}

        <span class="current">
            Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
        </span>

        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">next</a>
            <a href="?page={{ page_obj.paginator.num_pages }}">last &raquo;</a>
        {% endif %}
    </span>
</div>
```

### Нюансы и особенности

- Если `paginate_by` не указан, пагинация не будет применяться и все объекты будут выводиться на одной странице.
- Если запрошенный номер страницы выходит за границы допустимых значений, будет возбуждено исключение `InvalidPage`, которое по умолчанию обрабатывается как 404 ошибка.
- Если `allow_empty` равен `True` (по умолчанию), то пустой список объектов будет отображаться. Если `False`, то будет возбуждена ошибка 404.
- Контекстная переменная `is_paginated` указывает, используется ли пагинация на текущей странице.
- Переменная `page_obj` содержит объекты только текущей страницы, в то время как `paginator.object_list` содержит все объекты из queryset.

В целом, встроенная пагинация в Django ListView достаточно гибкая и позволяет быстро добавить постраничное отображение списка объектов. При необходимости можно переопределить методы пагинации и тонко настроить ее поведение под свои нужды.

### Бонус)

По умолчанию в классе лист пагинация происходит посредством метода `page()`, а, как мы помним, данный метод не выполняет обработку исключений, а просто выбрасывает их (если на вход получена строка или номер, выходящий за диапазон страниц), сама обработка остаётся на ответственности разработчика. Но мы можем изменить данное поведение, переопределив в классе представлении метод `paginate_queryset` следующим образом, чтобы использовать метод `get_page()` и не заниматься обработкой исключений самостоятельно:

```Python
class IndexView(DataMixin, ListView):
    queryset = Post.published.all()
    page_title = 'Главная страница'
    template_name = 'basefunc/index.html'
    context_object_name = 'posts'
    paginate_by = 5

    # модифицированный метод
    def paginate_queryset(self, queryset, page_size):
        paginator = self.get_paginator(
            queryset,
            page_size,
            orphans=self.get_paginate_orphans(),
            allow_empty_first_page=self.get_allow_empty(),
        )
        page_kwarg = self.page_kwarg
        page_number = self.kwargs.get(page_kwarg) or self.request.GET.get(page_kwarg) or 1
        page = paginator.get_page(page_number)
        return paginator, page, page.object_list, page.has_other_pages()

    # изначальный вид метода
    # def paginate_queryset(self, queryset, page_size):
    #     """Paginate the queryset, if needed."""
    #     paginator = self.get_paginator(
    #         queryset,
    #         page_size,
    #         orphans=self.get_paginate_orphans(),
    #         allow_empty_first_page=self.get_allow_empty(),
    #     )
    #     page_kwarg = self.page_kwarg
    #     page = self.kwargs.get(page_kwarg) or self.request.GET.get(page_kwarg) or 1
    #     try:
    #         page_number = int(page)
    #     except ValueError:
    #         if page == "last":
    #             page_number = paginator.num_pages
    #         else:
    #             raise Http404(
    #                 _("Page is not “last”, nor can it be converted to an int.")
    #             )
    #     try:
    #         page = paginator.page(page_number)
    #         return (paginator, page, page.object_list, page.has_other_pages())
    #     except InvalidPage as e:
    #         raise Http404(
    #             _("Invalid page (%(page_number)s): %(message)s")
    #             % {"page_number": page_number, "message": str(e)}
    #         )
```

Важно помнить, что в таком случае мы не будем иметь доступ к последней странице, используя строку `‘last’`, но этим можно и пренебречь, если нет такой необходимости)

<div class="page-break" style="page-break-before: always;"></div>
