---
title: Структура приложения
description: Как структурировать ваше приложение с модулями ES2015, доставкой кода на клиент и на сервер, и разделением кода множественных приложений.
discourseTopicId: 20187
---

После прочтения этой статьи вы будете знать:

1. Как Meteor приложение соотносится с другими типами приложений в терминах файловой структуры.
2. Как структурировать ваше приложения для больших и маленьких проектов.
3. Как форматировать код и именовать части вашего приложения последовательным и поддерживаемым образом.

<h2 id="meteor-structure">Universal JavaScript</h2>

Метеор это *full-stack* фрэймворк для создания JavaScript приложений. Это значит что Meteor приложения отличаются от большинства приложений тем что содержат код который выполняется на клиенте, внутри web браузера или мобильного приложения Cordova, код, запускаемый на сервере, внутри [Node.js](http://nodejs.org/) контейнера, и _common_ код который запускается и там и там. Инструмент [Meteor build tool](build-tool.html) позволяет вам просто указать какой JavaScript код, включая любые поддерживаемые UI шаблоны, CSS правила, и статические активы, запускать в каждом из окружений (клиентском или серверном) используя комбинации ES2015 `import` и `export` и правила системы сборки Meteor [default file load order](#load-order) .

<h3 id="es2015-modules">модули ES2015</h3>

С версии 1.3, Meteor поставляется с полной поддержкой [модулей ES2015](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/import). Стандартный модуль ES2015 заменяет [CommonJS](http://requirejs.org/docs/commonjs.html) и [AMD](https://github.com/amdjs/amdjs-api), which are commonly used JavaScript module format and loading systems.

В ES2015, вы можете создавать переменные доступные извне файла используя ключевое слово `export` . Чтобы использовать переменные гдето еще, вы должны `import` их используя путь к исходникам. Файлы которые export какие-либо переменные называются "модули", поскольку представляют собой единицу повторно-используемого кода. Явное импортирование модулей и пакетов, которые вы используете поможет вам писать свой код по модульному принципу, что позволяет избежать введения глобальных символов и "действия на расстоянии".

Поскольку это новая фича появившаяся в Meteor 1.3, вы встретите много кода в старом стиле, больше централизованных соглашений вокруг пакетов и приложений объявляющих глобальные symbols. Эта старая система продолжает быть работоспособной, так что чтобы перейти на новую модульную систему код модуля должен быть помещен в папку `imports/` вашего приложения. Мы ожидаем, что будущий выпуск Метеор будет включать все модули по умолчанию для всего кода, потому что это в большей степени совпадает с тем, как разработчики в сообществе JavaScript пишут свой код.

Подробнее о модульной системе вы можете прочесть в [`modules` package README](https://docs.meteor.com/#/full/modules). Этот пакет автоматически включен в каждое Meteor приложение как часть [`ecmascript` meta-package](https://docs.meteor.com/#/full/ecmascript), поэтому большинству приложений не нужно ничего делать, чтобы начать использовать модули сразу.

<h3 id="intro-to-import-export">Введение в использование `import` и `export`</h3>

Meteor позволяет вам `import` в ваше приложение не только JavaScript, но также CSS и HTML для управления загрузкой:

```js
import '../../api/lists/methods.js';  // импорт по относительному пути
import '/imports/startup/client';     // импорт модуля с index.js по абсолютному пути
import './loading.html';              // импорт Blaze compiled HTML по относительному пути
import '/imports/ui/style.css';       // импорт CSS по абсолютному пути
```

> Больше способов импорта стилей, описано в статье [Build System](build-tool.html#css-importing).

Meteor также поддерживает синтаксис стандартнх модулей ES2015 `export` :

```js
export const listRenderHold = LaunchScreen.hold();  // именованный экспорт
export { Todos };                                   // именованный экспорт
export default Lists;                               // дефолтный экспорт
export default new Collection('lists');             // дефолтный экспорт
```

<h3 id="importing-from-packages">Importing from packages</h3>

В Meteor, также просто и straightforward использовать синтаксис `import` для загрузки npm пакетов на клиент или сервер и  access the package's exported symbols as you would with any other module. Вы также можете import из пакетов Meteor Atmosphere , но import должен предваряться `meteor/` для избежания конфликтов с областью имен npm пакета. Например, для импорта `moment` из npm и `HTTP` из Atmosphere:

```js
import moment from 'moment';          // дефолтный импорт из npm
import { HTTP } from 'meteor/http';   // именованный импорт из Atmosphere
```

Подробнее об использовании `imports` в пакетах описано в [Using Packages](using-packages.html) руководства Meteor.

<h3 id="using-require">Использование `require`</h3>

В Meteor, выражение `import` компилируется в синтаксис CommonJS `require`. Тем не менее, в виде соглашения мы рекомендуем использовать синтаксис `import`.

С учетом сказанного, иногда вам может понадобиться `require` напрямую. One notable example is when requiring client or server-only code from a common file. As `import`s must be at the top-level scope, you may not place them within an `if` statement, так что вам нужно будет написать код так:

```js
if (Meteor.isClient) {
  require('./client-only-file.js');
}
```

Заметьте что динамический вызов `require()` (where the name being required может измениться во время выполнения) cannot be analyzed correctly and may result in broken client bundles.

If you need to `require` from an ES2015 module with a `default` export, you can grab the export with `require("package").default`.

Another situation where you'll need to use `require` is in CoffeeScript files. As CS doesn't support the `import` syntax yet, you should use `require`:

```cs
{ FlowRouter } = require 'meteor/kadira:flow-router'
React = require 'react'
```

<h3 id="exporting-from-coffeescript">Exporting with CoffeeScript</h3>

When using CoffeeScript, not only the syntax to import variables is different, but also the export has to be done in a different way. Variables to be exported are put in the `exports` object:

```cs
exports.Lists = ListsCollection 'lists'
```

<h2 id="javascript-structure">Файловая структура</h2>

Для полноценного использования модульной системы и уверенности втом что код запускается только когда мы его запрашиваем мы рекомендуем весь код вашего приложения размещать в папке `imports/`. Это приведет к тому что система сборки приложений Meteor будет включать эти файлы только если они запрашиваются из других файлов с использованием `import` (что также называют "ленивой загрузкой").

Meteor будет загружать все файлы вне папки `imports/` в приложение используя [default file load order](#load-order) rules (также называемой "жадной загрузкой"). Рекомендуется чтобы вы создавали только два жадно загружаемых файла, `client/main.js` и `server/main.js`, для того, чтобы определить явные точки входа для клиента и сервера. Meteor гарантирует что любые файлы в любой папке в папке `server/` будут доступны только на сервере, также и для файлов в любой папке папки `client/`. Это также предотвращает попытки `import` файлов которые должны использоваться только на сервере из любой папки с именем `client/` даже если они вложены в папку `imports/` и наоборот для импорта файлов клиента из папки `server/`.

Эти файлы `main.js` ничего сами не делают, но они должны импортировать какие-то _startup_ модули которые будут работать сразу, на клиенте и сервере соответственно, по загрузке приложения. Эти модули должны выполнять всю необходимую конфигурацию для используемых в приложении пакетов, и импортировать оставшийся код приложения.

<h3 id="example-app-structure">Пример структуры папок</h3>

Взгляните на [Todos example application](https://github.com/meteor/todos), прекрасный пример структуры приложения. Вот структура его папок:

```sh
imports/
  startup/
    client/
      index.js                 # импорт клиентского стартап кода через единственную точку входа index
      routes.js                # настройка всех роутов в приложении
      useraccounts-configuration.js # конфигурирует login шаблоны
    server/
      fixtures.js              # заполняет DB демо-данными при старте
      index.js                 # импорт серверного стартап кода через единственную точку входа index

  api/
    lists/                     # элемент логики домена
      server/
        publications.js        # все list-related публикации
        publications.tests.js  # tests for the list publications
      lists.js                 # объявление коллекции Lists
      lists.tests.js           # tests for the behavior of that collection
      methods.js               # методы относящиеся к lists
      methods.tests.js         # тесты для этих методов

  ui/
    components/                # все повторно-используемые компоненты в приложении
                               # может быть разбито на домены если их много
    layouts/                   # обертка компонентов для представления и поведения
    pages/                     # точки входа для рендеринга используемого роутером

client/
  main.js                      # точка входа клиента, импортирует весь клиентский код

server/
  main.js                      # точка входа сервера импортирует весь серверный код
```

<h3 id="structuring-imports">Structuring imports</h3>

Now that we have placed all files inside the `imports/` directory, let's think about how best to organize our code using modules. It makes sense to put all code that runs when your app starts in an `imports/startup` directory. Another good idea is splitting data and business logic from UI rendering code. We suggest using directories called `imports/api` and `imports/ui` for this logical split.

Within the `imports/api` directory, it's sensible to split the code into directories based on the domain that the code is providing an API for --- typically this corresponds to the collections you've defined in your app. For instance in the Todos example app, we have the `imports/api/lists` and `imports/api/todos` domains. Inside each directory we define the collections, publications and methods used to manipulate the relevant domain data.

> Note: in a larger application, given that the todos themselves are a part of a list, it might make sense to group both of these domains into a single larger "list" module. The Todos example is small enough that we need to separate these only to demonstrate modularity.

Within the `imports/ui` directory it typically makes sense to group files into directories based on the type of UI side code they define, i.e. top level `pages`, wrapping `layouts`, or reusable `components`.

For each module defined above, it makes sense to co-locate the various auxiliary files with the base JavaScript file. For instance, a Blaze UI component should have its template HTML, JavaScript logic, and CSS rules in the same directory. A JavaScript module with some business logic should be co-located with the unit tests for that module.

<h3 id="startup-files">Startup files</h3>

Some of your code isn't going to be a unit of business logic or UI, it's just some setup or configuration code that needs to run in the context of the app when it starts up. In the Todos example app, the `imports/startup/client/useraccounts-configuration.js` file configures the `useraccounts` login templates (see the [Accounts](accounts.html) article for more information about `useraccounts`). The `imports/startup/client/routes.js` configures all of the routes and then imports *all* other code that is required on the client:

```js
import { FlowRouter } from 'meteor/kadira:flow-router';
import { BlazeLayout } from 'meteor/kadira:blaze-layout';
import { AccountsTemplates } from 'meteor/useraccounts:core';

// Import to load these templates
import '../../ui/layouts/app-body.js';
import '../../ui/pages/root-redirector.js';
import '../../ui/pages/lists-show-page.js';
import '../../ui/pages/app-not-found.js';

// Import to override accounts templates
import '../../ui/accounts/accounts-templates.js';

// Below here are the route definitions
```

We then import both of these files in `imports/startup/client/index.js`:

```js
import './useraccounts-configuration.js';
import './routes.js';
```

This makes it easy to then import all the client startup code with a single import as a module from our main eagerly loaded client entry point `client/main.js`:

```js
import '/imports/startup/client';
```

On the server, we use the same technique of importing all the startup code in `imports/startup/server/index.js`:

```js
// This defines a starting set of data to be loaded if the app is loaded with an empty db.
import '../imports/startup/server/fixtures.js';

// This file configures the Accounts package to define the UI of the reset password email.
import '../imports/startup/server/reset-password-email.js';

// Set up some rate limiting and other important security settings.
import '../imports/startup/server/security.js';

// This defines all the collections, publications and methods that the application provides
// as an API to the client.
import '../imports/api/api.js';
```

Our main server entry point `server/main.js` then imports this startup module. You can see that here we don't actually import any variables from these files - we just import them so that they execute in this order.

<h3 id="importing-meteor-globals">Importing Meteor "pseudo-globals"</h3>

For backwards compatibility Meteor 1.3 still provides Meteor's global namespacing for the Meteor core package as well as for other Meteor packages you include in your application. You can also still directly call functions such as [`Meteor.publish`](http://docs.meteor.com/#/full/meteor_publish), as in previous versions of Meteor, without first importing them. However, it is recommended best practice that you first load all the Meteor "pseudo-globals" using the `import { Name } from 'meteor/package'` syntax before using them. For instance:

```js
import { Meteor } from 'meteor/meteor';
import { EJSON } from 'meteor/ejson';
```

<h2 id="load-order">Default file load order</h2>

Even though it is recommended that you write your application to use ES2015 modules and the `imports/` directory, Meteor 1.3 continues to support eager loading of files, using these default load order rules, to provide backwards compatibility with applications written for Meteor 1.2 and earlier. You may combine both eager loading and lazy loading using `import` in a single app. Any import statements are evaluated in the order they are listed in a file when that file is loaded and evaluated using these rules.

There are several load order rules. They are *applied sequentially* to all applicable files in the application, in the priority given below:

1. HTML template files are **always** loaded before everything else
2. Files beginning with `main.` are loaded **last**
3. Files inside **any** `lib/` directory are loaded next
4. Files with deeper paths are loaded next
5. Files are then loaded in alphabetical order of the entire path

```js
  nav.html
  main.html
  client/lib/methods.js
  client/lib/styles.js
  lib/feature/styles.js
  lib/collections.js
  client/feature-y.js
  feature-x.js
  client/main.js
```

For example, the files above are arranged in the correct load order. `main.html` is loaded second because HTML templates are always loaded first, even if it begins with `main.`, since rule 1 has priority over rule 2. However, it will be loaded after `nav.html` because rule 2 has priority over rule 5.

`client/lib/styles.js` and `lib/feature/styles.js` have identical load order up to rule 4; however, since `client` comes before `lib` alphabetically, it will be loaded first.

> You can also use [Meteor.startup](http://docs.meteor.com/#/full/meteor_startup) to control when run code is run on both the server and the client.

<h3 id="special-directories">Special directories</h3>

By default, any JavaScript files in your Meteor application folder are bundled and loaded on both the client and the server. However, the names of the files and directories inside your project can affect their load order, where they are loaded, and some other characteristics. Here is a list of file and directory names that are treated specially by Meteor:

- **imports**

    Any directory named `imports/` is not loaded anywhere and files must be imported using `import`.

- **node_modules**

    Any directory named `node_modules/` is not loaded anywhere. node.js packages installed into `node_modules` directories must be imported using `import` or by using `Npm.depends` in `package.js`.

- **client**

    Any directory named `client/` is not loaded on the server. Similar to wrapping your code in `if (Meteor.isClient) { ... }`. All files loaded on the client are automatically concatenated and minified when in production mode. In development mode, JavaScript and CSS files are not minified, to make debugging easier. CSS files are still combined into a single file for consistency between production and development, because changing the CSS file's URL affects how URLs in it are processed.

    > HTML files in a Meteor application are treated quite a bit differently from a server-side framework.  Meteor scans all the HTML files in your directory for three top-level elements: `<head>`, `<body>`, and `<template>`.  The head and body sections are separately concatenated into a single head and body, which are transmitted to the client on initial page load.

- **server**

    Any directory named `server/` is not loaded on the client. Similar to wrapping your code in `if (Meteor.isServer) { ... }`, except the client never even receives the code. Any sensitive code that you don't want served to the client, such as code containing passwords or authentication mechanisms, should be kept in the `server/` directory.

    Meteor gathers all your JavaScript files, excluding anything under the `client`, `public`, and `private` subdirectories, and loads them into a Node.js server instance. In Meteor, your server code runs in a single thread per request, not in the asynchronous callback style typical of Node.

- **public**

    All files inside a top-level directory called `public/` are served as-is to the client. When referencing these assets, do not include `public/` in the URL, write the URL as if they were all in the top level. For example, reference `public/bg.png` as `<img src='/bg.png' />`. This is the best place for `favicon.ico`, `robots.txt`, and similar files.

- **private**

    All files inside a top-level directory called `private/` are only accessible from server code and can be loaded via the [`Assets`](http://docs.meteor.com/#/full/assets_getText) API. This can be used for private data files and any files that are in your project directory that you don't want to be accessible from the outside.

- **client/compatibility**

    This folder is for compatibility with JavaScript libraries that rely on variables declared with var at the top level being exported as globals. Files in this  directory are executed without being wrapped in a new variable scope. These files are executed before other client-side JavaScript files.

    > It is recommended to use npm for 3rd party JavaScript libraries and use `import` to control when files are loaded.

- **tests**

    Any directory named `tests/` is not loaded anywhere. Use this for any test code you want to run using a test runner outside of [Meteor's built-in test tools](testing.html).

The following directories are also not loaded as part of your app code:

- Files/directories whose names start with a dot, like `.meteor` and `.git`
- `packages/`: Used for local packages
- `cordova-build-override/`: Used for [advanced mobile build customizations](mobile.html#advanced-build)
- `programs`: For legacy reasons

<h3 id="files-outside">Files outside special directories</h3>

All JavaScript files outside special directories are loaded on both the client and the server. Meteor provides the variables [`Meteor.isClient`](http://docs.meteor.com/#/full/meteor_isserver) and [`Meteor.isServer`](http://docs.meteor.com/#/full/meteor_isserver) so that your code can alter its behavior depending on whether it's running on the client or the server.

CSS and HTML files outside special directories are loaded on the client only and cannot be used from server code.

<h2 id="splitting-your-app">Splitting into multiple apps</h2>

If you are writing a sufficiently complex system, there can come a time where it makes sense to split your code up into multiple applications. For example you may want to create a separate application for the administration UI (rather than checking permissions all through the admin part of your site, you can check once), or separate the code for the mobile and desktop versions of your app.

Another very common use case is splitting a worker process away from your main application so that expensive jobs do not impact the user experience of your visitors by locking up a single web server.

There are some advantages of splitting your application in this way:

 - Your client JavaScript bundle can be significantly smaller if you separate out code that a specific type of user will never use.

 - You can deploy the different applications with different scaling setups and secure them differently (for instance you might restrict access to your admin application to users behind a firewall).

 - You can allow different teams at your organization to work on the different applications independently.

However there are some challenges to splitting your code in this way that should be considered before jumping in.

<h3 id="sharing-code">Sharing code</h3>

The primary challenge is properly sharing code between the different applications you are building. The simplest approach to deal with this issue is to simply deploy the *same* application on different web servers, controlling the behavior via different [settings](deployment.html#environment). This approach allows you to easily deploy different versions with different scaling behavior but doesn't enjoy most of the other advantages stated above.

If you want to create Meteor applications with separate code, you'll have some modules that you'd like to share between them. If those modules are something the wider world could use, you should consider [publishing them to a package system](writing-packages.html), either npm or Atmosphere, depending on whether the code is Meteor-specific or otherwise.

If the code is private, or of no interest to others, it typically makes sense to simply include the same module in both applications (you *can* do this with [private npm modules](https://www.npmjs.com/private-modules)). There are several ways to do this:

 - a straightforward approach is simply to include the common code as a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) of both applications.

 - alternatively, if you include both applications in a single repository, you can use symbolic links to include the common module inside both apps.

<h3 id="sharing-data">Sharing data</h3>

Another important consideration is how you'll share the data between your different applications.

The simplest approach is to point both applications at the same `MONGO_URL` and allow both applications to read and write from the database directly. This works well thanks to Meteor's support for reactivity through the database. When one app changes some data in MongoDB, users of any other app connected to the database will see the changes immediately thanks to Meteor's livequery.

However, in some cases it's better to allow one application to be the master and control access to the data for other applications via an API. This can help if you want to deploy the different applications on different schedules and need to be conservative about how the data changes.

The simplest way to provide a server-server API is to use Meteor's built-in DDP protocol directly. This is the same way your Meteor client gets data from your server, but you can also use it to communicate between different applications. You can use [`DDP.connect()`](http://docs.meteor.com/#/full/ddp_connect) to connect from a "client" server to the master server, and then use the connection object returned to make method calls and read from publications.

<h3 id="sharing-accounts">Sharing accounts</h3>

If you have two servers that access the same database and you want authenticated users to make DDP calls across the both of them, you can use the *resume token* set on one connection to login on the other.

If your user has connected to server A, then you can use `DDP.connect()` to open a connection to server B, and pass in server A's resume token to authenticate on server B. As both servers are using the same DB, the same server token will work in both cases. The code to authenticate looks like this:

```js
// This is server A's token as the default `Accounts` points at our server
const token = Accounts._storedLoginToken();

// We create a *second* accounts client pointing at server B
const app2 = DDP.connect('url://of.server.b');
const accounts2 = new AccountsClient({ connection: app2 });

// Now we can login with the token. Further calls to `accounts2` will be authenticated
accounts2.loginWithToken(token);
```

You can see a proof of concept of this architecture in an [example repository](https://github.com/tmeasday/multi-app-accounts).
