## Модели данных и ООП
***

## Интерфейсы
### Интерфейс - это специальная сущность языка PHP, своего рода "контракт", который обязан выполнить класс.
Интерфейс содержит в себе описание публичных методов, (и констант) которые ОБЯЗАН иметь класс, реализующий этот интерфейс:
~~~php
interface Orderable {
  public function getPrice();
  public function getWeight();
}

class Item implements Orderable {
    public function getPrice() {
        ...
    }
    
    public function getWeight() {
        ...
    }
}
~~~
С точки зрения автозагрузки интерфейс - это класс. Полное имя интерфейса будет передано в функцию автозагрузки.

#### Интерфейсы могут наследоваться друг от друга. Впрочем, это имеет ограниченное применение:
~~~php
interface HasPrice {
    public function getPrice();
}
...
interface Orderable extends HasPrice, HasWeight {
}

class Item implements Orderable ...
~~~


#### Класс может реализовывать несколько интерфейсов.
В этом отличие интерфейса от абстрактного класса. Предок абстрактного класса может быть только один, а реализация интерфейсов может быть несколько.
~~~php
class Item implements HasPrice, HasWeight {
  public function getPrice() { 
    ... 
  }
  
  public function getWeight() {
    ... 
  }
}
~~~
___


## Тайп-хинтинг
### Тайп-хинтинг - это возможность указать ожидаемый тип аргумента функции.
По умолчанию в php нестрогий режим приведения типов.

1. Скалярный. Используем название типов bool, int, float, string:
~~~php
function sum(int $a, int $b) {
    return $a + $b;
}

echo sum(1.2, 2.3); // 3
~~~
2. Массив. Используем тип array:
~~~php
function sum(array $numbers = []) {
    ...
}
~~~

Возможен "строгий режим" контроля типов (TypeError вместо приведения):
~~~php
declare(strict_types=1);

function sum(int $a, int $b) {
    return $a + $b;
}

echo sum(1.2, 2.3); // Fatal error!
~~~


### Тайп-хинтинг - это возможность указать ожидаемый КЛАСС аргумента функции:
1. Точное соответствие имени класса классу объекта:
~~~php
function send(User $u, $message) {
    ...
}
~~~
2. Класс-родитель:
~~~php
function send(User $u, $message) {
    ...
}

class Admin extends User { 
    ...
}

send (new Admin, 'Hello!');
~~~
3. Интерфейс:
~~~php
function send(HasEmail $u, $message) {
    ...
}

send($user, 'Hello!');
~~~
___


## Паттерн "Одиночка"
Как написать класс, чтобы было возможным создание ТОЛЬКО ОДНОГО объекта данного класса? Например, соединение с БД.
1. Запретить нормальное создание объектов этого класса. Например, сделав конструктор непубличным:
~~~php
class Singleton {
    protected function __construct() {}
}

$x = new Singleton(); // fatal error
~~~
2. Предусмотреть статический метод в классе, который будет возвращать объект:
~~~php
class Singleton {
    protected function __construct() {}
    
    public static function instance() {
        return new self;
    }
}
~~~
3. Поручить ему "считать" число объектов:
- Возвращать уже существующий, если есть.
- Или новый, если еще не было.
~~~php
class Singleton {
    protected static $instance;

    protected function __construct() {}
    
    public static function instance() {
        if (null === self::$instance) {
            self::$instance = new self;
        }
        return self::$instance;
    }
}

$s = Singleton::instance(); 
~~~
Сам по себе класс Singleton бесполезен, потому что если нам захочется сделать другой класс синглтоном, нам придется весь код писать заново.
Было бы круто, если он был отдельным, а классы к нему подключались и расширялись.

Как это можно сделать? Сделать abstract class Singleton:
~~~php
abstract class Singleton {
    protected static $instance;

    protected function __construct() {}
    
    public static function instance() {
        if (null === static::$instance) {
            static::$instance = new static();
        }
        return static::$instance;
    }
} 
~~~
Тут возникнет следующая проблема: унаследоваться в PHP можно лишь от одного класса. Поэтому мы не можем какой-нибудь класс
сделать и синглтоном и наследником чего-то еще.

На помощь приходят трейты.

___


## Трейты
### Трейт - это специальная сущность языка php, своего рода "заготовка", которую можно вставить в класс.

Внимание! Это не класс, т.е. нельзя создать экземпляр трейта. Нельзя наследовать один трейт от другого. Это просто текстовая
заготовка, которая содержит в себе методы, свойства неважно динамические или статические. Эту заготовку можно использовать
создавая свои классы.

Трейт может содержать в себе методы и свойства, динамические и статические.
При компиляции программы текст трейта будет вставлен в класс
~~~php
trait DateTime {
    protected $started;
    protected $finished;
    
    public function getDuration() {
        return $this->finished - $this->started;
    }
}

class Order {
    use DateTime;
}
~~~
С точки зрения автозагрузки трейт - это тоже класс. Полное имя трейта будет передано в функцию автозагрузки.


~~~php
trait Singleton {
    protected static $instance;

    protected function __construct() {}
    
    public static function instance() {
        if (null === static::$instance) {
            static::$instance = new static();
        }
        return static::$instance;
    }
}

class Db {
    use Singleton;
}
~~~
___


## CRUD и шаблон ActiveRecord
### ActiveRecord - это архитектурный паттерн "Активная запись":
1. Записи в БД соответствует объект на языке программирования;
2. Запись в базу данных может "сама себя" сохранить и удалить, используя методы объекта.

Например:
~~~php
$user = new User;
$user->name = 'Василий';
$user->email = 'vasya@test.com';

$user->save();
~~~

### CRUD - это сокращение от create, read, update, delete - 4 основные операции с данными