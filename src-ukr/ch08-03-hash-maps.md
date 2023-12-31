## Зберігання Ключів з Асоційованими Значеннями у Хеш-Мапах

Остання з поширених колекцій - це *хеш-мапа*. Тип `HashMap<K, V>` зберігає відображення ключів типу `K` на значення типу `V`, використовуючи *функцію хешування*, яка визначає, як розмістити ці ключі та значення у пам'яті. Багато мов програмування підтримують таку структуру даних, але часто використовують іншу назву, таку як хеш, відображення, хеш-таблиця, словник або асоціативний масив, це тільки декілька назв.

Хеш-мапи корисні, коли ви хочете шукати дані не за індексом, як у векторах, а за допомогою ключа довільного типу. Наприклад, у грі ви можете відстежувати результат кожної команди за допомогою хеш-мапи, в якій кожен ключ - це назва команди, а значення - її рахунок. Маючи назву команди, ви можете отримати її рахунок.

У цьому розділі ми розглянемо базовий API хеш-мап, але у функціях, визначених на `HashMap<K, V>` стандартною бібліотекою, ховається ще багато цікавого. Як завжди, зверніться до документації стандартної бібліотеки для додаткової інформації.

### Створення Нової Хеш-Мапи

Один зі способів створення порожньої хеш-мапи є використання `new` і додавання елементів за допомогою `insert`. У Блоці коду 8-20 ми відстежуємо рахунки двох команд, що називаються *Синя* та *Жовта*. Синя команда починає з 10 очками, а Жовта - з 50.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```


<span class="caption">Блок коду 8-20: Створення нової хеш-мапи і вставлення деяких ключів та значень</span>

Зверніть увагу, що нам, по-перше, треба зробити `use` для `HashMap` з розділу колекцій стандартної бібліотеки. З трьох наших загальних колекцій ця використовується найрідше, тож вона не включена до функціонала, який автоматично додається до області видимості у прелюдії. Хеш-мапи також мають меншу підтримку від стандартної бібліотеки; наприклад, немає вбудованого макросу для їх побудови.

Так само як і вектори, хеш-мапи зберігають свої дані у купі. Цей `HashMap` має ключі типу `String` і значення типу `i32`. Як і вектори, хеш-мапи є однорідними: усі ключі мають бути одного типу, і всі значення мають бути одного типу.

### Доступ до Значень Хеш-Мапи

Ми можемо отримати значення з хеш-мапи, надавши її ключ методу `get`, як показано у Блоці коду 8-21.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```


<span class="caption">Блок коду 8-21: Доступ до рахунку Синьої команди, що зберігається у хеш-мапі</span>

Тут `score` буде мати значення, пов'язане з Синьою командою, і результат буде `10`. Метод `get` повертає `Option<&V>`; якщо у хеш-мапі для цього ключа немає відповідного значення, `get` поверне `None`. Ця програма обробляє `Option` викликом `copied`, отримуючи `Option<i32>`, а не `Option<&i32>`, а тоді `unwrap_or`, щоб встановити `score` у нуль, якщо `scores` не має запису для цього ключа.

Ми можемо ітерувати по кожній парі ключ/значення в хеш-мапі подібно до того, як ми це робимо з векторами, використовуючи цикл `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Цей код виведе в консолі кожну пару в довільному порядку:

```text
Yellow: 50
Blue: 10
```

### Хеш-Мапи та Володіння

Для типів, що реалізують трейт `Copy`, таких як `i32`, значення копіюються у хеш-мапу. Для значень, які мають володіння, таких як `String`, значення будуть переміщені і хешмапа буде володіти цими значеннями, як це показано у Блоці коду 8-22.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```


<span class="caption">Блок коду 8-22: Демонстрація, що хеш-мапа володіє ключем та значенням після їх вставлення</span>

Ми не можемо використовувати змінні `field_name` і `field_value` після того, як вони були переміщені до хеш-мапи за допомогою виклику `insert`.

Якщо ми вставляємо посилання на значення до хешмапи, ці значення не будуть переміщені до хешмапи. Значення, на які вказують посилання, мають бути коректними щонайменше стільки ж, скільки існує хешмапа. Ми поговоримо про це докладніше у підрозділі ["Перевірка коректності посилань за допомогою часів існування"]()<!-- ignore --> Розділу 10.

### Оновлення Хеш-Мапи

Хоча кількість пар ключів і значень зростає, кожен унікальний ключ може мати тільки одне значення, пов’язане з ним, в кожен момент (але не навпаки: наприклад, команда Синя і Жовта могли мати значення 10, збережене в хешмапі `scores`).

Коли ви хочете змінити дані в хешмапі, вам необхідно вирішити, як обробляти випадок, коли ключ уже має присвоєне значення. Ви можете замінити старе значення на нове значення, повністю проігнорувавши старе значення. Ви можете зберегти старе значення і проігнорувати нове значення, і лише коли ключ *не має* значення, додавати нове. Або ж ви можете поєднати старе значення і нове значення. Подивімося, як зробити кожен варіант!

#### Перезапис Значення

Якщо ми вставляємо ключ і значення до хешмапи і тоді вставляємо той самий ключ із іншим значенням, то значення, асоційоване з цим ключем, буде замінено. Попри те, що код у Блоці коду 8-23 викликає `insert` двічі, хешмапа міститиме лише одну пару ключ/значення, оскільки ми обидва рази вставляємо значення для ключа Синьої команди.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```


<span class="caption">Блок коду 8-23: заміна значення, збереженого з певним ключем</span>

Цей код виведе `{"Blue": 25}`. Початкове значення `10` було перезаписане.

<!-- Old headings. Do not remove or links may break. -->
<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Додавання Ключа та Значення Тільки якщо Ключ Відсутній

Доволі поширено перевіряти, чи певний ключ уже присутній у хешмапі зі значенням, а тоді якщо ключ існує, то наявне значення має залишатися таким, яким воно є. Якщо ж ключ відсутній, вставити його і значення для нього.

Хешмапи мають спеціальне API для цього, що зветься `entry`, яке приймає параметром ключ, який ви хочете перевірити. Значення, що повертається з методу `entry` - це енум, що зветься `Entry`, який представляє значення, що може існувати або не існувати. Скажімо, ми хочемо перевірити, чи ключ для Жовтої команди має пов'язане з ним значення. Як ні, ми хочемо вставити значення 50, і те саме для Синьої команди. За допомогою API `entry`, код стає схожим на Блок коду 8-24.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```


<span class="caption">Блок коду 8-24: використання методу `entry` для вставляння лише якщо ключ ще не має відповідного значення</span>

Метод `or_insert` для `Entry` за визначенням повертає мутабельне посилання на відповідний ключ `Entry`, якщо ключ існує, а як ні, то вставляє параметр як нове значення для цього ключа і повертає мутабельне посилання на нове значення. Ця техніка набагато чистіша, ніж написання логіки самостійно і ще, крім того, краще працює з borrow checker.

Запуск коду у Блоці коду 8-24 надрукує `{"Yellow": 50, "Blue": 10}`. Перший виклик `entry` вставить ключ для Жовтої команди зі значенням 50, бо Жовта команда ще не має свого значення. Другий виклик `entry` не змінить хешмапу, бо Синя команда вже має значення 10.

#### Оновлення Значення на Основі Старого Значення

Інший поширений сценарій використання хешмап - пошук значення ключа і оновлення його на основі старого значення. Наприклад, Блок коду 8-25 показує код, який підраховує, скільки разів кожне слово з'являється в певному тексті. Ми використовуємо хешмапу з ключами - словами і збільшуємо значення, щоб відстежувати, скільки разів ми бачили це слово. Якщо ми зустрічаємо слово уперше, то спершу вставляємо значення 0.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```


<span class="caption">Блок коду 8-25: підрахунок кількості слів за допомогою хешмапи слів, що зберігає слова і кількість</span>

Цей код виведе `{"world": 2, "hello": 1, "wonderful": 1}`. Ви можете побачити ті ж пари ключів/значень, виведені в іншому порядку: згадайте з підрозділу ["Доступ до значень у хешмапі"][access]<!-- ignore --> , що ітерування по хешмапі відбувається у довільному порядку.

Метод `split_whitespace` повертає ітератор по підслайсах, розділених пробілами, значення у `text`. Метод `or_insert` повертає мутабельне посилання (`&mut V`) на значення для вказаного ключа. Тут ми зберігаємо це мутабельне посилання у змінній `count`, тож для того, щоб присвоїти цьому значенню, нам необхідно спочатку розіменувати `count` за допомогою зірочки (`*`). Мутабельне посилання виходить з області видимості в кінці циклу `for`, тож всі ці зміни є безпечними та дозволеними правилами позичання.

### Функції Хешування

За замовчуванням, `HashMap` використовує функцію хешування під назвою *SipHash*, яка забезпечує стійкість до атак на відмову в обслуговуванні (Denial of Service, DoS) з використанням хеш-таблиць [^siphash]<!-- ignore -->. Це не найшвидший з доступних алгоритмів хешування, але покращення безпеки, отримане з падінням продуктивності, того варте. Якщо ви профілюєте свій код і виявите, що функція хешування за замовчуванням є надто повільною для ваших потреб, ви можете перейти на іншу функцію, вказавши інший хешер. *Хешер* - це тип, який реалізує трейт `BuildHasher`. Ми поговоримо про трейти і як їх реалізовувати у Розділі 10. Вам не обов'язково потрібно реалізувати власний хеш з нуля; [crates.io](https://crates.io/)<!-- ignore --> містить бібліотеки, надані іншими користувачами Rust, що забезпечують реалізацію багатьох поширених алгоритмів хешування.

## Підсумок

Вектори, стрічки та хешмапи забезпечать значну частину функціональності, необхідної для програм під час зберігання, доступу і зміни даних. Ось деякі вправи, які ви вже маєте бути в змозі виконати:

* Дано список цілих чисел; використайте вектор і поверніть медіану (значення посередині після сортування) та моду (значення, яке зустрічається найчастіше; тут стане в пригоді хешмапа) цього списку.
* Перетворіть рядки на "поросячу латину". Перший приголосний кожного слова переноситься в кінець слова і додається "ay", так що "first" стає "irst-fay." До слів, що починаються на голосну, натомість додається в кінці “hay” (“apple” стає “apple-hay”). Не забувайте, що використовується кодування UTF-8!
* За допомогою хешмапи і векторів створіть текстовий інтерфейс, що дозволить користувачеві додати імена співробітників у відділ компанії. Наприклад, “Add Sally to Engineering” чи “Add Amir to Sales.” Тоді дайте користувачеві можливість отримати список усіх людей у відділі чи всіх людей у компанії по відділах, відсортованих за алфавітом.

Документація API стандартної бібліотеки описує методи, які мають вектори, стрічки і хешпами, які будуть корисними для цих вправ!

Ми переходимо до складніших програм, у яких операції можуть зазнавати невдачу, тому зараз ідеальний час для обговорення обробки помилок. Цим ми й займемося!
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

[access]: #accessing-values-in-a-hash-map
