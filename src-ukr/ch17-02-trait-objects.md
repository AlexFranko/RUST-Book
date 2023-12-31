## Використання Трейт-Об'єктів, які Допускають Значення Різних Типів

У розділі 8 ми казали, що одним з обмежень векторів є те, що вони можуть зберігати елементи тільки одного типу. Ми обійшли цю проблему в Роздруку 8-9, де ми визначили енум `SpreadsheetCell` який мав варіанти для зберігання цілих чисел, чисел з рухомою комою й тексту. Це означало, що ми мали змогу зберігати різні типи даних в кожній комірці та все одно мати вектор, який представляє рядок комірок. Це дуже гарне рішення коли наші взаємозамінні елементи є типами з фіксованим набором, відомим на етапі компіляції.

Проте іноді ми хочемо, щоб користувач нашої бібліотеки зміг розширити набір типів, які є допустимими в конкретній ситуації. Щоб показати, як ми можемо досягти цього, ми створимо приклад інструменту з графічним інтерфейсом користувача (GUI), який ітерує список елементів, викликаючи метод `draw` на кожному з них, щоб намалювати його на екрані — це поширена техніка для GUI інструментів. Ми створимо бібліотечний крейт `gui`, який містить структуру бібліотеки GUI. Цей крейт може містити деякі готові до використання типи, наприклад тип `Button` чи `TextField`. Крім того, користувачі крейту `gui` можуть захотіти створити свої власні типи, які можуть бути намальовані: наприклад, один програміст може додати тип `Image`, а інший - `SelectBox`.

Ми не будемо реалізовувати повноцінну GUI бібліотеку для цього прикладу, але покажемо як її частини будуть поєднуватися. Коли ми пишемо бібліотеку, ми не можемо знати та визначити всі типи, які можуть захотіти створити інші програмісти. Але ми знаємо що `gui` повинен відстежувати багато значень різного типу та викликати метод `draw` кожного з цих по-різному типізованому значень. Крейт не повинен знати, що станеться, коли ми викличемо метод `draw`, просто у значення буде доступний для виклику такий метод.

Для того, щоб зробити це на мові, в якій є наслідування, ми можемо визначити клас під назвою `Component`, який має метод `draw`. Інші класи, такі як `Button`, `Image`, та `SelectBox`, можуть успадкуватися від `Component` й таким чином успадкувати метод `draw`. Кожен з них може перевизначити реалізацію методу `draw`, щоб описати власну поведінку, але фреймворк може розглядати всі типи ніби вони є екземпляром `Component` та міг би викликати їх метод `draw`. Але, оскільки Rust не має механізму успадкування, нам потрібен інший спосіб структурувати `gui` бібліотеку, щоб дозволити користувачам розширювати її новими типами.

### Визначення Трейту для Спільної Поведінки

Для реалізації поведінки, яку ми хочемо мати в `gui`, визначимо трейт під назвою `Draw`, який буде містити один метод `draw`. Тоді ми можемо визначити вектор, який приймає *трейт-об'єкт*. Трейт-об'єкт вказує як на екземпляр типу, що реалізує вказаний нами трейт, так і на внутрішню таблицю, що використовується для пошуку методів трейту вказаного типу під час виконання. Ми створюємо трейт-об'єкт в такому порядку: використовуємо якийсь вид вказівнику, наприклад посилання `&` або розумний вказівник `Box<T>`, потім ключове слово `dyn` й відповідний трейт. (Ми будемо говорити чому трейт-об'єкти повинні використовувати вказівник у Розділі 19 в секції [“Dynamically Sized Types and the `Sized` Trait.”][dynamically-sized]<!-- ignore -->) Ми можемо використовувати трейт-об'єкт замість узагальненого або конкретного типу. Де б ми не використовували трейт-об'єкт, система типів Rust забезпечить, що під час компіляції будь-яке значення використане у цьому контексті буде реалізовувати трейт трейт-об'єкту. Отже, ми не повинні знати всі можливі типи під час компіляції.

Ми нагадували, що в Rust ми не називаємо структури та енуми "об'єктами", щоб розрізняти їх з об'єктами в інших мовах програмування. У структурі або енумі, дані в полях структури та поведінка в блоку `impl` розділені, тоді як в інших мовах вони об'єднанні в один концепт, який часто називають об'єкт. Однак, трейт-об'єкти *є* більше схожими на об'єкти в інших мовах, в тому сенсі що вони об'єднують дані та поведінку. Але трейт-об'єкти відрізняються від традиційних об'єктів у том, що ми не можемо додати дані до трейт-об'єкту. Трейт-об'єкти загалом не настільки корисні як об'єкти в інших мовах програмування: їх конкретна ціль - забезпечити абстракцію через загальну поведінку.

Блок коду 17-3 показує, як визначити трейт під назвою `Draw` з одним методом `draw`:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-03/src/lib.rs}}
```

<span class="caption">Блок коду 17-3: Визначення трейту `Draw`</span>

Цей синтаксис має бути знайомим після наших дискусій про те, як визначати трейти в розділі 10. Далі йде новий синтаксис: у Роздруку 17-4 визначена структура під назвою `Screen`, яка містить вектор з ім'ям `components`. Цей вектор має тип `Box<dyn Draw>`, який і є трейт-об'єктом; це позначення будь-якого типу всередині `Box`, який реалізує трейт `Draw`.

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-04/src/lib.rs:here}}
```


<span class="caption">Блок коду 17-4: Визначення структури `Screen` з полем `components`, яке є вектором трейт-об'єктів, що реалізують трейт `Draw`</span>

У структурі `Screen` ми визначено метод під назвою `run`, який буде викликати метод `draw` кожного елементу вектора `components`, як показано у Блоці коду 17-5:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-05/src/lib.rs:here}}
```


<span class="caption">Блок коду 17-5: Метод `run` в структурі `Screen`, який викликає метод `draw` кожного компоненту</span>

Це працює інакше ніж визначення структури, яка використовує параметр узагальненого типу з обмеженнями трейтів. Узагальнений параметр типу може бути замінений тільки одним конкретним типом, тоді як трейт-об'єкти дозволяють декільком конкретним типам бути на його місці під час виконання. Наприклад, визначимо структуру `Screen` використовуючи узагальнені типи та обмеження трейту в Блоці коду 17-6:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-06/src/lib.rs:here}}
```


<span class="caption">Блок коду 17-6: Альтернативна реалізація структури `Screen` та її методу `run` за допомогою узагальнених типів та обмежень трейту</span>

Це обмежує екземпляр `Screen` до одного з двох можливих варіантів: наповнений лише компонентами типу `Button`, або лише компонентами типу `TextField`. Якщо у вас коли-небудь будуть тільки однорідні колекції, використання узагальнених типів та обмежень трейту краще, оскільки визначення будуть мономорфізованими під час компіляції для використання з конкретними типами.

З іншого боку, за допомогою методу, який використовує трейт-об'єкт, один екземпляр `Screen` може містити `Vec<T>`, який містить `Box<Button>`, так само як і `Box<TextField>`. Нумо подивімось як це працює, а потім поговоримо про вплив на швидкодію під час виконання.

### Реалізація Трейту

Тепер ми додамо деякі типи, які реалізуються трейт `Draw`. Запровадимо тип `Button`. Знову ж таки, фактична реалізація бібліотеки GUI виходить за межі цієї книги, тому тіло методу `draw` не буде мати ніякої корисної реалізації. Щоб уявити, як може виглядати така реалізація, структура `Button` може мати поля для `width`, `height`, та `label`, як показано в Роздруку 17-7:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-07/src/lib.rs:here}}
```


<span class="caption">Блок коду 17-7: Структура `Button`, яка реалізує трейт `Draw`</span>

Поля `width`, `height`, та `label` структури `Button` будуть відрізнятися від полів інших компонентів; наприклад, тип `TextField` міг би мати такі самі поля плюс поле `placeholder`. Кожен тип, який ми хочемо намалювати на екрані, буде реалізовувати трейт `Draw`, але буде мати інший код методу `draw` для визначення того, як саме малювати конкретний тип, наприклад `Button` в цьому прикладі (без фактичного коду GUI, який виходить за межі цього розділу). Наприклад, тип `Button` може мати додаткові блоки `impl`, що містять методи, які визначають що станеться, коли користувач натисне на кнопку. Такі методи не застосовуватимуться до таких типів, як `TextField`.

Якщо користувач нашої бібліотеки вирішить реалізувати структуру `SelectBox`, яка має `width`, `height`, та `options` поля, він реалізує також і трейт `Draw` для структури `SelectBox`, як показано в Роздруку 17-8:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-08/src/main.rs:here}}
```


<span class="caption">Блок коду 17-8: Інший крейт використовує `gui` та реалізує трейт `Draw` для структури `SelectBox`</span>

Тепер користувач нашої бібліотеки може написати свою `main` функцію, щоб створити екземпляр `Screen`. До екземпляра `Screen`, він може додати `SelectBox` та `Button`, розмістивши кожен з них у `Box<T>`, щоб він став трейт-об'єктом. Потім він може викликати метод `run` в екземпляра `Screen`, який викличе метод `draw` для кожного компонента. Роздрук 17-9 показує цю реалізацію:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-09/src/main.rs:here}}
```


<span class="caption">Блок коду 17-9: Використання трейт-об'єктів для зберігання значень різних типів, які реалізують той самий трейт</span>

Коли ми писали бібліотеку, ми не знали, що хтось може додати тип `SelectBox`, але наша реалізація `Screen` мала змогу працювати з новим типом та малювати його, тому що `SelectBox` реалізує трейт `Draw`, що означає, що він реалізує метод `draw`.

Ця концепція, яка стосується тільки повідомлень на які значення відповідає, на відміну від конкретного типу в значення, аналогічна концепції *duck typing* (качкової типізації) у динамічно типізованих мовах: якщо хтось ходить як качка та крякає як качка, то він - качка! У реалізації методу `run` структури `Screen` в Роздруку 17-5, `run` не повинен знати конкретний тип кожного компонента. Він не перевіряє чи є компонент екземпляром `Button` чи `SelectBox`, він просто викликає метод `draw` компоненту. Вказавши `Box<dyn Draw>` як тип значень у вектору `components`, ми визначили `Screen` для значень у яких ми можемо викликати метод `draw`.

Перевага використання трейт-об'єктів і системи типів Rust для написання коду подібного до коду з використанням качкової типізації полягає в тому, що нам ніколи не потрібно перевіряти, чи реалізує значення певний метод під час виконання або турбуватися про отримання помилок якщо значення не реалізує метод. Rust не буде компілювати наш код, якщо значення не реалізують трейт потрібного трейт-об'єкту.

Наприклад, Блок коду 17-10 показує, що станеться, якщо ми спробуємо створити `Screen` з `String` як компонент:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-10/src/main.rs}}
```


<span class="caption">Роздрук 17-10: Спроба використати тип, який не реалізує трейт трейт-об'єкту</span>

Ми отримаємо помилку, тому що `String` не реалізує трейт `Draw`:

```console
{{#include ../listings/ch17-oop/listing-17-10/output.txt}}
```

Ця помилка дає зрозуміти, що або ми передаємо в компонент `Screen` щось, що ми не збиралися передавати, і тоді ми повинні передати інший тип, або ми повинні реалізувати трейт `Draw` у типу `String`, щоб `Screen` міг викликати `draw` у нього.

### Трейт-Об'єкти Виконують Динамічну Диспетчеризацію

Нагадаємо, у секції [“Швидкодія коду з узагальненими типами”]()<!-- ignore --> розділу 10 обговорюється процес мономорфізації, який виконується компілятором, коли ми використовуємо обмеження трейтів для узагальнених типів: компілятор генерує конкретні типи, які ми використовуємо замість параметра узагальненого типу. Код, отриманий в результаті мономорфізації, виконує *статичну диспетчеризацію*, коли компілятор знає який метод ви викликаєте під час компіляції. Це протилежний підхід до *динамічної диспетчеризації*, коли компілятор не може сказати під час компіляції, який метод ви викликаєте. У випадках динамічної диспетчеризації компілятор генерує код, який під час виконання визначає, який метод необхідно викликати.

Коли ми використовуємо трейт-об'єкти, Rust має використовувати динамічну диспетчеризацію. Компілятор не знає всі типи, які можуть бути використані з кодом, який використовує трейт-об'єкти, тому він не знає, який метод реалізований для якого типу при виклику. Замість цього, під час виконання, Rust використовує вказівники всередині трейт-об'єкту, щоб дізнатися який метод викликати. Такий пошук провокує додаткові витрати під час виконання, які не потребуються під час статичної диспетчеризації. Динамічна диспетчеризація також не дозволяє компілятору обрати вбудовування коду метода, що робить неможливим деякі оптимізації. Однак, ми отримали додаткову гнучкість у коді, який ми написали у Роздруку 17-5, і змогли підтримати у Роздруку 17-9, так що це - компроміс для розгляду.
ch10-01-syntax.html#performance-of-code-using-generics

[dynamically-sized]: ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait
