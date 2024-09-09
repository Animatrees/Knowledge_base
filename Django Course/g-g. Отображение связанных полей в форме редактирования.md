Основное описание

Проблема m2m связей

Решение проблемы похожих запросов для m2m полей

Решение проблемы дублирующих запросов для m2m связей

Бонус)

### Основное описание

Допустим мы хотим отобразить в форме редактирования категорий (родительская модель) всех женщин (связанная с нею дочерняя модель) из этой категории. Что нам для этого нужно сделать?

У администратора модели существует для этого атрибут - `inlines`

Сам этот атрибут используется только для того, чтобы указать названия классов вставок моделей, основная “соль” заключается в создании и настройке этого самого класса вставки)

Простой пример:

```Python
class PostInline(admin.TabularInline):
    model = Post
    extra = 0

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug')
    list_display_links = ('title',)
    ordering = ('title',)
    inlines = (PostInline,)
```

Используя данный код, мы получим следующее отображение женщин на странице редактирования категории.

![[Untitled 9 1.png|Untitled 9 1.png]]

Всего существует два способа отображения дочерних моделей в родительской - **TabularInline** класс (представлен выше) и **StackedInline** класс (см. картинку ниже), вся разница между ними заключается в шаблоне, который будет отрендерен для отображения данных. Оба эти класса наследуются от **InlineModelAdmin** и имеют общие атрибуты для настройки.

![[Untitled 1 6.png|Untitled 1 6.png]]

Эти классы можно настраивать практически также, как администратора модели, в них доступны, например, следующие атрибуты:

- [`fields`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fields)
- [`exclude`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.exclude)
- [`filter_horizontal`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.filter_horizontal)
- [`filter_vertical`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.filter_vertical)
- [`ordering`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.ordering)
- [`prepopulated_fields`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.prepopulated_fields)
- [`readonly_fields`](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.readonly_fields)
- и т.д.

Основная разница с администратором модели заключается в том, что для классов вставки нужно прописать атрибут `model` - модель, которую мы будем вставлять (обязательный атрибут).

Также в некоторых случаях может потребоваться указать имя внешнего ключа модели - `fk_name`. В большинстве случаев это происходит автоматически, но если существует несколько внешних ключей к одной и той же родительской модели, имя `fk_name` должно быть указано явно.

Атрибут `extra` используется для того, чтобы указать какое кол-во пустых форм для добавления новых объектов будет отображено в форме редактирования. По умолчанию - 3.

И ещё один полезный атрибут - `can_delete`. Отвечает за отображение чекбокса для удаления связанных объектов прямо из формы редактирования. По умолчанию - True.

Более подробно ознакомиться с классом InlineModelAdmin можно в [документации](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/#inlinemodeladmin-objects).

### Проблема m2m связей

Используя классы вставки, можно столкнуться со следующей проблемой - мы получаем большое кол-во дублирующих (или похожих) запросов к базе данных, так как наша модель **Post** “тащит” вместе с собой модель T**ags**, связанную с каждой женщиной отношением многие ко многим.

![[Untitled 2 5.png|Untitled 2 5.png]]

![[Untitled 3 2.png|Untitled 3 2.png]]

![[Untitled 4 2.png|Untitled 4 2.png]]

- Первая проблема возникает из-за того, что для каждой женщины идёт обращение в базу данных, чтобы получить значения категорий с которыми связана именно она.
- Вторая проблема возникает из-за того, чтобы отрисовать виджет выбора категории для каждой женщины мы каждый раз запрашивает данные из таблицы с тегами.

### Решение проблемы похожих запросов для m2m полей

Чтобы не дублировать запросы получения категорий, мы можем получить теги для всех женщин общим запросом, в момент формирования queryset-а с самими женщинами. Для этого кастомизируем метод get_queryset внутри нашего класса вставки с помощью метода - `prefetch_related`:

```Python
    def get_queryset(self, request):
        queryset = super().get_queryset(request)
        queryset = queryset.prefetch_related('tags')
        return queryset
```

Похожие запросы исчезают)

![[Untitled 5 2.png|Untitled 5 2.png]]

Осталось разобраться с дублируюшими запросами -_-

### Решение проблемы дублирующих запросов для m2m связей

Есть три способа решить данную проблему:

1. **Не отображаем** теги на странице редактирования **совсем**. В данном случае мы не делаем запросы к таблице тегов для формирования виджета для каждой женщины.
    
    *Если вам совсем не нужны теги в форме редактирования, то также не обязательно использовать кастомный `get_queryset`, так как обращения к таблице с тегами в данном случае не происходит.
    
    ```Python
    class PostInline(admin.TabularInline):
        model = Post
        extra = 0
        exclude = ('tags',)
        ...
    ```
    
2. Отображаем теги в виде строки с перечислением их id. Не самый интуитивный способ отображения информации, но для каких-то случаев может подойти)
    
    Здесь нам также не нужно рендерить виджет для выбора тега, поэтому дублирующих запросов не происходит. Однако, мы всё ещё запрашиваем информацию о тегах для каждой женщины, поэтому этот способ используем в связке с `get_queryset`.
    
    ```Python
    class PostInline(admin.TabularInline):
        model = Post
        extra = 0
        raw_id_fields = ('tags',)
        ...
    ```
    
    ![[Untitled 6 2.png|Untitled 6 2.png]]
    
3. И, наконец, самый заморочный, но самый полноценный способ отображения тегов - это их кеширование во время формирования формы. Здесь я не буду детально разбирать способ реализации, так как сама нашла его на [stackoverflow](https://stackoverflow.com/questions/40665770/django-inline-for-manytomany-generate-duplicate-queries)))
    
    Добавляем парочку классов для кеширования формы, класс формы, которую будет использовать класс вставки и атрибуты, чтобы применить эти изменения.
    
    ```Python
    class CachingModelChoicesFormSet(forms.BaseInlineFormSet):
        """
        Used to avoid duplicate DB queries by caching choices and passing them all the forms.
        To be used in conjunction with `CachingModelChoicesForm`.
        """
    
        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)
            sample_form = self._construct_form(0)
            self.cached_choices = {}
            try:
                model_choice_fields = sample_form.model_choice_fields
            except AttributeError:
                pass
            else:
                for field_name in model_choice_fields:
                    if field_name in sample_form.fields and not isinstance(
                            sample_form.fields[field_name].widget, forms.HiddenInput):
                        self.cached_choices[field_name] = [c for c in sample_form.fields[field_name].choices]
    
        def get_form_kwargs(self, index):
            kwargs = super().get_form_kwargs(index)
            kwargs['cached_choices'] = self.cached_choices
            return kwargs
    
    
    class CachingModelChoicesForm(forms.ModelForm):
        """
        Gets cached choices from `CachingModelChoicesFormSet` and uses them in model choice fields in order to reduce
        number of DB queries when used in admin inlines.
        """
    
        @property
        def model_choice_fields(self):
            return [fn for fn, f in self.fields.items()
                    if isinstance(f, (forms.ModelChoiceField, forms.ModelMultipleChoiceField,))]
    
        def __init__(self, *args, **kwargs):
            cached_choices = kwargs.pop('cached_choices', {})
            super().__init__(*args, **kwargs)
            for field_name, choices in cached_choices.items():
                if choices is not None and field_name in self.fields:
                    self.fields[field_name].choices = choices
    
    
    class PostInlineForm(CachingModelChoicesForm):
        class Meta:
            model = Post
            exclude = ()
    
    
    class PostInline(admin.TabularInline):
        model = Post
        extra = 0
    
        form = PostInlineForm
        formset = CachingModelChoicesFormSet
    
        def get_queryset(self, request):
            queryset = super().get_queryset(request)
            queryset = queryset.prefetch_related('tags')
            return queryset
    ```
    
    Классы кэширования форм - универсальные, чтобы использовать их в своих проектах, нужно добавить в своём коде то, что я выделила жирным шрифтом.
    
    Вуаля - все дублирующие запросы удалены!)
    
    ![[Untitled 7 2.png|Untitled 7 2.png]]
    

### Бонус)

Виджет для быстрого добавления/удаления женщин в категорию.

```Python
class PostAdminForm(forms.ModelForm):
    # Определение поля формы для выбора нескольких женщин
    women = forms.ModelMultipleChoiceField(queryset=Post.objects.all(),
                                           # Использование виджета с автозаполнением для удобства выбора
                                           widget=admin.widgets.FilteredSelectMultiple("women", is_stacked=False))

    class Meta:
        model = Category  # Указание модели, к которой привязана форма
        fields = '__all__'  # Включение всех полей модели в форму

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if self.instance.pk:
            # Если у объекта категории уже есть идентификатор (т.е. он существует в базе данных),
            # установка начальных значений для поля "women" равными текущим женщинам этой категории
            self.fields['women'].initial = self.instance.women.all()


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'number_of_women')
    list_display_links = ('title',)
    ordering = ('title',)
    # Подключение формы к административному интерфейсу
    form = PostAdminForm
```

![[Untitled 8 2.png|Untitled 8 2.png]]
<div class="page-break" style="page-break-before: always;"></div>
