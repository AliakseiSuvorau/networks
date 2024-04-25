# Доп. лекция

## Содержание

* [Протоколы для конфигурации хостов](https://github.com/AliakseiSuvorau/networks/blob/master/Lection-09a.md#протоколы-для-конфигурации-хостов)
  * BOOTP
* [DHCP](https://github.com/AliakseiSuvorau/networks/blob/master/Lection-09a.md#dhcp-dynamic-host-configuration-protocol)
  * Способы назначения ip-адресов
  * Участники DHCP обмена
  * Задачи DHCP
  * Транспорт
  * Формат
  * Опции
  * DHCP Message Type
  * Как работать с временными адресами?
  * Безопасность

## Протоколы для конфигурации хостов

* Задача
  * автоматическое централизованное конфигурирование параметров клиентских систем
    * ip-адрес/маска
    * шлюз по умолчанию
    * предпочитаемые DNS-сервера

* Стандарты
  * RFC 951 - BOOTP
  * RFC 2131 - DHCP

### BOOTP

* Кейс использования
  * Хост без ОС заходит в сеть
  * Хост посылает broadcast-запрос на то, чтобы ему дали ОС
  * Сервер BOOTP видит запрос и посылаем ему в ответ пакеты с некоторыми параметрами
  * Хост обращается к серверу за образом ОС, и т.д.

## DHCP (Dynamic Host Configuration Protocol)

* Развитие BOOTP
  * Умеем все, что умеет BOOTP
  * +динамическое назначение ip-адресов из пула
  * +другие конфиги

### Способы назначения ip-адресов

* Автоматический
  * Постоянный
  * Временный (надо обновлять периодически)
* Ручной
  * Задается сисадмином на DHCP-сервере, DHCP здесь - просто средство доставки этой информации клиенту

### Участники DHCP обмена

* Клиенты (хосты, получающие информацию о сетевых настройках)
* Сервера (специально сконфигурированные хосты, распространяющие сетевые настройки)
* Relay Agent (чаще всего маршрутизаторы, которые ретранслируют DHCP-сообщения в другие сегменты сети; для того, чтобы не нужно было иметь по DHCP-серверу в каждом сегменте сети)

### Задачи DHCP

* Сисадмин должен иметь возможность управления конфигурацией
* Клиенты не должны конфигурироваться вручную
* Сети не конфигурируются вручную
* Не требуется наличие DHCP-серверов в каждом сегменте сети
* DHCP-инфраструктура должна сосуществовать со статической конфигурацией
* Гарантия 1 интерфейс - 1 клиент
* Поддерживать постоянные конфиги, выдаваемые определенным клиентам
* Обслуживать BOOTP-клиентов
* Клиент может получать предложения с сетевыми настройками одновременно от нескольких серверов, а также может выбирать, чьи настройки принять, а чьи отвергнуть. В случае, если настройки сервера отвергнуты, то
  он может выдать их другому клиенту
* DHCP-сервер должен по возможности выдавать клиенту одни и те же сетевые настройки после перезагрузки клиента

### Транспорт

* Broadcast-рассылка (где можем - unicast)
* Дейтаграммный режим передачи

Следовательно,
* UDP (порт сервера - 67, порт клиента - 68)
* размер до 576 октет (минимальный размер MTU, при котором гарантированно не будет фрагментации)

### Формат

* 8 бит - op (тип сообщения)
  * BOOTPREQUEST
  * BOOTPREPLY
* 8 бит - htype (тип канального адреса)
* 8 бит - hlen (размер канального адреса)
* 8 бит - hops (количество пересылок через relay agent)
* 32 бита - xid (id транзакции, чтобы понять, на какой запрос пришел ответ)
* 16 бит - secs
* 16 бит - flags (первый бит говорит, может ли сервер посылать ответ unicast'ом)
* 32 бита - ciaddr (заполняется клиентом, если клиент уже получал какие-то настройки и хочет их обновить)
* 32 бита - yiaddr (назначенный сервером клиентский ip)
* 32 бита - siaddr (ip сервера, для unicast-рассылки)
* 32 бита - giaddr (ip-адрес relay агента, по нему поймем, из какого сетевого диапазона выдавать адрес клиенту)
* 16 октет - chaddr (канальный адрес, например - mac)
* 64 октета - sname (опциональное имя сервера)
* 128 октет - file (имя файла с образом ОС для загрузки)
* Опции

### Опции

* Magic cookie: 99, 130, 83, 99
* Собственно опции:
  * Subnet Mask - 1
  * Router Option - 3 (маршрут по умолчанию)
  * DNS Options - 6 (информация о предпочитаемом DNS-сервере)
  * Host Namr Option - 12
  * Domain Name - 15
  * Static Route Option - 33
  * End Option - 255
  * Paging - 0 (для выравнивания в конце)

> [!IMPORTANT]
> Расширение DHCP (нет в стандарте):
> * vendor class id - 60 (особые настройки для определенных систем)
> * user class id - 77 (опции для особых классов пользователей)
> * DHCP Message Type - 53 (с точки зрения BOOTP - только RESPONSE/REQUEST; тут информация уже для DHCP)

### DHCP Message Type

* DHCPDISCOVER
  * broadcast-запрос клиента для поиска сервера
* DHCPOFFER
  * ответ клиенту от сервера на сообщение DHCPDISCOVER с предложением конфигов
* DHCPREQUEST
  * сообщение от клиента серверу:
    * запрос предложенных параметров от одного сервера (и неявный отказ от предложенных другими серверами параметров, отсылается broadcast'ом)
    * подтверждаение корректности ранее выданных адресов (используется unicast)
    * продление аренды конкретного адреса
* DHCPACK
  * сообщение от сервера клиенту с конфигами, включая ip
* DHCPNAK
  * сообщение от сервера клиенту о том, что его параменты некорректны
* DHCPDECLINE
  * сообщение от клиента серверу о том, что выделенный ip-адрес уже используется (мог быть получен в результате статической конфигурации) 
* DHCPRELEASE
  * сообщение от клиента серверу о том, что он освобождает используемый адрес
* DHCPINFORM
  * сообщение от клиента серверу с запросом только локальных конфигов, если клиент уже имеет ip
  * общение DHCP-серверов друг с другом

### Как работать с временными адресами?

* Клиент вытается обновить адрес через $T_1=0.5 \cdot время-аренды$
* Если первый сервер, с которым он общался до этого, не отвечает, то через $T_2=0.875 \cdot время-аренды$ клиент пытается обновить адрес у любого другого доступного сервера (используем broadcast)

> [!NOTE]
> Если клиент уже привязался к определенному серверу и его $T_1$ и $T_2$ не закончились, то он игнорирует все оферы.

> [!NOTE]
> Если DHCP-серевов больше одного, то обычно их конфигурируют так, чтобы раздавали адреса из непересекающихся сетевых диапазонов.

### Безопасность

* Нет механизма борьбы с пиратскими DHCP-серверами
* Можем блокировать DHCP-трафик, идущий не от DHCP-серверов
* Фильтрация DHCP-запросов на клиентских портах коммутатора (не больше 1 запроса на порт)