# 1-й вопрос

Есть бизнес структура. Она состоит из товаров которые продаются в разных ГЕО. При этом в каждом ГЕО есть своя валюта и своя цена (учитывая доставку). К каждому ГЕО, на каждый товар приходят лиды, которых надо фиксировать. При этом есть коэффициент который динамически повышает или понижает цену в зависимости от количества лидов пришедших на ГЕО и товар. По принципу - больше лидов - меньше цена. Рейтинг обновляется раз в 10 минут.
Отразите схему организации мест хранения данных и взаимодействия с ними в виде запросов.


Думаю что нужно выбрать реляционную БД для хранения Лидов. Должна быть таблица ГЕО, на которую ссылаются Лиды. Должна быть таблица коэффициентов для хранения истории.

Реляционная БД:
GEO (PK id, name, x , y, price, currency)
Lids (PK id, FK geo_id,  name, ip, attributes…)
Koeff(PK id,  datetime datetime) – история ведется по дате-времени

Product(id, name, price)
Order(id, product_id, datetime)

Цена товара бывает с доставкой, бывает без доставки. 
total_price = product_price + delivery_price

Формирование цен с доставкой это очень сложный процесс и может различаться от ГЕО, итоговой цены и количества товаров который куплен.

Как это всё работает? Зашёл Лид на сайт и по нему из запроса браузера узнаётся IP-адрес, имя человека из регистраций. Если пользователь пользуется ботом в телеграмм он сообщает своё имя, неопционально номер телефона и имя пользователя в Telegram. Потом сайт делает запросы на внешние источники – сервисы для уточнения информации о пользователе. Это могут быть специальные сайты, уточняющие информацию о пользователе, Facebook. По ним мы узнаём предпочтения пользователя и что лучше потом ему рекламировать с других сайтов где выставляется реклама компании.
Будет CRON, который каждые 10 минут проверяет информацию о количестве пользователей и составляет новые рейтинги.
Страницы которые выдаются Лидам лучше кешировать, чтобы делать меньше запросов к БД. Кешируются информация о товаре, цене товара, характеристики товара. Кеширование можно настроить так чтобы оно происходило каждые 5 минут. Кеширование происходит из реляционной БД в кеш.


# 2-й вопрос
Необходимо сделать сервис для лайк/дизлайк, который может быть интегрирован в разные места проекта, учитывающий что на разных страницах есть разные сущности для лайков. Количество экшнов за день равно от 1млн. Опишите с точки зрения технологии, языков и базовой структуры как вы организуете код и взаимодействие его частей друг с другом

Лайки хранить к Кеше. Через каждые 30 минут загружать их в реляционную БД.
Product(id, name)
Lides(id, name)
Likes(id, product_id, lides_id)
Лайки хранят информацию о товаре и Лиде, но при этом сама таблица  Likes непосредственно не используется а используется его дубликат в виде словаря в кеше.


Но когда страница загружается данные Лайков берутся из Кеша, который формируется из CRON. Когда страница отображается пользователю лайки берутся из этого кеша.
В кешах(словаре) likes хранится ссылка на товар в виде ключа product_id.

Данные кеша могут загружаться в БД обратно опять же через CRON-задачи.


# 3-й вопрос

У вас есть хранимые изображения на жестком диске. их количество более 1млн, при этом вес каждой от 10 до 50 мб. Изображения постоянно дополняются. До этого момента они не были никак каталогизированны. В настоящий момент надо организовать, при каждом новом пополнении выполняется определение похожих картинок и пометка данной категории. 

Таблицы:
category(id, category_id, name) – ссылается на саму себя, большой уровень вложенности категорий
category_image(id, category_id, image_id)
image(id, name)
В таблице категории могут ссылаться друг на друга через поле category_id. Картинки категории связаны через промежуточную таблицу `category_image`, связь многие-ко-многим.

Для опредения похожих картинок я бы выбрал внешний сервис. Лучше воспользоваться сервисом https://www.duplichecker.com/ru или сделать подписку на ChartGPT или другой искусственный интеллект для распознавания образов.

Лучше это делать через CRON. Просто каталогизировать через заданный интервал времени.
Там однозначно, будут применяться сервисы искусственного интеллекта. Возможно ну
Картинки лучше хранить в виде имён связанный с ключами в БД.

Код может выглядеть следующим образом:
1. Цикл по категориям.
2. Узнается есть ли в картинке образ с названием категории
3. Если имеется, то создается запись в таблице `category_image`.

Категории, в данном случае, являются ключами похожести. Оператор БД добавляет ключи похожести, а потом по этим картинкам проверяется есть ли категория в картинке.

 
