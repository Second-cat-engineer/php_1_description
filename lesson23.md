## Изоляция уровня представления
***

## Представление в MVC
### Представление создает внешний вид:
- Его задача - сформировать ответ на запрос клиента;
- Представление использует HTML, CSS, JavaScript;
- Представление может содержать логику, нужную для вывода данных, она может быть написана на PHP - не бойтесь смешивать PHP и HTML.


##### View - это часть MVC;
##### Представление - это логика отображения, как именно мы отображаем данные;
##### Шаблон - это файл, в котором логика представления содержится.


### Правильно разделять представление и бизнес-логику:
- Сверстать шаблон страницы. Обычный HTML;
- Определить, какие данные будут ей переданы;
- Подготовить данные, получив их от моделей;
- Передать (в простейшем случае - просто include);
- В шаблоне реализовать ЛОГИКУ ПРЕДСТАВЛЕНИЯ, то есть то, как будут отображаться данные;
- Создать специальный объект, который будет управлять представлением: хранить в себе данные для него, отображать заданный шаблон с этими данным.
___


## Альтернативный шаблон
### Альтернативный синтаксис конструкций, операторов if, wile, for, foreach, switch без фигурных скобок.
Первое применение альтернативного синтаксиса - не запутаться в фигурных скобках.
~~~html
<ul>
    <?php foreach ($items as $i) : ?>
        <li><?php echo $i->title; ?></li>
    <?php endforeach; ?>
<ul>
~~~

~~~html
if (УСЛОВИЕ):
...
endif;
~~~
~~~html

while (УСЛОВИЕ):
...
endwhile;
~~~
___


## "Магия" в ООП
### "Магией" называют методы (и не только) автоматически вызывающиеся в определенных ситуациях:
- Попытка использования класса, чье имя ранее не встречалось:
~~~php
function __autoload($class);
~~~
- Присваивание значения недоступному (несуществующее или защищенное) свойству:
~~~php
public function __set($key, $value);
~~~
- Чтение значения недоступного (несуществующее или защищенное) свойства объекта:
~~~php
public function __get($key)
~~~
- Проверка существования недоступного свойства объекта
~~~php
public function __isset($key)
~~~

Вся "магия" начинается с двух знаков подчеркивания. Не называть так свои функции!


~~~php
// view.php
namespace App;

class View {
    protected $data = [];
    
    public function __set($k, $v) {
        $this->data[$k] = $v;
    }
    
    public function __get($k){
        return $this->data[$k];
    }
    
    public function display($template) {
        include $template;
    }
}

// index.php
require __DIR__ . '/autoload.php';

$view = new \App\View();
$view->users = App\Models\User::findAll();

$view->display(__DIR__ . '...');
~~~
___


## PHPDOC
### PHPDOC - стандарт документирования кода:
Все комментарии PHPDOC находятся в блоке вида
~~~php
/**
* 
*/
~~~
и располагаются перед комментируемым объектом;


Примеры:
~~~php
/**
* @return int Сумма двух чисел
*/
function sum(int $a, int $b);

/**
* @property \App\Config $config
*/
public $config;

/**
* @deprecated
*/
function someOldAction();
~~~
___

## Стандартные интерфейсы SPL
### SPL: Standard PHP Library
- Есть функция count();
~~~php
$a = [1, 2, 3];
echo count($a); // 3
~~~
- Однако ее никак не применить к объекту
~~~php
$a = new View ;
$a->foo = 1;
$a->bar = 2;
$a->baz = 3;
echo count($a); // 1
~~~
- Решение: реализовать в классе стандартный интерфейс Countable:
~~~php
class View implements \Countable
{
    protected $data = [];
    
    public function __set($k, $v) {
        $this->data[$k] = $v;
    }
    
    public function __get($k){
        return $this->data[$k];
    }
    
    public function display($template) {
        include $template;
    }
    
    public function count(){
     return count($this->$data);
    }
    
    public function render($template) {
        ob_start();
    
        foreach ($this->data as $prop => $v) {
            $$prop = $v;        // переменная, чье имя содержится в переменной
        }
    }
}
~~~