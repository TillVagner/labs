# ФИТ НГУ, курс ООП, весенний семестр 2024

## Сетевое программирование, сериализация

### 1

Напишите программу для общения через Internet.

Программа должна состоять из двух частей: сервер и клиент.

Сервер стартует в качестве отдельного приложения на определенном порту (задано в конфигурации).

Клиент в виде приложения на Swing подсоединяется к серверу по имени сервера и номеру порта.

Клиент должен правильно обрабатывать обыв соединения с сервером и переподключаться, при переподключении нужно уточнить какие новые сообщения были во время отсутствия связи и получать только их.

### 2

Минимальные возможности чата:

- каждый участник чата имеет собственный ник, который указывается при присоединению к серверу.
- можно посмотреть список участников чата.
- можно послать сообщение в чат (всем участникам).
- клиент показывает все сообщения, которые отправили в чат с момента подключения + некоторое число, отправленных до; список сообщений обновляется в онлайне.
- клиент отображает такие события как: подключение нового человека в чат и уход человека из \* чата. Сервер должен корректно понимать ситуацию отключения клиента от чата (по таймауту).
  сервер должен логгировать все события, которые происходят на его стороне (включается/отключается в конфигурационном файле).
- чат работает через TCP/IP протокол.

### 3

Необходимо создать 2 версии клиента/сервера.

- Первый вариант сериализацию/десериализацию Java-объектов для посылки/приема сообщений,
- второй - использует XML сообщения(расширенная версия).

### 4

Клиент и сервер должны поддерживать стандартный протокол для XML варианта.
Это необходимо для возможности общение между клиентами, созданными разными учениками.

Протокол описан ниже. Расширения протокола приветствуются, например можно добавить, чтобы пользователь мог выбрать цвет сообщений.

Вначале XML сообщения идут 4 байта (Java int) с его длиной. То есть сначала читаются первые 4 байта и узнается длина оставшегося сообщения (в байтах).
Затем считывается само сообщение и далее обрабатывается как XML документ.

### 5

Рекомендуется использовать следующие техники:

- Сервер слушает порт с помощью класса java.net.ServerSocket
- Клиент подсоединяется к серверу с помощью класса java.net.Socket
- XML сообщение читать с помощью DOM parser:
  DocumentBuilderFactory.newInstance().newDocumentBuilder().parse()
  Сериализация/десериализация объекта выполняется через классы ObjectInputStream и ObjectOutputStream

## Минимальный протокол

Минимальный протокол взаимодействия для XML сообщений (расширения приветствуются):

### Регистрация

Запрос:

```xml
<command name=”login”>
  <name>USER_NAME</name>
  <type>CHAT_CLIENT_NAME</type>
</command>
```

Ответ:

Server error answer

```xml
<error>
  <message>REASON</message>
</error>
```

```xml
Server success answer
<success>
  <session>UNIQUE_SESSION_ID</session>
</success>
```

### Запрос списка пользователей чата

request:

```xml
<command name=”list”>
  <session>UNIQUE_SESSION_ID</session>
</command>
```

Server error answer

```xml

<error>
  <message>REASON</message>
</error>
```

Server success answer

```xml
<success>
  <listusers>
    <user>
      <name>USER_1</name>
      <type>CHAT_CLIENT_1</type>
    </user>
    …
    <user>
      <name>USER_N</name>
      <type>CHAT_CLIENT_N</type>
    </user>
  </listusers>
</success>
```

### Сообщение от клиента серверу

```xml
<command name=”message”>
  <message>MESSAGE</message>
  <session>UNIQUE_SESSION_ID</session>
</command>
```

Server error answer

```xml

<error>
  <message>REASON</message>
</error>
```

Server success answer

```xml
<success>
</success>
```

### Сообщение от сервера клиенту

Сообщения

```xml
<event name="message">
  <message>MESSAGE</message>
  <name>CHAT_NAME_FROM</name>
</event>
```

Новый клиент

```xml
<event name=”userlogin”>
  <name>USER_NAME</name>
</event>
```

Пользователь покинул чат

```xml
<event name=”userlogout”>
  <name>USER_NAME</name>
</event>
```

### Отключение

```xml
<command name=”logout”>
  <session>UNIQUE_SESSION_ID</session>
</command>
```

Server error answer

```xml
<error>
  <message>REASON</message>
</error>
```

Server success answer

```xml
<success>
</success>
```

## Тонкости реализации

Для того, чтобы обеспечить однопотопную неблокирующую обработку сразу нескольких клиентов вам придется воспользоваться библиотекой `java.nio.channels`.

- ServerSocketChannel - для прослушивания в неблокирующем режиме.
- SocketChannel - для чтения данных в неблокирующем режиме.
- Selector - для одновременного получения каналов данных от нескольких клиентов и чтения данных из этих каналов (используется в связке с ServerSocketChannel и SocketChannel).