Автоматизируем фикстуры

Проверяем содержимое первой страницы

Проверяем вторую страницу

Иииили пишем нормальный код и проверяем сразу все страницы))

Проверяем содержимое поста

Атрибуты и методы для работы с БД

Фикстуры vs транзакции

### Автоматизируем фикстуры

По умолчанию при запуске тестов база данных создаётся пустая, если же нам для тестов нужно наполнить её данными, создаём [[l-a. Django Fixtures]] и указываем их в атрибуте `fixtures` тестирующего класса. Для того, чтобы в ручном режиме не создавать фикстуры каждый раз, можно добавить команду для формирования файла с фикстурами для всей БД перед запуском тестов.

1. Создаём файл с командой по следующему пути - `management/commands/save_fixture.py`
    
    Тут важно, чтобы эти папки являлись модулями (содержат файлы `__init__.py`), и находились в приложении, которое подключено у нас в `installed_apps`, прикрепляю скрин моей структуры.
    
    ![[Untitled 21.png|Untitled 21.png]]
    
    Содержимое:
    
    ```Python
    import os
    from django.core.management.base import BaseCommand
    from django.core.management import call_command
    
    
    class Command(BaseCommand):
        help = 'Save the current state of the database to a fixture.'
    
        def add_arguments(self, parser):
            parser.add_argument('fixture_name', type=str, help='The name of the fixture file to save.')
    
        def handle(self, *args, **kwargs):
            fixture_name = kwargs['fixture_name']
            fixture_path = os.path.join('fixtures', f'{fixture_name}.json')
            os.makedirs(os.path.dirname(fixture_path), exist_ok=True)
            call_command('dumpdata', '--output', fixture_path)
            self.stdout.write(self.style.SUCCESS(f'Database saved to {fixture_path}'))
    ```
    
    Итого у нас есть работающая команда для создания фикстур, убедиться в работоспособности можно вызовом команды:
    
    `python manage.py save_fixture testdata`
    
2. Модифицируем тестовый раннер, у меня он добавлен по пути - `testpet/testpet/test_runner.py`
    
    ![[Untitled 1 11.png|Untitled 1 11.png]]
    
    Внутри прописываем дополнительную логику, по которой будет создаваться и уничтожаться тестовое окружение:
    
    ```Python
    import os
    from django.core.management import call_command
    from django.test.runner import DiscoverRunner
    
    
    class FixtureDiscoverRunner(DiscoverRunner):
        def setup_test_environment(self, **kwargs):
            super().setup_test_environment(**kwargs)
            fixture_name = 'testdata'
            fixture_path = os.path.join('fixtures', f'{fixture_name}.json')
            if not os.path.exists(fixture_path):
                call_command('save_fixture', fixture_name)
            call_command('loaddata', fixture_path)  # Загрузите фикстуру с полным путем
    
        def teardown_test_environment(self, **kwargs):
            super().teardown_test_environment(**kwargs)
            fixture_name = 'testdata'
            fixture_path = os.path.join('fixtures', f'{fixture_name}.json')
            if os.path.exists(fixture_path):
                os.remove(fixture_path)
    ```
    
3. В `settings.py` прописываем путь до нашего кастомизированного раннера:
    
    ```Python
    TEST_RUNNER = 'testpet.test_runner.FixtureDiscoverRunner'
    ```
    
    В итоге получаем автоматизированное создание фикстур для всей БД)
    
    Далее будут описаны сами проверяющие методы из лекции, но модифицированные под мой код.
    

### Проверяем содержимое первой страницы

```Python
from django.core.cache import cache
from django.test import TestCase
from django.urls import reverse
from basefunc.models import Post

class DataTestCase(TestCase):
    fixtures = ['fixtures/testdata.json']

    def setUp(self):
        cache.clear()

    def test_data_mainpage(self):
        posts = Post.published.all()
        path = reverse('home')
        response = self.client.get(path)
        self.assertQuerySetEqual(response.context_data['posts'], posts[:3])
```

- Создаём тестовый класс `DataTestCase`, который наследуется от `TestCase`.
- Задаём `fixtures` для загрузки тестовых данных из `fixtures/testdata.json`.
- Определяем метод `setUp`, который очищает кэш перед каждым тестом. (этого не было в лекции, добавила после некоторых тестов с прошлой лекции))
- Определяем тестовый метод `test_data_mainpage`, который:
    - Извлекает все опубликованные посты из базы данных.
    - Создает URL для главной страницы с помощью `reverse('home')`.
    - Выполняет GET-запрос к главной странице и получает ответ.
    - Проверяет, что в контексте ответа `response.context_data['posts']` содержатся первые три опубликованных поста. Три, потому что в моей реализации именно такой `paginate_by`.

### Проверяем вторую страницу

```Python
    def test_paginate_mainpage(self):
        path = reverse('home')
        page = 2
        paginate_by = 3
        response = self.client.get(path + f'?page={page}')
        posts = Post.published.all()
        self.assertQuerySetEqual(response.context_data['posts'], posts[(page - 1) * paginate_by:page * paginate_by])
```

Данный метод:

- Создает URL для главной страницы с помощью `reverse('home')`.
- Устанавливает номер страницы (`page = 2`) и количество элементов на странице (`paginate_by = 3`).
- Выполняет GET-запрос к главной странице с параметром `page=2` и получает ответ.
- Извлекает все опубликованные посты из базы данных.
- Проверяет, что в контексте ответа `response.context_data['posts']` содержатся посты, соответствующие второй странице пагинации (посты с индекса 3 до 5).

### Иииили пишем нормальный код и проверяем сразу все страницы))

```Python
    from basefunc.utils import PaginateMixin
    
    def test_paginate_pages(self):
        path = reverse('home')
        posts = Post.published.all()
        paginate_by = PaginateMixin.paginate_by
        pages = math.ceil(len(posts) / paginate_by)
        for page in range(1, pages + 1):
            response = self.client.get(path + f'?page={page}')
            self.assertQuerySetEqual(response.context_data['posts'], posts[(page - 1) * paginate_by:page * paginate_by])
```

### Проверяем содержимое поста

```Python
    def test_content_post(self):
        post = Post.published.get(pk=1)
        path = reverse('post', args=[post.slug])
        response = self.client.get(path)
        self.assertEqual(post.content, response.context_data['post'].content)
```

Данный метод:

- Извлекает опубликованный пост с `pk=1`.
- Создает URL для страницы поста с использованием `slug` поста.
- Выполняет GET-запрос к странице поста.
- Проверяет, что содержимое поста в базе данных совпадает с содержимым поста на странице ответа.

- Ииииили… ну вы поняли, что можно написать метод для проверки всех постов)
    
    ```Python
        def test_check_posts_content(self):
            posts = Post.published.all()
            for p in posts:
                path = reverse('post', args=[p.slug])
                response = self.client.get(path)
                self.assertEqual(p.content, response.context_data['post'].content)
    ```
    

### Атрибуты и методы для работы с БД

1. `**databases**`:
    
    Устанавливает, какие базы данных будут использоваться в тестах - в виде set-а.. По умолчанию используется только база данных `default`.
    
    ```Python
    databases = {DEFAULT_DB_ALIAS}
    ```
    
2. `**reset_sequences**`:
    
    Если установить в `True`, сбрасывает последовательности автоинкремента в базе данных перед каждым тестом. По умолчанию False.
    
    ```Python
    reset_sequences = True
    ```
    
3. `**fixtures**`:
    
    Список фикстур, которые будут автоматически загружены в базу данных перед запуском тестов.
    
    ```Python
    fixtures = ['myapp/fixtures/testdata.json']
    ```
    
4. `**serialized_rollback**`:
    
    Если установить в `True`, сохраняет состояние базы данных в сериализованном виде перед каждым тестом и восстанавливает его после теста. По умолчанию False.
    
    ```Python
    serialized_rollback = True
    ```
    
5. `**assertQuerySetEqual**`:
    
    Метод для проверки того, что два набора запросов (querysets) содержат одинаковые данные.
    
    ```Python
    def test_queryset(self):
        self.assertQuerySetEqual(
            MyModel.objects.filter(name="test"),
            ["<MyModel: test>"],
            transform=repr,
        )
    ```
    
6. `**assertNumQueries**`:
    
    Метод для проверки того, что определенное количество запросов к базе данных выполнено за время выполнения кода.
    
    ```Python
    def test_num_queries(self):
        with self.assertNumQueries(2):
            list(MyModel.objects.all())
    ```
    

### Фикстуры vs транзакции

Меня довольно сильно смутила данная информация из документации:

[![](https://ucarecdn.com/95cf752a-91f8-4719-a6f7-62c7a0963dfb/)](https://ucarecdn.com/95cf752a-91f8-4719-a6f7-62c7a0963dfb/)

Выглядит как противоречие между собой (и с тем, что было сказано в лекции). Оказалось, что в коде этот момент реализован следующим образом:

- **TransactionTestCase**:
    - Фикстуры устанавливаются в методе _fixture_setup().
    - Фикстуры загружаются для каждого теста с помощью команды loaddata.
    - После каждого теста база данных очищается в методе _fixture_teardown().
- **TestCase:**
    - Наследует от TransactionTestCase, но использует транзакции для изоляции тестов.
    - Фикстуры загружаются один раз в методе setUpClass() для всего класса тестов.
    - Используется атомарный контекст для каждого теста, что позволяет откатывать изменения после каждого теста без повторной загрузки фикстур.

Еще немного покопавшись в коде, обнаружила, что БД создаётся только один раз, перед запуском всех тестов, за что отвечает метод `setup_databases`, который вызывается тестовым раннером (`DiscoverRunner`). А уничтожается она после отработки всех тестов.

**Итого**:

1. База данных создаётся **один раз** для всех тестов.

2. В случае использования класса TestCase, **фикстуры не загружаются перед каждым тестовым методом**, а загружаются один раз перед всеми тестами класса. Очистка же происходит за счёт транзакций.

<div class="page-break" style="page-break-before: always;"></div>
