## Куки. Сессии. Авторизация.
***

## HTTP
### HYPER TEXT TRANSFER PROTOCOL
Из чего состоит запрос:
- Метод, URL, версия протокола:
> GET/HTTP/1.1
- Обязательный заголовок Host:
> Host:lenta.ru
- Другие заголовки:
> User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36
- Данные.

Из чего состоит ответ:
- HTTP/версия, код состояния, пояснение:
> HTTP/1.1 200 OK
- Другие заголовки:
> Content-type: application/json
- Два перевода строки;
- Тело ответа.

#### Проблема в том, что мы не можем идентифицировать клиента. По сути разговор всегда начинается заново.
___


## COOKIE
- Это специальный заголовок ОТВЕТА СЕРВЕРА:
> Set-Cookie: foo=bar
- Это ХРАНИЛИЩЕ в браузере, где он обязан сохранять поступившие ему от сервера пары "ключ-значение". Обычно данные cookie хранятся в небольших текстовых файлах;
- Браузер при следующем запросе обязан выслать хранящиеся данные серверу с помощью заголовка:
> Cookie: foo=bar

#### Подробности о cookie
Атрибуты:
- Ключ и значение
- Срок действия. Если нет - до закрытия браузера
- Путь (можно на определенную страницу, если корень то весь сайт)
- Домен (автоматически доступны и для поддоменов)
- Признаки HTTPS (запрет на передачу от клиента по незашифрованному соединению)
- HTTPOnly (недоступен через JavaScript)

> Set-Cookie: foo=bar; expires=Fri, 31 Dec 2010 23:59:59 GMT; path=/; domain=.example.net
___

- Сервер может лишь попросить клиента установить cookie, то что отослали, не значит, что клиент их сохранит (может это вообще не браузер);
- У клиента cookie могут не сохраняться по тем или иным причинам, вы не контролируете этот процесс никак. Отправили заголовок, все, на этом соединение закончено;
- Клиент может не передать cookie в ответ;
- Убедиться то что клиент сохранил cookie можно только при следующем запросе.
- 


#### Использование cookie в php
Отдавать заголовки нужно до вывода любого текста:
~~~php
header('set-cookie: foo=bar');
~~~

Можно проще:
~~~php
setcookie('foo', 23, time() + 86400);
~~~

Для установки cookie (передача заголовка клиенту) используют функцию setcookie():
- $name
- $value
- $expired
- $path
- $domain
- $secure
- $httponly

#### Для чтения cookie (переданных от клиента!) использовать суперглобальный массив $_COOKIE. 

В $_COOKIE содержится сведения о сохраненных куках, которые сообщил клиент при данном запросе. Только читать! Запись в него не имеет смысла.

Это массив, который содержит то, что клиент сообщил. Как любые данные от клиента, верить этому не стоит, т.к. небезопасная вещь.
Использовать куки нужно по назначению - пометить как либо клиента.

В куках нельзя хранить какую-то конфиденциальную информацию, только некие метки, которые говорят о том, кто этот клиент (хорошая идея - случайное число, плохая - email клиента, потому что любой, кто передал этот email станет этим клиентом).


#### Безопасность в cookie:
- Браузер может не сохранить куку;
- Браузер может не передать ее при следующем запросе;
- Клиент может удалить куку из браузера;
- Клиент может изменить любые данные в куке;
- Куку могут украсть;
- Данные cookie могут быть подделаны MitM.

#### Вывод: в cookie нельзя хранить никакие чувствительные данные!
___


## SESSION
### Сессии - встроенный в php способ хранения данных на сервере между последовательными запросами одного и того же клиента:
- Уникальный идентификатор сессии;
- Выбор способа передачи идентификатора (cookie или get-параметр);
- Выбор хранилища данных сессий;
- Удобный интерфейс для доступа к данным: массив $_SESSION.

#### На каждой страниц, которую может посетить клиент:
~~~php
session_start();
~~~
Присвоит уникальную куку, чтоб не подобрать. Действует до закрытия браузер.
~~~php
$count = $_SESSION['count'] ?? 0;
$count++;
$_SESSION['count'] = $count;

echo $count;
~~~

#### Полезные функции:
- session_start();
- session_id();
- session_regenerate_id();
___


### Кука идентифицирует клиента, на сервере эти данные хранятся.
Механизм сессии позволяет решить проблему с хранением данных конфиденциальных, т.е. кука ставится на клиента, она случайна, а все данные связанные с ним уже хранятся на сервере.
Это позволяет безопасно реализовать корзину с товарами, сбор заказа и т.д., так, чтобы при переходе на страницу со страницы эта информация не потерялась, пока клиент не закроет браузер.
___

### Хэш-функции
#### Это функция шифрования (одностороннего), позволяющее по тексту произвольной длины получить некий короткий, достаточно случайный "хэш" - идентификатор текста.

Двухстороннее шифрование rea;
Односторонние - после уже не восстановить;

Алгоритмы: md5, sha1, sha256, sha512, gost-crypto.


#### Хэш функции позволяют:
- "Свертывать" длинные значения в короткие;
- Получить псевдослучайные строки (используя время, соль, рандом);
- Односторонние шифрование.

#### Встроенные функции хэширования.
Хэширование часто применяется для одностороннего шифрования пароля:
- Храним в базе пользователей не пароли, а хэши паролей;
- Когда пользователь вводи пароль в форму входа - снова вычисляем хэш;
- Если вычисленный и сохраненный хэши совпали - считаем проверку пароля пройденной;

В php следует ВСЕГДА использовать встроенные функции:
- password_hash();
- password_verify()
они обеспечивают максимально безопасное хэширование и проверку хэша.
___


### Идентификация. Аутентификация. Авторизация.
Идентификация - когда клиент называет себя (он себя идентифицировал, но вы еще не проверили, не знаем точно он или нет, нужно задать контрольный вопрос);
Аутентификация - процесс проверки идентифицированной информации;
Авторизация - выдача доступа к ресурсам когда вы убедились что это - он (вы показываете страницу админа и т.п).
___


### Еще раз о безопасности:
- Аксиома некомпетентности (готовое решение лучше велосипеда);
- Логирование всего и вся;
- Проактивная защита (Google Captcha например);
- Поведенческая защита (временный пароль на почту или телефон).