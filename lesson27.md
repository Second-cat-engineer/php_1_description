## Возможности PHP
***

## Анонимные функции.
#### Анонимная функция - это функция, которая в отличие от обычной, не имеет имени
~~~php
function ($x) {
    return $x * 2;
}
~~~

- Анонимную функцию можно вызвать сразу же, "на месте":
~~~php
echo (function ($x) {
    return $x * 2;
})(2);
~~~
- Можно присвоить переменной и так вызвать:
~~~php
$f = function ($x) {
    return $x * 2;
};

echo $f(2);
~~~

В чем профит такой функции? Всю память, которую мы займем внутри функции, мы автоматически освободим когда функция закончится.


#### Анонимная функция - это "объект первого класса":
- Она создается в рантайме (во время исполнения);
- Анонимную функцию можно присвоить переменной;
- Ее можно вызвать из функции:
~~~php
function getFilter() {
    return function ($s) {
        return str_replace(' ', '', $s);
    }
}

$filter = getFilter();
echo $filter('Hello, world!');
~~~
- Ее можно передать в функцию:
~~~php
function apply($str, callable $filter) {
    return $filter($str);
}

echo apply(
    'Hello, world!',
    function ($s) {
        return str_replace(' ', '', $s);
    }
);
~~~
___


## Замыкание
### Замыкание - это способ передать функции контекста:
- Замыкание позволяет "замкнуть" на анонимную функцию какое-либо значение из родительского контекста:
~~~php
$hello = 'Hello';
$h = function ($s) use ($hello) {
    return $hello . ' ' . $s;
}

...

echo $h('World');
~~~
- Замыкание - это не глобальная переменная. Значение замыкания "фиксируется" на момент создания анонимной функции;
- Важно: именно на момент создания, а не на момент вызова функции.
~~~php
$translate = [
    'hello' => 'Привет'
];

$hello = function ($name) use ($translate) {
    return $translate['hello'] . ', ' . $name;
};

function ctrl($name, $func) {
    return $func($name);
}

echo ctrl('Saf', $hello); // Привет, Saf
~~~

То значение, которое у замыкаемой переменной было в момент создания функции, с этой функцией и останется.

#### На вопрос "а что такое замыкания?" сказать: "скорее всего вы имеете в виду анонимные функции".
(Разработчики php ошиблись, они сами анонимные функции назвали замыканиями). Рассказать про анонимные функции, а потом
"а еще есть замыкание контекста".

### Стрелочные функции
Вместо:
~~~php
$m2 = function ($x) {
    return $x * 2;
}
~~~
Можно:
~~~php
$m2 = fn($x) => $x * 2;
~~~

Важно: стрелочные функции автоматически замыкают на себя весь контекст места их "рождения".
___


## Генераторы
#### Генераторы - это функция, которая умеет генерировать последовательность значений, за один раз возвращая только одно из значений, останавливаясь и возвращая следующее, когда это требуется.
Выглядит генератор как функция, но вместо return - yield:
~~~php
function generate() {
    for ($x = 1; $x < 10; $x++) {
        yield $x;
    }
}
~~~

Фактически же генератор функцией не является. Это просто сахар для создания объекта с интерфейсом Iterator.

Единственное применение генератора, это его использование в контексте foreach:
~~~php
foreach (generate() as $num) {
    echo $num;
}
~~~

Главный смысл: использование памяти. Даже в случае последовательности из 1000 элементов, требуется память для одного.


Пример, требуется память для всех 100000 элементов:
~~~php
function firstNums() {
    $ret = [];
    
    $i = 1;
    while ($i <= 100000) {
        $ret[] = $i;
        $i++;
    }
    
    return $ret;
}

foreach (firstNums() as $num) {
    echo $num;
    echo '<br>';
}
~~~
Генератор:
~~~php
function firstNums() {
    $i = 1;
    while ($i <= 100000) {
        yield $i;
        $i++;
    }
}

foreach (firstNums() as $num) {
    echo $num;
}
~~~

Как это работает: foreach видит, что это генератор (ключевое слово yield вместо return).
Что делает foreach? Вызывает эту функцию, она возвращает первое значение, после чего эта функция ставится как бы на паузу.
Цикл foreach первое значение превращает в $num, цикл выполняется, функция снимается с паузы, начинает выполняться дальше,
выбрасывает следующее значение и т.д. пока не закончится.

Генератор - это функция, которая умеет ставиться на паузу и возвращать по одному значению.
Фактически же генератор функцией не является. Это просто сахар для создания объекта с интерфейсом Iterator.
Единственное применение генератора, это его использование в контексте foreach.
Главный смысл: использование памяти. Даже в случае последовательности из 1000 элементов, требуется память для одного.


#### Особенности Генераторов
- Можно генерировать значения с ключами:
~~~php
function generate() {
    for ($x = 1; $x < 10; $x++) {
        yield $x => $x * 100;
    }
}

foreach (generate() as $i => $num) {
    echo $i . '=' . $num;
}
~~~
- Можно делегировать генерацию:
~~~php
function count_to_ten() {
    yield 1;
    yield 2;
    yield from [3, 4, 5, 6];
    yield from get78(); // Делегирование, когда 1 генератор обращается к массиву или другому генератору
    yield 9;
    yield 10;
}
~~~
___


## Переменное число аргументов
Вообще говоря в php в функцию можно передать большее число аргументов чем требуется. Да он будет в функцию передан,
но имени у него нет, и доступа тоже.

Однако есть функция 
> func_get_args();

Если эту функцию вызвать внутри другой функции, она возвращает массив аргументов переданных в данную.
~~~php
function sum($x, $y) {
    $x = func_get_args();
    var_dump($x);
}
~~~

Так неявно, есть более явная запись
~~~php
function sum(...$nums) {
    return array_sum($nums);
}

echo sum(1, 2, 3);
echo sum(...[1, 3, 4]);
~~~

~~~php
function test($a, $b, ...$options) {
    ...
}
~~~
$a и $b - обязательные, получат это имя, остальные упакованы в массив.


Такая же обратная операция есть со []:
~~~php
$x = 10;
$y = 20;
$nums = [$x, $y]; // [10, 20]

$nums = [10, 20];
[$x, $y] = $nums; // x = 10, y = 20
~~~
___


## Тернарные операторы
### Традиционный "?:":
~~~php
$x ? $a : $b;
// Эквивалентно
if ($x === true) {
    $a;
} else {
    $b;
}
~~~

if это оператор выбора "что сделать?".
Тернарный оператор - это оператор выбора, что из 2х значений взять для дальнейшего использования.
~~~php
$vip = true;
$score = $vip ? 100 : 1; // в зависимости от значения переменной $vip запиши 100 либо 1

$model = ['name' => 'Saf'];
$user = !empty($model) ? $model : 'Guest';
~~~

### Сокращенный "?:":
~~~php
$x ?: $b;
// Эквивалентно
if ($x === true) {
    $x;
} else {
    $b;
}
~~~

~~~php
$model = ['name' => 'Saf'];
$user = $model ?: 'Guest'; // проверит, что значение слева существует и не пусто, иначе справа.
~~~


### Умный "??":
~~~php
$x ?? $b;
// Эквивалентно
if (isset($x)) {
    $x;
} else {
    $b;
}
~~~

~~~php
$model = ['name' => 'Saf'];
$user = $model ?? 'Guest'; // если существует значение слева, то возьми его, иначе возьми правый
~~~

?? - isset (если существует)

 
### Космический <=>:
Оператор сравнения
~~~php
$x <=> $b;
// эквивалентно
if ($x < $y) {
    -1;
} elseif ($x == $y) {
    0;
} elseif ($x > $y) {
    1;
}
~~~

~~~php
$arr = [
    ['name' => 'Name1', 'date' => '2000'],
    ['name' => 'Name2', 'date' => '2001'],
    ['name' => 'Name3', 'date' => '1999'],
];

usort($arr, function ($x1, $x2) {
    if ($x1['date'] > $x2['date']) {
        return 1;
    } elseif ($x1['date'] < $x2['date']) {
        return -1;
    } else {
        return 0;
    }
});

// А можно
usort($arr, function ($x1, $x2) {
    return $x1['date'] <=> $x2['date'];
});
~~~
___

## Константные выражения
#### Использовать в определении констант константные выражения (т.е. есть выражения, которые могут быть вычислены на этапе компиляции).
~~~php
const TWELVE = 6 * 2;
const DOZEN = TWELVE . 'eggs';
~~~

Создавать константы массивы:
~~~php
const STATUSES = [
    'SUCCESS' => 0,
    'FAIL' => 1,
];

$this->status = STATUS['SUCCESS'];
~~~

#### Функции и константы тоже могут быть в пространствах имен, как и классы:
- Определение констант внутри пространства имен:
~~~php
namespace Math {
    const E = 2.71828;
    
    function tg($x) {
        return sin($x) / cos($x);
    }
}
~~~

- Обращение по полному имени:
~~~php
echo Math\E;
echo Math\th(1);
~~~

- Импорт:
~~~php
use function Math\tg;
use function Math\tg as tangens;
use conts Math\E;
~~~
___

## Улучшения тайп-хинтинга:
- Возвращаемый тип void:
~~~php
function test(): void
{
    // можно без return
    // можно просто return;
    // нельзя return null;
    // нельзя return любоезначение;
}
~~~

- Псевдотип iterable:
~~~php
function test(iterable $arg) {
    foreach ($arg as $x) { ... }
}
~~~

- Nullable-хинты:
~~~php
function test(?int $x): ?string
{
}
~~~

- Тип object:
~~~php
function test(object $arg);
~~~
___


## Анонимные классы
#### Анонимный класс - это класс, не имеющий формально имени, создающийся "на лету".
Анонимный класс не висит в памяти
~~~php
interface Logger {
    public function log(string $msg);
}


class Application {
    ...
    
    public function setLogger(Logger $logger)
    {
        $this->logger = $logger;
    }
}


$app = new Application;
$app->setLogger(
    new class implements Logger {
        public function log(string $msg) {
            echo $msg;
        }
});
~~~