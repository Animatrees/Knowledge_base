**GROUP BY в SQL:**

- Оператор `GROUP BY` в SQL используется для группировки результирующих строк по значениям определенного поля или выражения.
- Он используется в сочетании с агрегирующими функциями, такими как `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` и другими, для вычисления агрегатов (например, общее количество, сумма, среднее значение и т. д.) внутри каждой группы.
- Это позволяет выполнять агрегации на уровне групп, что является необходимым для корректного вычисления агрегированных значений на основе отдельных категорий данных.

```SQL
SELECT country, AVG(salary) as average_salary
FROM employees
GROUP BY country;
```

**2. annotate() с агрегатной функцией:**

- При использовании `annotate()` с агрегирующей функцией, например, `Count`, Django автоматически генерирует SQL запрос с оператором `GROUP BY`, чтобы сгруппировать данные по полям. Важный нюанс - в данном случае группировка происходит по всем полям!
- Это обеспечивает правильное вычисление агрегированных значений для каждой группы данных в запросе.

```Python
from django.db.models import Avg

# Аннотация с AVG
employees = Employee.objects.annotate(average_salary=Avg('salary'))
```

```SQL
# Преобразованный SQL, * - обозначает все поля
SELECT *, AVG(salary) AS average_salary
FROM employees
GROUP BY *;
```

**3. values() с annotate():**

- Метод `values()` позволяет указать конкретные поля, по которым должны быть сгруппированы данные вместо использования всех полей модели.
- При использовании `values()` совместно с `annotate()` можно оптимизировать запросы и уменьшить количество возвращаемых данных.
- Это особенно полезно, если вы заинтересованы только в агрегированных значениях и не требуется получение полных объектов модели.

```Python
# Группировка по стране с AVG
employees = Employee.objects.values('country').annotate(average_salary=Avg('salary'))
```

```SQL
# Преобразованный SQL
SELECT country, AVG(salary) AS average_salary
FROM employees
GROUP BY country;
```

**Порядок** `**annotate()**` **и** `**values()**` **важен!**

- `**values() -> annotate()**` : Сначала группируем, потом вычисляем аннотации по группам.
- `**annotate() -> values()**` : Сначала аннотируем все объекты, потом `values()` оставляет только указанные поля. В этом случае вычисленные аннотации (такие как `average_rating`) нужно включать в `values()` явно.

<div class="page-break" style="page-break-before: always;"></div>
