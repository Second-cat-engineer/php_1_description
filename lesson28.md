## Обзор современных фреймворков
***

### Зачем нужны фреймворки:

#### ВО-ПЕРВЫХ фреймворк – это набор инструментов. Которые уже написаны за вас. Не нужно изобретать велосипед и писать свои:
- Набор функций для работы с базой данных;
- Правила валидации входящих данных;
- Систему HTML-форм;
- Подсистему интернационализации;
- Классы конфигов разных форматов;
- Консольные команды;

Всё уже написано до вас!


#### ВО-ВТОРЫХ фреймворк – это набор методик и технологий:
- MVC;
- Реализация паттерна Dependency Injection;
- Работа с источниками данных на базе схем ORM и ActiveRecord;
- Строгая система именования классов, пространств имен, компонентов приложения;
- Часто – автоматическая генерация нужного вам «рутинного» кода;


#### В-ТРЕТЬИХ фреймворк – это быстрый старт:
- Удобное развертывание;
- Мгновенное создание начального веб-приложения;
- Актуальная документация;
- Сообщество разработчиков;

#### И САМОЕ ГЛАВНОЕ Фреймворк – это инверсия контроля:
Библиотека: Вы подключаете ее и Ваш код вызывает функции, классы, методы библиотеки внутри себя.
Фреймворк: Вы пишете код, который будет вызываться фреймворком!
___


### MVC
Вопросы:
- Есть ли контроллер, от которого можно унаследовать свой?
- Если ли объект представления?
- Есть ли базовая модель?
- Возможно существует объект приложения?
- Роутер? Правила роутинга? Каскад правил?
- Что насчет сервисов? Часто они называются «компоненты приложения»?
- Как настраивается конфигурация приложения?
- Как устроены шаблоны?

### Работа с БД. Модели
#### МИГРАЦИИ – способ управления структурой базы данных из приложения.
Вопросы:
- Как создать миграцию, есть ли генератор кода миграций?
- Могут ли миграции выполняться транзакционно? (зависит, конечно, от БД)
- Как накатить миграцию?
- Предусмотрен ли откат миграций?
- Какие методы предусмотрены для упрощения написания миграций?
- Возможно ли написать в миграции просто SQL?

#### МОДЕЛИ – как реализация паттернов ORM и/или ActiveRecord
Есть ли базовый класс «абстрактная модель», от которого можно унаследовать свои модели или иной механизм?
##### ORM:
- Получение из БД моделей нужного класса
- Коллекции?
- Связи и отношения?
- Построение запросов к БД?
- Различные хранилища данных (DBAL?) Паттерны хранения данных?
##### ActiveRecord:
- save() или аналоги (может быть insert() / update() )
- delete() или аналоги
- статус синхронизации модели

#### ПРОЧЕЕ
##### КОНСОЛЬНЫЕ КОМАНДЫ – такая же важная часть вашего приложения, как и веб!
- Как написать консольную команду?
- Как ее запустить?
- Можно ли составить справку по консольным командам? 

Более серьезные возможности:
- Собственный менеджер расписания
- Очередь команд
- Многопроцессность и/или многопоточность


#### ВНЕДРЕНИЕ ЗАВИСИМОСТЕЙ (DEPENDENCY INJECTION)
Было:
~~~php
class Database {
    ...
}

class User {
    public function getUser($id) {
        $db = new Database;
        return $db->findOneUser($id);
    }
}

$user = new User;
$user->getUser(128500);
~~~

Стало:
~~~php
class Database {
    ...
}

class User {
    protected $db;
    
    public function setDataBase($db) {
        $this->db = $db;
    }
    
    public function findUser($id) {
        return $this->db->findOneUser($id);
    }
}
$user = new User;
$user->setDataBase($db);
$user->getUser(128500);
~~~
Недостаток: можем забыть setDatabase. Нужно засунуть в конструктор
~~~php
class Database {
    ...
}

class User {
    protected $db;
    
    public function __construct($db) {
        $this->db = $db;
    }
    
    public function findUser($id) {
        return $this->db->findOneUser($id);
    }
}

$user = new User($db);
$user->getUser(128500);
~~~


#### Паттерн Сервис контейнер. Возвращает настроенные объекты
~~~php
class Container {
    public function getUser() {
        $db = new Database;
        $user = new User($db);
        return $user;
    }
}

$container
    ->getUser()
    ->findUser(128500);
~~~










