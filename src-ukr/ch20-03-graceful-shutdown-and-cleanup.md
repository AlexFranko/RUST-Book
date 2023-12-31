## Плавне Вимкнення та Очищення

Код у Блоці коду 20-20 відповідає на запити асинхронно, використовуючи пулу потоків, так, як ми й планували. Ми отримуємо деякі попередження про поля `workers`, `id`, і `threads`, які ми не використовуємо напряму, що нагадує нам ми нічого не очищуємо. Коли ми використовуємо менш елегантний метод зупинки основного потоку за допомогою <span
class="keystroke">ctrl-c</span>, решта потоків також негайно зупиняється, навіть якщо ми посередині обробки запиту.

Наступним ми реалізуємо трейт `Drop`, щоб викликати `join` для кожного з потоків у пулі, щоб вони могли завершити запити, над якими працюють, перед закриттям. Потім ми реалізуємо спосіб повідомити потокам, що вони мають припинити отримувати нові запити і вимкнутися. Щоб побачити цей код у дії, ми змінимо наш сервер, щоб він приймав лише два запити перед плавним вимиканням пулу потоків.

### Реалізація Трейту `Drop` для `ThreadPool`

Почнімо з реалізації `Drop` для нашого пулу потоків. Коли пул очищується, всі потоки повинні приєднатися до основного, щоб переконатися, що вони завершили роботу. Блок коду 20-22 показує першу спробу реалізації `Drop`; цей код ще не зовсім працює.

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/listing-20-22/src/lib.rs:here}}
```


<span class="caption">Блок коду 20-22: приєднання всіх потоків, коли пул потоків виходить з області видимості</span>

Спершу ми в циклі перебираємо всі `workers` в пулі потоків. Для цього ми використовуємо `&mut`, бо `self` є мутабельним посиланням, і ми також мусимо мати можливість змінити `worker`. Для кожного worker ми виводимо повідомлення, що цей конкретний worker вимикається, а потім викликаємо `join` для потоку цього worker. Якщо виклик `join` буде невдалим, ми використовуємо `unwrap`, щоб Rust запанікував і грубо припинив роботу.

Ось помилка, яку ми отримуємо, коли ми компілюємо цей код:

```console
{{#include ../listings/ch20-web-server/listing-20-22/output.txt}}
```

Ця помилка каже нам, що ми не можемо викликати `join`, бо ми маємо лише мутабельне позичання кожного `worker`, а `join` перебирає володіння своїм аргументом. Щоб вирішити цю проблему, ми маємо перемістити потік з екземпляра `Worker`, що володіє цим `thread`, щоб `join` міг поглинути потік. Ми робили це у Блоці коду 17-15: якщо `Worker` містить `Option<thread::JoinHandle<()>>`, ми можемо викликати метод `take` для `Option`, щоб перемістити значення з варіанту `Some` і залишити варіант `None` на своєму місці. Іншими словами, `Worker`, який працює, матиме варіант `Some` у `thread`, і коли ми хочемо очистити `Worker`, то ми замінимо `Some` на `None`, тож `Worker` не матиме потоку для виконання.

Отож ми знаємо, що хочемо оновити визначення `Worker` наступним чином:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-04-update-worker-definition/src/lib.rs:here}}
```

Тепер покладемося на компілятор, щоб знайти інші місця, які потрібно змінити. При перевірці цього коду ми отримуємо дві помилки:

```console
{{#include ../listings/ch20-web-server/no-listing-04-update-worker-definition/output.txt}}
```

Розберімося з другою помилкою, що вказує, на код в кінці `Worker::new`; ми маємо обгорнути значення e `thread` у `Some`, коли ми створюємо нового `Worker`. Зробіть такі зміни, щоб виправити цю помилку:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-web-server/no-listing-05-fix-worker-new/src/lib.rs:here}}
```

Перша помилка знаходиться у нашій реалізації `Drop`. Ми вже згадували раніше, що збиралися викликати `take` для значення `Option`, щоб перемістити `thread` з `worker`. Наступні зміни роблять це:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/no-listing-06-fix-threadpool-drop/src/lib.rs:here}}
```

Як обговорювалося в Розділі 17, метод `take` для `Option` забирає варіант `Some` і залишає `None` на своєму місці. Ми використовуємо `if let` для деструктуризації `Some` і отримуємо потік; тоді ми викликаємо `join` для потоку. Якщо потік worker вже `None`, то ми знаємо, що цей worker уже очистив свій потік, тож в цьому випадку нічого не відбудеться.

### Подавання Сигналів Потокам Припинити Чекати на Завдання

Після всіх змін, які ми зробили, наш код компілюється без попереджень. Однак, погана новина в тому, що цей код ще не функціонує так, як ми цього хочемо. Причина в логіці в замиканнях, що виконуються в потоках екземплярів `Worker`: наразі, ми викликаємо `join`, але це не вимикає потоки, бо їхні цикли `loop` постійно шукають завдання. Якщо ми спробуємо очистити `ThreadPool` з нашою поточною реалізацією `drop`, головний потік заблокується назавжди, чекаючи на завершення першого потоку.

Щоб розв'язати цю проблему нам знадобиться зміна в реалізації `drop` для `ThreadPool`, а також зміна в циклі `Worker`.

Спершу ми змінимо реалізацію `drop` для `ThreadPool`, щоб явно очищати `sender` перед очікуванням на завершення потоків. Блок коду 20-23 показує зміни до `ThreadPool` для явного очищення `sender`. Ми використовуємо ту ж техніку `Option` і `take`, якою вже користувалися з потоком, щоб перемістити `sender` зі `ThreadPool`:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch20-web-server/listing-20-23/src/lib.rs:here}}
```


<span class="caption">Блок коду 20-23: явне очищення `sender` перед приєднанням потоків worker</span>

Очищення `sender` закриває канал, що позначає, що більше повідомлень не буде надіслано. Коли це стається, всі виклики до `recv`, зроблені worker в нескінченому циклі повернуть помилку. У Блоці коду 20-24 ми змінюємо цикл у `Worker` на для плавного виходу з циклу в цьому випадку, тобто потоки завершаться, коли реалізація `drop` для `ThreadPool` викличе для них `join`.

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/listing-20-24/src/lib.rs:here}}
```


<span class="caption">Блок коду 20-24: явне переривання циклу, коли `recv` повертає помилку</span>

Щоб побачити цей код в дії, змінімо `main`, щоб приймати лише два запити перед плавним вимиканням сервера, як показано в Блоці коду 20-25.

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/listing-20-25/src/main.rs:here}}
```


<span class="caption">Блок коду 20-25: вимикання сервера виходом з циклу після обслуговування двох запитів</span>

Вам би не сподобалося, якби справжній вебсервер вимикався після обслуговування лише двох запитів. Це код демонструє, що плавне вимикання і очищення працюють, як слід.

Метод `take`, визначений в трейті `Iterator`, обмежує ітерації максимум першими двома елементами. `ThreadPool` вийде з області видимості в кінці `main` і запуститься реалізація `drop`.

Запустіть сервер за допомогою `cargo run`і зробіть три запити. Третій запит призведе до помилки, і у вашому терміналі ви маєте побачити виведення, схоже на це:


<!-- manual-regeneration
cd listings/ch20-web-server/listing-20-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Ви можете побачити іншу послідовність worker і виведених повідомлень. Ми бачимо цих повідомлень, як працює цей код; worker 0 і 3 отримали два перші запити. Сервер припинив приймати з'єднання після другого з'єднання, і реалізація `Drop` для `ThreadPool` почала виконуватися до того, як worker 3 розпочав роботу. Очищення `sender` від'єднує всіх workers і наказує їм вимкнутися. Кожен worker виводить повідомлення при роз'єднанні, і тоді пул потоків викликає `join`, чекаючи, доки кожен worker завершиться.

Зверніть увагу на один цікавий аспект конкретно цього виконання: `ThreadPool` очистив `sender`, і до того, як будь-який worker отримав помилку, ми намагалися приєднати worker 0. Worker 0 ще не отримав помилку від `recv`, тому основний потік заблокувався, чекаючи на завершення worker 0. Тим часом worker 3 отримав завдання, а потім всі потоки отримали помилку. Коли worker 0 завершив роботу, основний потік зачекав на завершення роботи решти workers. У цей момент, вони всі вийшли з циклів і зупинилися.

Вітання! Ми завершили наш проєкт; у нас є примітивний вебсервер, який використовує пул потоків для асинхронних відповідей. Ми можемо виконати плавне вимикання нашого сервера, яке очищує потоки в пулі.

Ось повний код для звірки:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/main.rs}}
```

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch20-web-server/no-listing-07-final-code/src/lib.rs}}
```

Ми могли б зробити ще більше! Якщо ви хочете продовжити покращувати цей проєкт, ось деякі ідеї:

* Додати більше документації до `ThreadPool` та його публічних методів.
* Додати тести для функціонала бібліотеки.
* Замінити виклики `unwrap` надійнішою обробкою помилок.
* Використати `ThreadPool` для виконання інших завдань, крім обслуговування вебзапитів.
* Знайти крейт пулу потоків на [crates.io](https://crates.io/) та реалізувати аналогічний вебсервер за допомогою цього крейта. Тоді порівняти його API і надійність з пулом потоків, реалізованим нами.

## Підсумок

Хороша робота! Ви дісталися кінця книги! Ми хочемо подякувати вам за те, що приєдналися до нас у цій подорожі по Rust. Тепер ви готові реалізовувати свої власні проєкти Rust і допомагати іншим людям у їхніх проєктах. Не забувайте, що існує гостинна спільноту Растацеанців, які залюбки допоможуть вам з будь-якими викликами, з якими ви стикаєтеся у подорож по Rust.
