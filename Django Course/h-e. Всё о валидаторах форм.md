Процесс валидации форм

Сообщения об ошибках

Встроенные валидаторы Django

Кастомный валидатор в виде функции

Кастомный валидатор в виде класса

Метод clean_field в классе формы

### **Процесс валидации форм**

Метод `is_valid()` является точкой входа в процесс валидации формы. Когда вы вызываете этот метод на экземпляре формы, тогда последовательно выполняются следующие шаги:

1. `to_python()`:
    - Это первый шаг в процессе валидации.
    - Метод преобразует входное значение в правильный Python-объект.
    - Если преобразование невозможно, метод должен поднять `ValidationError`.
    - Возвращаемое значение - это преобразованный объект.
    - Например, `FloatField` преобразует входные данные в Python float, либо поднимает `ValidationError`.
2. `validate()`:
    - Этот метод отвечает за валидацию, специфичную для данного поля.
    - Он принимает уже преобразованное в правильный тип значение и может поднять `ValidationError` в случае ошибки.
    - Метод не должен возвращать ничего и не должен изменять значение.
    - Его следует переопределять, чтобы реализовать логику валидации, которую нельзя или не хочется помещать в валидатор.
3. `run_validators()`:
    - Этот метод запускает все валидаторы, прикрепленные к полю, и объединяет все поднятые ошибки в единый `ValidationError`.
    - Не рекомендуется переопределять этот метод.
4. `clean()`:
    - Этот метод ответственен за последовательный вызов `to_python()`, `validate()` и `run_validators()`, а также за распространение поднятых ошибок.
    - Если любой из вызываемых методов поднимает `ValidationError`, валидация прекращается и ошибка поднимается дальше.
    - Этот метод возвращает очищенные данные, которые затем помещаются в словарь `cleaned_data` формы.
5. `clean_<fieldname>()`:
    - Этот метод вызывается на подклассе формы, где `<fieldname>` заменяется на имя атрибута поля формы.
    - Метод выполняет очистку и валидацию, специфичную для данного поля, независимо от типа поля.
    - В этом методе нет параметров, вместо этого необходимо получать значение поля из `self.cleaned_data`, помня, что это будет объект Python, а не изначальная строка, отправленная в форме.
    - Возвращаемое значение этого метода **заменяет существующее значение** в `cleaned_data`, поэтому оно должно быть либо значением поля из `cleaned_data` (даже если метод его не изменял), либо новым очищенным значением.

В целом, эти методы выполняются последовательно для каждого поля формы, а затем вызывается метод `clean()` самой формы, независимо от того, были ли ошибки в предыдущих методах.

### Сообщения об ошибках

Параметр `error_messages` поля формы Django предназначен для настройки пользовательских сообщений об ошибках, которые будут выводиться, если при валидации поля возникнет ошибка.

1. По умолчанию, каждое поле формы имеет набор стандартных сообщений об ошибках, связанных с различными типами ошибок (например, "Обязательное поле", "Введите действительный электронный адрес" и т.д.).
2. Разработчик может переопределить эти стандартные сообщения об ошибках, передав в конструктор поля словарь `error_messages`, где ключами будут коды ошибок, а значениями - пользовательские сообщения.

**Пример:**

```Python
from django import forms

class MyForm(forms.Form):
    email = forms.EmailField(
        error_messages={
            "required": "Пожалуйста, введите ваш email-адрес.",
            "invalid": "Введенный адрес электронной почты недействителен."
        }
    )
```

В этом примере мы переопределили сообщения об ошибках для поля `email`. Теперь, если при валидации этого поля возникнет ошибка "required" или "invalid", будут выведены наши пользовательские сообщения вместо стандартных.

Сообщения об ошибках могут включать в себя подстановочные значения, которые будут автоматически заменены соответствующими значениями во время формирования окончательного сообщения. Например:

```Python
class MyForm(forms.Form):
    age = forms.IntegerField(
        min_value=18,
        error_messages={
            "min_value": "Вы должны быть не моложе %(limit_value)s лет."
        }
    )
```

В этом случае, если введенное значение будет меньше 18, будет выведено сообщение "Вы должны быть не моложе 18 лет."

### Встроенные валидаторы Django

В Django уже предусмотрена логика для некоторых типов валидации. Эти классы можно передавать в параметр `validators` поля формы, следующим образом

```Python
from django.core.validators import MinLengthValidator, MaxLengthValidator


slug = forms.SlugField(max_length=255, label="URL", validators=[
        MinLengthValidator(5),
        MaxLengthValidator(100),
    ])
```

**Описание встроенных валидаторов**

|   |   |   |   |
|---|---|---|---|
|Валидатор|Описание|Синтаксис|Параметры|
|`RegexValidator`|Проверяет, что значение соответствует заданному регулярному выражению.|`RegexValidator(regex=None, message=None, code=None, inverse_match=None, flags=0)`|- `regex`: регулярное выражение в виде строки или пре-скомпилированного регулярного выражения  <br>-   <br>`message`: сообщение об ошибке  <br>-   <br>`code`: код ошибки   <br>-   <br>`inverse_match`: если `True`, валидатор поднимает ошибку, если совпадение найдено   <br>-   <br>`flags`: флаги для компиляции регулярного выражения|
|`EmailValidator`|Проверяет, что значение выглядит как корректный email-адрес.|`EmailValidator(message=None, code=None, allowlist=None)`|- `message`: сообщение об ошибке  <br>-   <br>`code`: код ошибки  <br>-   <br>`allowlist`: список разрешенных доменов, для которых не применяется проверка|
|`URLValidator`|Проверяет, что значение выглядит как корректный URL.|`URLValidator(schemes=None, regex=None, message=None, code=None, max_length=2048)`|- `schemes`: список разрешенных схем URL  <br>-   <br>`regex`: регулярное выражение для проверки URL  <br>-   <br>`message`: сообщение об ошибке  <br>-   <br>`code`: код ошибки  <br>-   <br>`max_length`: максимальная допустимая длина URL|
|`validate_email`|Экземпляр `EmailValidator` без специальных настроек.|`validate_email`|(нет параметров)|
|`validate_slug`|Экземпляр `RegexValidator`, проверяет, что значение состоит только из букв, цифр, подчеркиваний и дефисов.|`validate_slug`|(нет параметров)|
|`validate_unicode_slug`|Экземпляр `RegexValidator`, проверяет, что значение состоит только из Юникод-букв, цифр, подчеркиваний и дефисов.|`validate_unicode_slug`|(нет параметров)|
|`validate_ipv4_address`|Экземпляр `RegexValidator`, проверяет, что значение выглядит как корректный IPv4-адрес.|`validate_ipv4_address`|(нет параметров)|
|`validate_ipv6_address`|Проверяет, что значение выглядит как корректный IPv6-адрес.|`validate_ipv6_address`|(нет параметров)|
|`validate_ipv46_address`|Проверяет, что значение выглядит как корректный IPv4 или IPv6-адрес.|`validate_ipv46_address`|(нет параметров)|
|`validate_comma_separated_integer_list`|Экземпляр `RegexValidator`, проверяет, что значение представляет собой список целых чисел, разделенных запятыми.|`validate_comma_separated_integer_list`|(нет параметров)|
|`int_list_validator`|Экземпляр `RegexValidator`, проверяет, что значение представляет собой список целых чисел, разделенных заданным разделителем.|`int_list_validator(sep=',', message=None, code='invalid', allow_negative=False)`|- `sep`: разделитель в списке целых чисел   <br>-   <br>`message`: сообщение об ошибке  <br>-   <br>`code`: код ошибки   <br>-   <br>`allow_negative`: разрешать отрицательные числа|
|`MaxValueValidator`|Поднимает `ValidationError`, если значение больше `limit_value`.|`MaxValueValidator(limit_value, message=None)`|- `limit_value`: максимально допустимое значение   <br>-   <br>`message`: сообщение об ошибке|
|`MinValueValidator`|Поднимает `ValidationError`, если значение меньше `limit_value`.|`MinValueValidator(limit_value, message=None)`|- `limit_value`: минимально допустимое значение   <br>-   <br>`message`: сообщение об ошибке|
|`MaxLengthValidator`|Поднимает `ValidationError`, если длина значения больше `limit_value`.|`MaxLengthValidator(limit_value, message=None)`|- `limit_value`: максимальная допустимая длина   <br>-   <br>`message`: сообщение об ошибке|
|`MinLengthValidator`|Поднимает `ValidationError`, если длина значения меньше `limit_value`.|`MinLengthValidator(limit_value, message=None)`|- `limit_value`: минимальная допустимая длина   <br>-   <br>`message`: сообщение об ошибке|
|`DecimalValidator`|Поднимает `ValidationError` при превышении ограничений на количество цифр и десятичных знаков.|`DecimalValidator(max_digits, decimal_places)`|- `max_digits`: максимальное количество цифр   <br>-   <br>`decimal_places`: максимальное количество десятичных знаков|
|`FileExtensionValidator`|Поднимает `ValidationError`, если расширение файла не входит в список разрешенных.|`FileExtensionValidator(allowed_extensions, message, code)`|- `allowed_extensions`: список разрешенных расширений файлов   <br>-   <br>`message`: сообщение об ошибке   <br>-   <br>`code`: код ошибки|
|`validate_image_file_extension`|Использует Pillow, чтобы убедиться, что value.name (value - файл) имеет допустимое расширение изображения.|`validate_image_file_extension`|(нет параметров)|
|`ProhibitNullCharactersValidator`|Поднимает `ValidationError`, если значение содержит символы NULL.|`ProhibitNullCharactersValidator(message=None, code=None)`|- `message`: сообщение об ошибке   <br>-   <br>`code`: код ошибки|
|`StepValueValidator`|Поднимает `ValidationError`, если значение не является кратным `limit_value` с учетом `offset`.|`StepValueValidator(limit_value, message=None, offset=None)`|- `limit_value`: шаг для допустимых значений   <br>-   <br>`message`: сообщение об ошибке   <br>-   <br>`offset`: смещение для проверки кратности к `limit_value`|

### Кастомный валидатор в виде функции

1. Ипортируйте необходимые модули и определите функцию-валидатор, которая будет реализовывать нужную вам логику, например, проверять, что число является четным:
    
    ```Python
    # validators.py
    from django.core.exceptions import ValidationError
    
    
    def validate_even(value):
        if value % 2 != 0:
            raise ValidationError(f"{value} is not an even number")
    ```
    
    - Функция принимает значение `value` и проверяет, что оно делится на 2 без остатка.
    - Если число не является четным, она поднимает `ValidationError` с пользовательским сообщением об ошибке.
2. Импортируйте необходимые модули в forms.py и создайте форму с полем, использующим валидатор:
    
    ```Python
    from django import forms
    from .validators import validate_even
    
    
    class MyForm(forms.Form):
        even_field = forms.IntegerField(validators=[validate_even])
    ```
    
    - Определяется поле формы `even_field` типа `IntegerField`.
    - В аргумент `validators` передается список, содержащий функцию-валидатор `validate_even`.
3. Используйте форму:
    
    ```Python
    form = MyForm(data={'even_field': 7})
    if form.is_valid():
    # form.cleaned_data['even_field'] будет содержать только четные числа
        print(form.cleaned_data['even_field'])
    else:
        print(form.errors)
    ```
    
    - Создается экземпляр формы `MyForm` с данными, содержащими нечетное число.
    - Вызывается метод `is_valid()`, который автоматически запускает валидацию поля `even_field` с помощью `validate_even`.
    - Если значение не проходит валидацию, в `form.errors` будут записаны ошибки.
    - Если валидация пройдена успешно, `form.cleaned_data['even_field']` будет содержать только четное число.

### Кастомный валидатор в виде класса

1. **Импортируйте необходимые модули**:
    
    ```Python
    # validators.py
    from django.core.exceptions import ValidationError
    from django.utils.deconstruct import deconstructible
    ```
    
2. **Определите класс-валидатор** `**EvenNumberValidator**`:
    
    ```Python
    @deconstructible
    class EvenNumberValidator:
        code = 'not_even'
        default_message = "%(value)s is not an even number."
    
        def __init__(self, message=None):
            self.message = message or self.default_message
    
        def __call__(self, value):
            if value % 2 != 0:
                raise ValidationError(
                    self.message,
                    code=self.code,
                    params={'value': value}
                )
    ```
    
    - Класс отмечен декоратором `@deconstructible`, чтобы он мог корректно сериализоваться для системы миграций.
    - Определён атрибут `code`, который будет использоваться в качестве кода ошибки.
    - Определён атрибут `default_message` с сообщением об ошибке по умолчанию.
    - В `__init__()` принимается необязательный аргумент `message`, который переопределяет сообщение об ошибке.
    - Метод `__call__()` реализует логику проверки: если значение `value` не является чётным, поднимается `ValidationError` с соответствующим сообщением и кодом ошибки.
3. **Использование** `**EvenNumberValidator**` **в форме или модели**:
    
    ```Python
    from django import forms
    from .validators import EvenNumberValidator
    
    class MyForm(forms.Form):
        even_field = forms.IntegerField(validators=[EvenNumberValidator()])
    ```
    
    - Импортируем определённый ранее класс-валидатор `EvenNumberValidator`.
    - В `MyForm` поле `even_field` типа `IntegerField` использует `EvenNumberValidator` в качестве валидатора.

### Метод clean_field в классе формы

Создание кастомного метода `clean_even_field()` позволяет реализовать дополнительную логику очистки и валидации для конкретного поля формы, не затрагивая общую логику валидации. Это полезно, когда необходимо выполнить сложные проверки, выходящие за рамки возможностей стандартных валидаторов.

1. **Определение класса формы с кастомным методом очистки**:
    
    ```Python
    from django import forms
    from django.core.exceptions import ValidationError
    
    
    class MyForm(forms.Form):
        even_field = forms.IntegerField()
    
        def clean_even_field(self):
            even_field = self.cleaned_data.get('even_field')
            if even_field % 2 != 0:
                raise ValidationError(
                    "The value for even_field must be an even number."
                )
            return even_field
    ```
    
2. **Логика метода** `**clean_even_field()**`:
    
    - Метод получает значение `even_field` из `self.cleaned_data`, которое было очищено стандартными методами очистки формы.
    - Если значение не является чётным, метод поднимает `ValidationError` с сообщением об ошибке.
    - Если всё проверки пройдены успешно, метод возвращает очищенное значение `even_field`.
<div class="page-break" style="page-break-before: always;"></div>
