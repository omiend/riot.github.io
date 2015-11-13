---
layout: ru
title: Структура приложения
---

{% include ru/guide-tabs.html %}

## Инструменты, не политики

Riot включает в себя пользовательские теги, слушатель событий и маршрутизатор. Мы верим, что это - основной набор, достаточный для клиентского приложения.

1. Пользовательские теги для интерфейсов.
2. Слушатель событий для модульности, и
3. Маршрутизатор, который берёт на себя URL и кнопку "назад".


Riot стремится  не применять строгие правила, а, скорее, обеспечить основные инструменты. Такой подход оставляет архитектурные решения для разработчика, но избавляет его от рутины.

Мы также считаем, что компоненты приложения должны быть как можно более минималистичными. Это упрощает понимание API и ускоряет загрузку.

## Наблюдатель

Наблюдатель - это основной инструмент, который отправляет и принимает события. С его помощью реализуется шаблон проектирования, помогающий создать "модульность" без образования зависимостей и связей. Благодаря этому паттерну, большая программа может быть разбита на более мелкие и простые единицы. Модули могут быть добавлены, удалены или изменены, не затрагивая другие части приложения.

Обычной практикой является разделение приложения на ядро и несколько расширений. Ядро посылает события, это может быть что угодно: новый элемент добавляется, существующий элемент удаляется, или что-то загружается с сервера.

При использовании наблюдателя, расширения могут слушать эти события и реагировать на них. Они относятся к ядру так, что ядро не знает про эти модулей. Это называется "слабая связь".

Эти модули могут быть пользовательскими тегами (компоненты пользовательского интерфейса) или модулями без пользовательского интерфейса.

Если ядро и события тщательно продуманы, члены команды могут развивать систему очень самостоятельно, не мешая другим.

[API наблюдателя](/api/observable/)

## Маршрутизация

Маршрутизатор является основным инструментом для работы с URL и кнопкой "назад". Это самая маленькая реализация роутера, которую вы можете найти. Он может сделать следующее:

1. Изменение хэш-части URL (часть после символа #)
2. Отслеживание изменений хеш-части URL
3. Парсинг текущей хэш-части

Вы можете размещать логику маршрутизации где угодно: в пользовательских тегах или в модулях без пользовательского интерфейса.

Практически каждому приложению в браузере необходим маршрутизатор, так как всегда есть URL в адресной строке.

[API Маршрутизатора](/api/route/)

## <a name="modularity"></a> Модульность

Пользовательские теги - это представление, вид вашего приложения. В модульном применении эти теги не должны знать друг друга, они должны быть изолированы. В идеале вы можете использовать один и тот же тег в разных проектах, независимо от макета HTML.

Если два тега знают друг о друге они становятся зависимым и появляется "тесная связь". Эти теги не могут свободно удаляться, не нарушая систему.

Чтобы уменьшить связь, используйте паттерн наблюдателя чтобы прослушивать события. Не обращайтесь друг к другу напрямую. Всё, что вам нужно, это публикация и подписка на события, построенная на `riot.observable` или на аналоге.

Эта система событий может очень простой, или более сложной, такой как Facebook Flux.

### Пример модульного приложения Riot

Это лишь заготовка для авторизации на Riot.

```js
// Login API
var auth = riot.observable()

auth.login = function(params) {
  $.get('/api', params, function(json) {
    auth.trigger('login', json)
  })
}
```
```html
<!-- login view -->
<login>
  <form onsubmit="{ login }">
    <input name="username" type="text" placeholder="username">
    <input name="password" type="password" placeholder="password">
  </form>

  login() {
    opts.login({
      username: this.username.value,
      password: this.password.value
    })
  }

  // любой тег в системе может слушать событие login
  opts.on('login', function() {
    $(body).addClass('logged')
  })
</login>
```

Монтируем приложение в шаблон

```html
<body>
  <login></login>
  <script>riot.mount('login', auth)</script>
</body>
```

На приведенном выше примере теги в системе не должны знать друг о друге, так как они могут просто слушать события и делать то, что им заблагорассудится. Наблюдатель является классическим компонентом для построения модульной архитектуры.