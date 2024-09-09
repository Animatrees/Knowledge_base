self.client

Response

asserts

### self.client

`self.client` - каждый тест имеет доступ к экземпляру тестового клиента Django. Доступ к этому клиенту можно получить по имени self.client. Этот клиент создается заново для каждого теста, поэтому вам не нужно беспокоиться о том, что состояние (например, куки) будет переноситься из одного теста в другой.

**Основные возможности и методы self.client:**

1. `self.client.get(path, data=None, follow=False, **extra)` - имитирует HTTP GET запрос на указанный path (URL). Можно передавать данные запроса через data, следовать перенаправлениям с помощью follow и дополнительные параметры через **extra.
2. `self.client.post(path, data=None, content_type=MULTIPART_CONTENT, follow=False, **extra)` - имитирует HTTP POST запрос на указанный path. Позволяет передавать данные формы через data, устанавливать тип содержимого через content_type, следовать перенаправлениям и дополнительные параметры.
3. `self.client.put(path, data=None, content_type=MULTIPART_CONTENT, follow=False, **extra)` - имитирует HTTP PUT запрос.
4. `self.client.head(path, data=None, follow=False, **extra)` - имитирует HTTP HEAD запрос.
5. `self.client.delete(path, data=None, follow=False, **extra)` - имитирует HTTP DELETE запрос.
6. `self.client.options(path, data=None, follow=False, **extra)` - имитирует HTTP OPTIONS запрос.

> [!important]  
> Методы возвращают объект Response, содержащий атрибуты для проверки, такие как status_code, content, context_data и другие, что позволяет писать ассерты для тестирования ответов приложения на соответствующие запросы.  

**А также:**

1. `self.client.login(**credentials)` - выполняет вход в систему с указанными учетными данными.
2. `self.client.logout()` - выполняет выход из системы.
3. `self.client.force_login(user, backend=None)` - принудительно выполняет вход от имени указанного пользователя user.

### Response

Методы get() и post() возвращают объект Response. Этот объект Response - не то же самое, что объект HttpResponse, возвращаемый представлениями Django; объект тестового ответа содержит некоторые дополнительные данные, полезные для проверки тестовым кодом.

**Основные атрибуты:**

1. `client` - экземпляр тестового клиента, который был использован для выполнения запроса.
2. `content` - тело ответа в виде байтовой строки. Это окончательный контент страницы, отрендеренный представлением, или любое сообщение об ошибке.
3. `context` - экземпляр Context, использовавшийся для рендеринга шаблона, создавшего контент ответа. Если использовалось несколько шаблонов, context будет списком экземпляров Context в порядке их рендеринга.
4. `exc_info` - кортеж из трёх значений, предоставляющий информацию об необработанном исключении, если таковое произошло во время выполнения представления (type, value, traceback).
5. `json(**kwargs)` - тело ответа, распарсенное как JSON. Дополнительные ключевые аргументы передаются в json.loads().
6. `request` - данные запроса, инициировавшего ответ.
7. `wsgi_request` - экземпляр WSGIRequest, сгенерированный тестовым обработчиком для получения ответа.
8. `status_code` - HTTP статус ответа в виде целого числа.
9. `templates` - список экземпляров Template, использовавшихся для рендеринга окончательного контента в порядке их рендеринга.
10. `resolver_match` - экземпляр ResolverMatch для ответа, позволяющий проверить представление, обслужившее запрос.

Также можно получить доступ к заголовкам ответа через `response.headers`, например response.headers['Content-Type'].

### asserts

В модуле `[unittest](https://docs.python.org/3/library/unittest.html#assert-methods)` в Python доступно множество методов группы assert, которые можно использовать для проверки утверждений в тестах. Некоторые наиболее часто используемые методы:

- `assertEqual(a, b)` - проверяет равенство a и b.
- `assertNotEqual(a, b)` - проверяет, что a не равно b.
- `assertTrue(x)` - проверяет, что x истинно (True).
- `assertFalse(x)` - проверяет, что x ложно (False).
- `assertIs(a, b)` - проверяет, что a и b ссылаются на один и тот же объект.
- `assertIsNot(a, b)` - проверяет, что a и b ссылаются на разные объекты.
- `assertIsNone(x)` - проверяет, что x равно None.
- `assertIsNotNone(x)` - проверяет, что x не равно None.
- `assertIn(a, b)` - проверяет, что a содержится в b (коллекции).
- `assertNotIn(a, b)` - проверяет, что a не содержится в b.
- `assertIsInstance(a, type)` - проверяет, что a является экземпляром указанного типа.
- `assertNotIsInstance(a, type)` - проверяет, что a не является экземпляром указанного типа.
- `assertRaises(exception, callable, *args, **kwargs)` - проверяет, что вызов callable с args и kwargs вызывает указанное исключение.
- `assertWarns(warning, callable, *args, **kwargs)` - проверяет, что вызов callable с args и kwargs вызывает указанное предупреждение.
- `assertAlmostEqual(a, b)` - проверяет, что a приблизительно равно b с точностью по умолчанию.
- `assertNotAlmostEqual(a, b)` - проверяет, что a не приблизительно равно b с точностью по умолчанию.
- `assertGreater(a, b)` - проверяет, что a строго больше b.
- `assertGreaterEqual(a, b)` - проверяет, что a больше или равно b.
- `assertLess(a, b)` - проверяет, что a строго меньше b.
- `assertLessEqual(a, b)` - проверяет, что a меньше или равно b.
- `assertRegex(text, regex)` - проверяет, что строка text соответствует регулярному выражению regex.
- `assertNotRegex(text, regex)` - проверяет, что строка text не соответствует регулярному выражению regex.
- `assertCountEqual(list1, list2)` - проверяет, что два списка содержат одинаковые элементы, независимо от их порядка.

Помимо стандартных assert методов из модуля unittest, Django также предоставляет дополнительные assert методы, специфичные для тестирования веб-приложений Django.

Эти методы определены в модуле `django.test` и включают следующие:

- `assertContains(response, text, count=None, status_code=200, msg_prefix='', html=False)` - проверяет, что ответ содержит указанный текст. Можно указать количество вхождений текста с помощью count, а также ожидаемый код состояния и префикс сообщения об ошибке.
- `assertNotContains(response, text, status_code=200, msg_prefix='', html=False)` - противоположность assertContains, проверяет, что ответ не содержит указанный текст.
- `assertFormError(response, form, field, errors, msg_prefix='')` - проверяет, что в указанной форме присутствуют ошибки валидации для указанного поля.
- `assertFormSetError(formset, form_index, field, errors, msg_prefix='')` - проверяет ошибки валидации для указанного поля в указанной форме внутри формсета.
- `assertRedirects(response, expected_url, status_code=302, target_status_code=200, msg_prefix='', fetch_redirect_response=False)` - проверяет, что ответ перенаправляет на указанный URL.
- `assertTemplateUsed(response, template_name, msg_prefix='', ignore_unused_templates=False)` - проверяет, что для рендеринга ответа использовался указанный шаблон.
- `assertTemplateNotUsed(response, template_name, msg_prefix='')` - проверяет, что для рендеринга ответа не использовался указанный шаблон.
- `assertQuerySetEqual(qs, values, transform=None, ordered=True, msg=None)` - проверяет, что QuerySet содержит ожидаемые значения.

Больше методов в [документации](https://docs.djangoproject.com/en/5.0/topics/testing/tools/#assertions).

<div class="page-break" style="page-break-before: always;"></div>
