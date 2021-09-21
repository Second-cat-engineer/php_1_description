## Быстрый старт: выражения и переменные, типы.
***

### PHP – это язык программирования.

Результат работы программы на php – ЭТО ТЕКСТ, который передается клиенту в ответ на его запрос.

### Выражения и значения.
Выражение - это некое значение, заданное или в явном виде ("литерал") или в виде некоего вычисления (последовательность операций).

~~~
2       // это число
1.5     // это тоже число, но нецелое
'foo'   // это строка из 3 символов
2 + 2   // это выражение, его результат 4
2 * 2   // это выражение, тоже 4
~~~

Выражение всегда имеет значение (то, чему оно равно).

### Переменные.
Переменная - это некое значение (выражение), сохраненное под понятным для нас именем. Например:
~~~
$result = 2 + 2 * 2;
$email = 'test@test.com';
~~~

Имя переменной в php начинается со знака "$". $foo и $FOO - это разные переменны.

### Типы.
Тип - это определение того, что может быть значением.

В php типы имеют значения, а не переменные. Переменная - это просто имя для значения, чтобы его сохранить на будущее.

Тип выводится автоматически:
- int, integer. Целые числа. Любые (включая 0). Замкнуты относительно сложения, вычитания, умножения - результат снова будет целым числом.
- float. Не целые числа (имеют дробную часть, иногда нулевую, но все же имеющие). Являются приближенными.
- string. Строки. В php строки могут быть любой длины, могут быть нулевой длины (пустые), могут заключаться в одинарные или двойные кавычки.
- boolean. Логическое значение, или "истина" (ture) или ложь (false). Ответ на какой-либо вопрос или результат сравнения.

#### Тип результата операции всегда определяется оператором.
Например, арифметические операции всегда производят числа:
~~~
$a = $x + $y;       // всегда число
$a = 2 + '2';       // 4
$a = '2' * '2';     // тоже 4
~~~

А оператор "точка" (сложение строк) всегда производит строки:
~~~
$a = $x . $y;       // всегда строка
$a = 2 . 2;         // '22'
$a = '2' . 2;       // тоже '22'
~~~

Этот механизм называется "приведением типов". Тип значения приводится к наиболее подходящему, чтобы соответствовать типу оператора.