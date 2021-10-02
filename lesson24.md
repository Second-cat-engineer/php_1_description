## Контроллеры и фронт-контроллер
***

## Контроллер в MVC
### Контроллер - это точка входа
- Ее задача принять запрос от клиента и понять, что хочет клиент;
- Контроллер работает с данными через модели;
- Контроллер готовит данные для представления и передает их ему;
- Контроллер отвечает за выдачу ответа клиенту (возможно - используя слой View).

### Контроллер не должен быть:
- Толстым;
- Тупым;
- Уродливым.

Чем меньше кода в контроллере - тем правильнее выбрана архитектура проекта.
Валидации данных в контроллере быть не должно, это задача модели в любом случае.
Не должно быть какой-то бизнес логики.


### Класс контроллер нужен, чтобы:
- Контроллер был объектом, чтобы получить все возможности ООП;
- Чтобы использовать преимущества наследования.
~~~php
namespace App\Controllers;

class News extends \App\Controller
{
    public function action();
}
~~~

Немного "магии":
- Иногда бывает так, что у объекта есть только один публичный метод
- В таком случае можно не давать этому методу какое-то отдельное имя. Достаточно назвать его __invoke() и тогда можно вызывать этот метод, не называя его -весь объект станет "функцией"!
~~~php
namespace App\Controllers;

class News extends \App\Controller
{
    protected $view;
    
    public function __construct() {
       $this->view = new View();
    }
    
    public function __invoke() {}
}

$controller = new News;
$controller(); // или $controller->__invoke();
~~~
__invoke() приводит к тому, что объект становится callable объектом, и может быть вызвана как функция.


### Что еще можно сделать в контроллере:
- В конструкторе создать нужные служебные объекты, например View;
- Или получить там ссылки на объекты, например - текущего пользователя;
- Создать метод beforeAction(), чтобы выполнить какие-то действия до действия;
- Написать метод access($action), чтобы проверять права доступа.

Главная дилемма - вызвать ли View в явном виде в action, либо это делать позже? Каждый фреймворк решает это по-своему.


~~~php
namespace App\Controllers;

use App\View;
use App\Models\News;

class News extends \App\Controller
{
    protected $view;
    
    public function __construct()
    {
       $this->view = new View();
    }
    
    public function actionIndex()
    {
        $this->view->title = 'My app';
        $this->view->news = News::findAll();
        $this->view->display(__DIR__ . '/../templates/index.php');
    }
}
~~~

~~~php
// controllers/News.php
namespace App\Controllers;

use App\View;
use App\Models\News;

class News extends \App\Controller
{
    protected $view;
    
    public function __construct()
    {
       $this->view = new View();
    }
    
    public function action($action)
    {
        $methodName = 'action' . $action;
        $this->beforeAction();
        return $this->$methodName();
    }
    
    protected function beforeAction()
    {
        echo 'Hi';
    }
    
    protected function actionIndex()
    {
        ...
    }
    
    protected function actionOne()
    {
        $id = $_GET['id'];
        $this->view->article = News::findById($id);
        $this->view->display(__DIR__ . '/../template/article.php');
    }
}

// one.php
require __DIR__ . '/autoload.php';

$controller = new \App\Controllers\News();

$controller->action('One');     // one.php
$controller->action('Index');   // index.php
~~~


### Фронт контроллер и роутер
Самый главный вопрос - а где же создать контроллер? В фронт-контроллере

Этим термином обозначают ту часть программы, где:
- Инициализируют приложение (выполняет некие начальные действия);
- В зависимости от запроса пользователя определяет, какой контроллер и экшн нужно вызвать (это называется "роутинг", а часть программы - роутер). В $_SERVER['REQUEST_URI'];
- Вызывает из;
- Возможно - обрабатывает ответ от действия.