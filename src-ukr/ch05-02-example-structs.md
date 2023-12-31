## Приклад Програми з Використанням Структур

Щоб зрозуміти, де можна використовувати структури, напишімо програму, що обчислює площу прямокутника. Почнемо з окремих змінних, а потім рефакторизуємо її так, щоб вона використовувала структури.

За допомогою Cargo створімо двійковий проєкт програми, що зветься *rectangles*, яка прийматиме ширину і висоту прямокутника в пікселях і обчислюватиме його площу. Блок коду 5-8 показує коротку очевидну програму, що робить саме те, що треба, у *src/main.rs* нашого проєкту.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```


<span class="caption">Блок коду 5-8: Обчислення площі прямокутника, заданого шириною та висотою в окремих змінних</span>

Тепер запустимо програму командою `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

This code succeeds in figuring out the area of the rectangle by calling the `area` function with each dimension, but we can do more to make this code clear and readable.

Проблема в цьому коді очевидна, якщо поглянути на сигнатуру функції `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Функція `area` має обчислювати площу одного прямокутника, але функція, яку ми написали, приймає два параметри, і з коду зовсім не ясно, що ці параметри пов'язані. Для кращої читаності та керованості буде краще згрупувати ширину і висоту разом. Ми вже обговорювали один зі способів, як це зробити, у підрозділі ["Тип кортеж"][the-tuple-type]<!-- ignore --> Розділу 3: за допомогою кортежів.

### Рефакторинг з Кортежами

Блок коду 5-9 показує версію нашої програми із кортежами.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```


<span class="caption">Блок коду 5-9: Ширина та висота прямокутника задані кортежем</span>

З одного боку, ця програма краща. Кортежі додають трохи структурованості, і тепер ми передаємо лише один аргумент. Але з іншого боку ця версія менш зрозуміла: кортежі не мають назв для своїх елементів, тому тепер доводиться індексувати частини кортежу, що робить наші обчислення менш очевидними.

Не має значення, якщо ми переплутаємо ширину і висоту при обчисленні площі, але якщо ми захочемо намалювати прямокутник на екрані, це матиме значення! Нам доведеться пам'ятати, що `ширина` має індекс `0` у кортежі, а `висота` має індекс `1`. Для когось іще буде ще складніше розібратися в цьому і пам'ятати, якщо він буде використовувати наш код. Оскільки ми не показали сенс наших даних у коді, тепер легше припускатися помилок.

### Рефакторинг зі Структурами: Додаємо Більшого Сенсу

Ми використовуємо структури, щоб додати сенс за допомогою "ярликів" до даних. Ми можемо перетворити наш кортеж на тип даних з іменами як для цілого, так і для частин, як показано в Блоці коду 5-10.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

<span class="caption">Блок коду 5-10: визначення структури `Rectangle`</span>

Тут ми визначили структуру і назвали її `Rectangle`. Всередині фігурних дужок ми визначили поля `width` та `height`, обидва типу `u32`. Далі в `main` ми створюємо конкретний екземпляр `Rectangle` з шириною `30` і висотою `50`.

Наша функція `area` тепер має визначення з одним параметром, який ми назвали `rectangle`, тип якого - немутабельне позичення екземпляра структури `Rectangle`. Як ми вже казали в Розділі 4, ми можемо позичити структуру замість перебирати володіння ним. Таким чином `main` зберігає володіння і може продовжувати використовувати `rect1`, тому ми застосовуємо `&` у сигнатурі функції та при її виклику.

Функція `area` звертається до полів `width` та `height` екземпляру `Rectangle` (зверніть увагу, що доступ до полів позиченого екземпляру структури не переміщує значення полів, ось чому ви часто бачитимете позичання структур). Сигнатура функції `area` тепер каже саме те, що ми мали на увазі: обчислити площу `Rectangle` за допомогою полів `width` та `height`. Це сповіщає, що ширина і висота пов'язані одна з іншою, і дає змістовні імена значенням замість індексів кортежу `0` та `1`. Це виграш для ясності.

### Додаємо Корисний Функціонал Похідними Трейтами

Було б непогано мати змогу виводити екземпляр нашого `Rectangle` при зневадженні програми та бачити значення його полів. Блок коду 5-11 намагається вжити [макрос `println!`][println]<!-- ignore --> так само як це було в попередніх розділах. Але цей код не працює.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```


<span class="caption">Listing 5-11: Attempting to print a `Rectangle` instance</span>

Якщо скомпілювати цей код, ми дістанемо помилку із головним повідомленням:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Макрос `println!` може виконувати багато різних видів форматувань, і за замовчанням фігурні дужки кажуть `println!` використати форматування, відоме як `Display`: вивести те, що призначене для читання кінцевим споживачем. Примітивні типи, з яким ми досі стикалися, реалізують `Display` за замовчанням, оскільки є лише один спосіб, яким можна показати `1` чи якийсь інший примітивний тип користувачу. Але зі структурами вже не настільки очевидно, як `println!` має форматувати вивід, оскільки є багато можливостей виведення: потрібні коми чи ні? Чи треба виводити фігурні дужки? Чи всі поля слід показувати? Через цю невизначеність, Rust не намагається відгадати, чого ми хочемо, і структури не мають підготовленої реалізації `Display`, яку можна було б використати у `println!` за допомогою заовнювача `{}`.

Якщо ми подивимося помилки далі, то знайдемо цю корисну примітку:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Спробуймо це зробити! Виклик макросу `println!` тепер виглядає так: `println!("rect1 = {:?}", rect1);`. Додавання специфікатора `:?` у фігурні дужки каже `println!`, що ми хочемо використати формат виведення, що зветься `Debug`. Трейт `Debug` дозволяє вивести нашу структуру у спосіб, зручний для розробників, щоб дивитися її значення під час зневадження коду.

Скомпілюймо змінений код. Трясця! Все одно помилка:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Але знову компілятор дає нам корисну примітку:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust *має* функціонал для виведення інформації для зневадження, та нам доведеться увімкнути його у явний спосіб, щоб зробити доступним для нашої структури. Щоб зробити це, додамо зовнішній атрибут `#[derive(Debug)]` прямо перед визначенням структури, як показано в Блоці коду 5-12.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```


<span class="caption">Блок коду 5-12: Додавання атрибута для успадкування трейта `Debug` і виведення екземпляра `Rectangle` за допомогою форматування для зневадження</span>

Коли ми знову запустимо програму, то не отримаємо жодної помилки й побачимо таке:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Чудово! Це не найкрасивіший вивід, але він показує значення всіх полів цього екземпляру, що точно допоможе при зневадженні. Коли у нас будуть більші структури, корисно мати зручніший для читання вивід; в цих випадках, ми можемо використати `{:#?}` замість `{:?}` у стрічці `println!`. Якщо скористатися стилем `{:#?}` у цьому прикладі, вивід виглядатиме так:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Інший спосіб вивести значення у форматі `Debug` - скористатися [макросом `dbg!`][dbg]<!-- ignore -->, який перебирає володіння виразом (на відміну від `println!`, що приймає посилання), виводить файл і номер рядка, де був у вашому коді викликаний макрос `dbg!` і обчислене значення виразу, і повертає володіння значенням.

> Примітка: виклик макроса `dbg!` виводить до стандартного потоку помилок у консолі (`stderr`), на відміну від `println!`, що виводить до стандартного потоку виводу консолі (`stdout`). Ми поговоримо більше про `stderr` і `stdout` у підрозділі ["Виведення повідомлень про помилки до стандартного потоку помилок замість стандартного вихідного потоку" Розділу 12][err]<!-- ignore -->.

Here’s an example where we’re interested in the value that gets assigned to the `width` field, as well as the value of the whole struct in `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Ми можемо написати `dbg!` навколо виразу `30 * scale` і, оскільки `dbg!` повертає володіння виразу, поле `with` отримає це саме значення, як ніби й не було виклику `dbg!`. Ми не хочемо, щоб `dbg!` перебирав володіння `rect1`, так що ми використовуємо посилання на `rect1` при наступному виклику. Ось як виглядає те, що виводить цей приклад:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Ми бачимо що перший фрагмент був виведений з рядка 10 *src/main.rs*, де ми зневаджуємо вираз `30 * scale`, а його обчислене значення `60` (реалізація форматування `Debug` для цілих чисел виводить самі їхні значення). Виклик `dbg!` у рядку 14 *src/main. s* виводить значення `&rect1`, яке дорівнює структурі `Rectangle`. Виведення використовує покращене форматування `Debug` для типу `Rectangle`. Макрос `dbg!` може бути дійсно корисним, коли ви намагаєтеся розібратися, що робить ваш код!

На додачу до трейту `Debug`, Rust надає нам ряд трейтів, що можна використовувати з атрибутом `derive`, які можуть додати корисну поведінку до наших власних типів. Ці трейти та їхня поведінка перераховані в [Додатку C][app-c]<!--
ignore -->. Ми розглянемо, як реалізувати ці трейти з кастомізованою поведінкою і як створювати свої власні трейти в Розділі 10. Також існує багато атрибутів, відмінних від 

`derive`; для отримання додаткової інформації дивіться [розділ "Атрибути" Довідника Rust][attributes].

Функція `area` дуже конкретна: вона розраховує лише площу прямокутників. Було б корисно прив'язати цю поведінку до нашої структури `Rectangle`, оскільки вона не буде працювати з жодним іншим типом. Подивімося, як ми можемо продовжувати рефакторизовувати цей код, перетворивши функцію `area` на *метод* `area`, визначений на нашому типі `Rectangle`.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
