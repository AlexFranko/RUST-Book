## Використання Потоків для Одночасного Виконання Коду

В більшості сучасних операційних систем, код виконуємої програми запускається в середині *процесу*, а операційна система керує багатьма процесами одночасно. Всередині програми ви можете мати незалежні частини, які виконуються одночасно. Фунціонал, який виконує такі незалежні частини називають *потоками* або *тредами*. Наприклад, вебсервер може використовувати декілька потоків і таким чином мати змогу обробляти більше одного запиту одночасно.

Розділення обчислень між декількома потоками для одночасного виконання кількох завдань може покращити швидкість виконання програми, але також підвищує складність програми. Оскільки потоки можуть виконуватись одночасно, немає жодної гарантії стосовно порядку виконання частин коду в різних потоках. Це може призвести до наступних проблем:

* стан гонитви, за якого потоки отримують доступ до даних або ресурсів в довільному порядку
* дедлоки, коли два потоки чекають один одного, перешкоджаючи продовженню виконанняя обох потоків
* помилки, що виникають виключно за певних обставин і які важко відворити та надійно виправити

Rust намагається помʼякшити негативні наслідки використання потоків, але програмування в багатопоточному контексті все ще вимагає ретельного обдумування та потребує структуру коду, відмінну від програм, що виконуються в одному потоці.

Мови програмування імплементують потоки декількома різними способами, а багато операційних систем надають API, який мова може використовувати для створення нових потоків. Стандартна бібліотека Rust використовує модель імплементації потоку *1:1*, за якої програма використовує один потік операційної системи на один потік, що використовує мова. Існують крейти, котрі імплементують інші моделі потоків, що йдуть на інші компроміси порівняно з моделью 1:1.

### Створення Нового Потоку зі `spawn`

Для того, щоб створити новий потік, ми викликаємо функцію `thread::spawn` і передаємо їй замикання (ми вже говорили про них в Розділі 13), що містить в собі код, який ми хочемо запустити всередині нового потоку. Приклад у Лістінгу 16-1 виводить на екран деякий текст з основного потоку, а також інший текст з нового потоку:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```


<span class="caption">Блок коду 16-1: Створення нового потоку для виводу на екран деяку стрічку, поки основний потік виводить іншу стрічку</span>

Зверніть увагу, що коли основний потік Rust програми завершується, всі створені потоки припиняють існувати, незалежно від того завершили вони своє виконання чи ні. Вивід цієї програми може дещо відрізнятись від запуску до запуску, але виглядатиме приблизно так:


<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Виклики `thread::sleep` змушують потік припинити виконання на короткий період часу, дозволяючи виконувати інший потік. Ймовірно, потоки будуть виконуватись по черзі, проте це не гарантовано, а залежить від того, як ваша операційна система планує виконання потоків. Цього разу, основний потік вивів на екран стрічку першим, незважаючи на те, що print statement в новому потоці зʼявляється у коді раніше. Незважаючи на те, що ми запрограмували новостворений потік виводити екран стрічки доти, поки `i` не дорівнюватиме 9, він вивів лише 5 стрічок до завершення основного потоку.

Якщо ви запускаєте цей код і бачите лише вивід з основного потоку або ж не бачите жодного оверлепу, спробуйте збільшити діапазон чисел, щоб створити для операційної системи більше умов для переключання між потоками.

### Очікування Завершення Виконання Всіх Потоків з Використанням Дескрипторів `join`

Код в Блоці коду 16-1 не лише передчасно зупиняє створений потік в більшості випадків через завершення основного потоку, але оскільки гарантії щодо порядку виконання потоків відсутні, ми не можемо гарантувати, що створені потоки взагалі будуть виконуватись!

Ми можемо вирішити проблему, коли створений потік не виконується або ж передчасно завершує виконання, зберігаючи значення, що повертає `thread::spawn` в змінну. `thread::spawn` повертає значення, що має тип `JoinHandle`. `JoinHandle` - це owned value (значення, яким володіють), котре при виклику на ньому методу `join`, чекатиме завершення виконання потоку. Блок коду 16-2 демонструє як використовувати `JoinHandle` потоку, котрий ми створили в блоці коду 16-1, а також викликати `join` щоб пересвідчитись, що новостворений потік закінчує виконання раніше, ніж `main`:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```


<span class="caption">Блок коду 16-2: Збереження `JoinHandle` з `thread::spawn` щоб гарантувати виконання потоку до завершення</span>

Виклик `join` на обробнику (хендлері) блокує потік, що наразі виконується аж до моменту, коли потік, представлений обробником, не завершиться. *Блокування* потоку означає, що такий потік не може виконуватися або ж завершитись. Оскільки ми помістили виклик `join` після циклу `for` в основному потоці, виконання Блоку коду 16-2 має вивести на екран щось схоже на наступне:


<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Два потоки продовжують виконуватися по черзі, але основний потік чекає через виклик `handle.join()` і не завершується, доки не завершиться створений потік.

Однак давайте поглянемо, що трапиться, якщо замість цього розмістити `handle.join()` перед циклом `for` всередині `main`, як тут:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

Основний потік чекатиме на завершення виконання створеного треду і лише після цього виконуватиме цикл `for`, тому вивід більше не буде чергуватись, як показано тут:


<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Дрібні деталі, такі як-от місце виклику `join`, можуть впливати на те чи будуть ваші потоки виконуватись одночасно.

### Використання `move` Замикань із Потоками

Ми будемо часто використовувати ключове слово `move` разом з замиканнями, переданими всередину `thread::spawn`, оскільки замикання тоді почне володіти значеннями, які вона використовує з оточуючого контексту (середовища), таким чином передаючи володіння значеннями з одного потоку в інший. У підрозділі ["Захоплення посилань або переміщення володіння"][capture]<!-- ignore
--> Розділу 13, ми розглядали 

`move` в контексті замикань. Зараз же, ми сконцентруємось більше на взаємодії між `move` та `thread::spawn`.

Зверніть увагу, в Блоці коду 16-1 замикання, яке ми передаємо в `thread::spawn` не приймає жодних аргументів: ми не використовуємо жодних даних з основного потоку в коді створеного потоку. Для того, щоб використовувати дані з основного потоку в створеному потоці, замикання створеного потоку має захопити (capture) потрібні значення. Блок коду 16-3 показує спробу створити вектор в основному потоці, а потім використати його в створеному. Однак, це не запрацює одразу, як ви зможете незабаром пересвідчитись.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```


<span class="caption">Блок коду 16-3: Спроба використати вектор, створений основним потоком, в іншому потоці</span>

Замикання використовує `v`, отже, воно захопить `v` і зробить його частиною контексту замикання. Оскільки `thread::spawn` виконує замикання в новому потоці, ми повинні мати доступ до `v` всередині нового потоку. Однак, коли ми скомпілюємо цей приклад, ми отримаємо наступну помилку:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust *здогадується* як захопити `v`, і саме тому `println!` потребує тільки посилання на `v`, а замикання намагається запозичити `v`. Однак є проблема: Rust не може здогадатись як довго потік буде виконуватись, отже, Rust не знає чи буде посилання на `v` завжди валідним (valid).

Блок коду 16-4 показує випадок, який, швидше за все, матиме невалідне посилання на `v`:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```


<span class="caption">Блок коду 16-4: Потік з замиканням, що намагається захопити (capture) посилання на `v` з основного потоку, який видаляє `v`</span>

Якщо Rust дозволить нам виконати цей код, існує ймовірність, що створений потік буде негайно переведений в бекграунд (background) і не буде виконуватись взагалі. Створений потік має посилання на `v` всередині себе, але основний потік негайно викидає (drops) `v`, використовуючи функцію `drop`, котру ми розглядали у Розділі 15. Потім, коли створений потік починає виконуватись, `v` більше невалідний, тому посилання на нього також невалідне. О ні!

Щоб виправити помилку компілятора в блоці коду 16-3, ми можем скористатись порадою, яку надає повідомлення про помилку:


<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Додаючи ключове слово `move` перед замиканням, ми змушуємо замикання взяти володіння над значеннями, котрі воно використовує, не дозволяючи Rust робити припущення стосовно позичання (borrowing) значень. Модифікація до Блоку коду 16-3, показана в Блоці коду 16-5 скомпілюється і виконуватиметься так, як ми задумали:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```


<span class="caption">Блок коду 16-5: Використання ключового слова `move`, щоб примусити замикання взяти володіння над значеннями, які воно використовує</span>

В нас може виникнути спокуса спробувати той самий підхід, щоб виправити код у Блоці коду 16-4, де основний потік викликав `drop`, використовуючи замикання з `move`. Однак, це не спрацює, оскільки те, що Блок коду 16-4 намагається зробити, заборонено з іншої причини. Якщо ми додамо `move` до замикання, ми перенемістили `v` в контекст замикання, тому ми більше не можемо викликати `drop` на ньому в основному потоці. Замість цього ми отримаємо помилку компіляції:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Правила володіння Rust знову нас врятували! Ми отримали помлку в Блоці коду 16-3, оскільки Rust був консервативним і позичив лише `v` для потоку, а отже, основний потік міг теоретично зробити посилання, що використовувалось у створеному потоці, невалідним. Сказавши Rust передати володіння `v` створеному потоку, ми гарантуємо Rust, що основний потік більше не використовуватиме `v`. Якщо ми змінимо Блок коду 16-4 таким же чином, то ми порушимо правила володіння, коли намагатимемось використати `v` в основному потоці. Ключове слово `move` бере гору над правилами володіння, що Rust використовує за замовчуванням; таким чином не ми не порушуємо правила володіння.

Маючи базове розуміння потоків і API потоків (прикладний програмний інтерфейс потоків), давайте поглянемо що ми можемо *робити* з потоками.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
