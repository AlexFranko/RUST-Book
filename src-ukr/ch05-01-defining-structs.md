## Визначення та Створення Екземпляра Структури

Структури подібні до кортежів, про які ми говорили в підрозділі [“Тип кортеж”][tuples]<!--
ignore --> Розділу 3, бо обидва складаються з кількох пов'язаних значень. Як і у кортежах, частини структур можуть бути різних типів. На відміну від кортежів, у структурі ви називаєте кожен елемент даних, щоб було зрозуміло, що ці значення означають. Завдяки цим іменам структури гнучкіші за кортежі: ви не мусите покладатися на порядок даних, щоб визначати чи отримувати доступ до значень екземпляра.

Для визначення структури, ми вводимо ключове слово `struct` і називаємо всю структуру. Ім'я структури має описувати сенс групування цих елементів даних. Потім, у фігурних дужках, ми визначаємо імена і типи елементів даних, які звуться *полями*. Наприклад, Блок коду 5-1 показує структуру, що зберігає інформацію про обліковий запис користувача.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

<span class="caption">Блок коду 5-1: Визначення структури `User`</span>

Щоб скористатися структурою по визначенню, ми створюємо *екземпляр* цієї структури, визначаючи конкретні значення для кожного поля. Ми створюємо екземпляр, вказуючи назву структури, а потім додаємо фігурні дужки, що містять пари *ключ: значення*, де ключі - це імена полів, а значення - дані, які ми хочемо зберігати в цих полях. Поля не обов'язково вказувати у тому ж порядку, в якому вони були проголошені в структурі. Іншими словами, визначення структури - це загальний шаблон типу, а екземпляри заповнюють цей шаблон конкретними даними, щоб створити значення цього типу. Наприклад, ми можемо проголосити конкретного користувача, як показано в Блоці коду 5-2.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```


<span class="caption">Listing 5-2: Creating an instance of the `User` struct</span>

Щоб отримати конкретне значення зі структури, використовують записом через точку. Якщо ми хочемо отримати адресу електронної пошти користувача, ми можемо написати `user1.email`. Якщо екземпляр є мутабельним, ми можемо змінити значення за допомогою запису через точку і присвоюванням конкретному полю. Блок коду 5-3 показує, як змінити значення поля `email` мутабельного екземпляра `User`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```


<span class="caption">Блок коду 5-3: Зміна значення поля `email` екземпляру `User`</span>

Зверніть увагу, що мутабельним має бути весь екземпляр; Rust не дозволяє позначати лише окремі поля як мутабельні. Як і з будь-яким виразом, ми можемо написати новий екземпляр останнім виразом у тілі функції, щоб неявно повернути цей новий екземпляр.

Блок коду 5-4 демонструє функцію `build_user`, що повертає екземпляр `User` зі встановленими адресою та ім'ям. Поле `active` отримує значення `true`, а `sign_in_count` - значення `1`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```


<span class="caption">Listing 5-4: A `build_user` function that takes an email and username and returns a `User` instance</span>

Має сенс називати аргументи такої функції тими ж іменами, що й імена відповідних полів структури, але необхідність повторювати імена полів `email` та `username` утомлює. Якщо у структурі більше полів, повторення кожного імені дратує ще більше. На щастя, є зручне скорочення!

<!-- Old heading. Do not remove or links may break. -->
<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Скорочена Ініціалізація Полів

Оскільки назви параметрів та полів структури є абсолютно однаковими у Блоці коду 5-4, ми можемо використати синтаксис *скороченої ініціалізації полів* для переписування `build_user` так, щоб він поводився точно так само, але не містив повторення `username` і `email`, як показано у Блоці коду 5-5.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```


<span class="caption">Блок коду 5-5: функція `build_user`, що використовує скорочену ініціалізацію полів, бо параметри `username` і `email` мають такі самі назви, як і поля структури</span>

Ми створюємо новий екземпляр структури `User`, яка має поле з назвою `email`. Ми хочемо встановити значення поля `email` у значення параметра `email` функції `build_user`. Оскільки поле `email` і параметр `email` мають одну назву, можна писати скорочено `email` замість `email: email`.

### Створення Екземплярів Структур з Інших Екземплярів Використовуючи Синтаксис Оновлення

Часто буває корисним створити новий екземпляр структури, що бере більшу частину даних з екземпляра, що вже існує, проте деякі змінює. Ви можете зробити так за допомогою *синтаксису оновлення структури*.

Для початку, Блок коду 5-6 показує, як створити новий екземпляр `User`, що зветься `user2`, без синтаксису оновлення. Ми виставляємо нове значення поля `email`, проте решта полів використовує значення зі структури `user1`, створеної у Блоці коду 5-2.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```


<span class="caption">Блок коду 5-6: Створення нового екземпляру `User` з деякими значеннями з `user1`</span>

Синтаксис оновлення структури дає той самий результат із меншою кількістю коду, як показано у Блоці коду 5-7. Запис `..` позначає, що решта полів, що їх не було явно виставлено, отримають ті значення, що були в заданому екземплярі.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```


<span class="caption">Блок коду 5-7: використання синтаксису оновлення структури для встановлення нового значення `email` у екземплярі `User`, але використовуючи решту значень з `user1`</span>

Код у Блоці коду 5-7 також створює екземпляр `user2`, що має відмінне значення `email`, але має ті ж значення `username`, `active` та `sign_in_count`, що й `user1`. Запис `..user1` має бути останнім, щоб позначити, що решта полів отримають значення з відповідних полів у `user1`, але ми можемо зазначати значення для будь-якої кількості полів у будь-якому порядку, без урахування того, як вони йдуть у визначенні структури.

Зверніть увагу, що синтаксис оновлення структури використовує `=`, як при присвоєнні; це тому, що він переміщує дані, як показано в підрозділі ["Способи взаємодії змінних і даних: переміщення"][move]<!-- ignore --> . У цьому прикладі, ми більше не зможемо використовувати `user1` після створення `user2`, бо `String` з поля `username` структури `user1` було переміщено у `user2`. Якби ми надали `user2` нові значення типу `String` для обох `email` і `username` і таким чином використали тільки значення `active` і `sign_in_count` з `user1`, тоді `user1` все ще був би коректним після створення `user2`. Типи полів `active` і `sign_in_count` реалізовують трейт `Copy`, тому застосовується поведінка, яку ми обговорювали в підрозділі ["Дані в стеку: копіювання"][copy]<!-- ignore --> .

### Використання Структур-Кортежів без Названих Полів для Створення Нових Типів

Rust також підтримує структури, які виглядають схожими на кортежі, що звуться *структури-кортежі* (tuple struct). Структури-кортежі надають значення структурі, бо мають назву, але не мають назв полів, тільки типи. Структури-кортежі корисні, коли ви хочете дати кортежу ім'я і зробити кортеж окремим типом, але називати кожне поле, як у звичайній структурі, буде надто багатослівним чи надмірним.

Щоб визначити структуру-кортеж, треба вказати ключове слово `struct` і ім'я структури, а потім типи в кортежі. Наприклад, ось визначення і приклади застосування двох структур-кортежів, що звуться `Color` і `Point`:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

Зауважте, що значення `black` та `origin` мають різні типи, бо вони є екземплярами різних структур-кортежів. Кожна визначена нами структура має свій власний тип, навіть якщо поля структур мають однакові типи. Наприклад, функція, що приймає параметр типу `Color`, не може прийняти аргументом `Point`, хоча обидва типи складаються з трьох значень `i32`. В іншому ж структури-кортежі поводяться як кортежі: ви можете деструктуризувати їх на окремі шматки, і ви можете використовувати `.` з індексом, щоб отримати доступ до окремого значення.

### Одиничні Структури без Полів

Також можна визначати структури без жодних полів! Вони звуться *одинично-подібні структури* (unit-like struct), бо поводяться аналогічно до `()`, одничного типу, згаданого в підрозділі [“Тип кортеж”][tuples]<!-- ignore --> . Одинично-подібні структури можуть бути корисними в ситуаціях, коли вам потрібно реалізувати трейт на якомусь типі, але у вас немає потреби зберігати якісь дані в цьому типі. Про трейти ми поговоримо в Розділі 10. Ось приклад проголошення та створення одиничної структури під назвою `AlwaysEqual`:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

Щоб визначити `AlwaysEqual`, ми використовуємо ключове слово `struct`, назву структури та крапку з комою. Дужки не потрібні - ані звичайні, ані фігурні! Тоді ми можемо створити екземпляр з `AlwaysEqual` у змінній `subject` у схожий спосіб: використовуючи ім'я, яке ми визначили, без будь-яких - звичайних чи фігурних - дужок. Уявімо, що згодом ми реалізуємо поведінку для цього типу, так що кожен екземпляр `AlwaysEqual` завжди дорівнює будь-якому екземпляру будь-якого іншого типу, скажімо, щоб мати завжди відомий результат для тестування. Нам не потрібні жодні дані для реалізації такої поведінки! У Розділі 10 ви побачите, як визначити трейти і реалізувати їх на будь-якому типі, включно з одинично-подібними структурами.

> ### Володіння Даними Структури
> 
> У структурі `User` з Блоку коду 5-1 ми використовували тип `String`, що має володіння, а не стрічковий слайс `&str`. Це свідомий вибір, бо ми хочемо, щоб екземпляри цієї структури володіли всіма своїми даними і щоб ці дані були коректними, поки структури в цілому коректна.
> 
> Структура також може зберігати посилання на дані, якими володіє хтось інший, але це потребує використання *часу життя*, особливості Rust, що обговорюється у Розділі 10. Час життя гарантує, що дані, на які посилається структура, будуть коректними весь час існування структури. Наприклад, якщо ви спробуєте зберегти посилання у структурі без уточнення часу життя, ось так, то дістанете помилку:
> 
> <span class="filename">Файл: src/main.rs</span>
> 
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
> 
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
> 
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
> 
> Компілятор поскаржиться, що потрібно зазначити час існування:
> 
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
> 
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
> 
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` due to 2 previous errors
> ```
> 
> Ми обговоримо, як це виправити, щоб можна було зберігати посилання у структурах, у Розділі 10, а поки що будемо виправляти подібні помилки за допомогою типів, що володіють своїми даними, на кшталт `String`, замість посилань на кшталт `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
