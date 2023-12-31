# Вступ

> Примітка: Ця редакція книжки така ж сама, як [Мова програмування Rust][nsprust], доступна у форматі друку та електронної книжки у [No Starch Press][nsp].

Вітаємо вас у *Мові програмування Rust*, вступній книзі про Rust. Мова програмування Rust допоможе вам писати швидше та надійніше програмне забезпечення. Ергономіка високого рівня та контроль низького рівня часто є несумісними у дизайні мови програмування; Rust кидає виклик цій суперечності. Завдяки балансу потужних технічних можливостей та великого досвіду розробників Rust надає вам можливість контролювати деталі низького рівня (наприклад, використання пам’яті) без будь-яких клопотів, традиційно пов’язаних із таким контролем.

## Для Кого Призначено Rust

Rust ідеально підходить багатьом людям з різних причин. Розгляньмо кілька найважливіших груп.

### Команди Розробників

Rust показав себе ефективним інструментом співпраці серед великих команд розробників з різним рівнем знання системного програмування. Низькорівневий код схильний до різних специфічних помилок, які в більшості інших мов програмування можна знайти лише завдяки глибокому тестуванню та ретельному перегляду коду досвідченими розробниками. У Rust компілятор виконує роль воротаря, відмовляючись компілювати код із цими невловними помилками, включно з помилками конкурентності. Працюючи разом із компілятором, команда може ефективніше витратити свій час, зосереджуючись на програмній логіці, а не на виловлюванні помилок.

Rust також привносить сучасні засоби розробники у світ системного програмування:

* Cargo, менеджер залежностей та інструмент побудови, включений у комплект, робить додавання, компіляцію та управління залежностями безболісним і послідовним в усій екосистемі Rust.
* Інструмент форматування Rustfmt забезпечує єдиний стиль кодування у різних розробників.
* Rust Language Server надає інтегрованому середовищу розробки (IDE) інтеграцію для завершення коду та вбудованих повідомлень про помилки.

Використовуючи ці та інші інструменти в екосистемі Rust, розробники можуть бути такими продуктивними під час написання коду системного рівня.

### Студенти

Rust призначена для студентів та зацікавлених у вивченні системних концепцій. Використовуючи Rust, багато людей вивчили такі теми, як розробка операційних систем. Доброзичлива спільнота із задоволенням відповідає на питання студентів. Різними заходами, такими, як створення цієї книжки, команди Rust хочуть зробити системні концепції більш доступними для більшої кількості людей, особливо для програмістів-новачків.

### Компанії

Сотні компаній, великих і малих, використовують Rust у виробництві для багатьох різних завдань, таких як інструменти командного рядка, вебслужби, інструменти DevOps, вбудовані пристрої, аудіо- та відеоаналіз та перекодування, криптовалюти, біоінформатика, пошукові системи, застосунки "Інтернету речей", машинне навчання та навіть суттєві частини веббраузера Firefox.

### Розробники Відкритого Коду

Rust - для людей, які хочуть розбудовувати мову програмування Rust, спільноту, інструменти розробника та бібліотеки. Ми дуже хотіли б, щоб ви внесли свій внесок у мову Rust.

### Люди, які Прагнуть Швидкості та Стабільності

Rust призначений для людей, які вимагають від мови швидкості та стабільності. Під швидкістю ми маємо на увазі як швидкість, з якою код Rust може виконуватися, так і швидкість, з якою Rust дозволяє вам писати програми. Перевірки компілятора Rust забезпечують стабільність під час розширення функціонала та рефакторинга. Це відрізняється від крихкого застарілого коду мовами без цих перевірок, який розробники часто бояться змінювати. Прагнучи до абстракцій з нульовою вартістю, конструкцій вищого рівня, що компілюються в код нижчого рівня, так само швидкий, як і написаний вручну, Rust намагається зробити так, щоб безпечний код був швидким кодом.

Мова Rust сподівається підтримати й багатьох інших користувачів; тут згадані лише деякі з найбільш зацікавлених учасників. Загалом, найвищою метою Rust є усунення компромісів, на які програмісти йшли десятиліттями, забезпечуючи безпеку *та* продуктивність, швидкість *та* ергономічність. Спробуйте Rust і побачите, що його вибір вам підходить.

## Для Кого Призначена Ця Книга

Ця книга передбачає, що ви вже писали код іншою мовою програмування, але не робить жодних припущень щодо того, якою саме. Ми спробували зробити матеріал широко доступним для тих, хто має найрізноманітніший досвід програмування. Ми не витрачаємо багато часу на розмови про те, *що* таке програмування або як думати про нього. Якщо ви зовсім ще новачок у програмуванні, вам краще буде прочитати книгу, яка спеціально написана для вступу до програмування.

## Як Користуватися Цією Книгою

Загалом ця книжка передбачає, що ви читатимете її послідовно з початку до кінця. Пізніші розділи спираються на концепції, роз'яснені у попередніх розділах, а початкові розділи можуть не надто заглиблюватися в деталі конкретної теми; ми зазвичай повертаємося до таких тем у наступних розділах.

У цій книзі ви знайдете два типи розділів: розділи про концепції та розділи про проєкти. У розділах про концепції ви дізнаєтесь про різні боки Rust. У розділах проєктів ми разом писатимемо невеликі програми, застосовуючи те, чому ви навчилися. Розділи 2, 12 та 20 - це розділи про проєкти; решта - це розділи про концепції.

Розділ 1 пояснює, як встановити Rust, як написати програму “Hello, world!” і як користуватися Cargo, менеджером пакунків і інструментом побудови Rust. Розділ 2 є практичним вступом до написання програми на Rust, де ви створите гру з відгадуванням чисел. Тут ми висвітлюємо концепції на високому рівні, а пізніші розділи нададуть додаткові подробиці. Якщо ви хочете одразу зануритися в мову, Розділ 2 дає добрий початок. Розділ 3 охоплює функціонал Rust, схожих на аналогічний в інших мовах програмування, а в Розділі 4 ви дізнаєтеся про систему володіння Rust. Якщо ви особливо скрупульозний учень і вважаєте за краще вивчити кожну деталь, перш ніж переходити до наступної, можливо, ви захочете пропустити Розділ 2 і перейти прямо до Розділу 3, повернувшись до Розділу 2, коли захочете застосувати вивчене у проєкті.

У Розділі 5 обговорюються структури та методи, а в Розділі 6 - енуми, вирази `match` та керівні конструкції `if let`. Структури та енуми використовуються для створення власних типів у Rust.

У Розділі 7 ви дізнаєтесь про систему модулів Rust та про правила доступу для організації вашого коду та його публічного програмного інтерфейсу (API). У Розділі 8 розглядаються деякі узагальнені структури даних - колекції, які надає стандартна бібліотека, такі як вектори, рядки та геш-таблиці. Розділ 9 досліджує філософію та техніки обробки помилок Rust.

Розділ 10 заглиблюється в узагальнене програмування, трейти та часи існування, що надає вам змогу визначати код, застосований до різних типів. У Розділі 11 йдеться про тестування, яке є необхідним для забезпечення правильної логіки вашої програми навіть за гарантій безпеки Rust. У Розділі 12 ми створимо власну реалізацію підмножини функціонала інструмента командного рядка `grep`, який шукає заданий текст у файлах. Для цього ми використаємо багато концепцій, обговорених у попередніх розділах.

Розділ 13 досліджує замикання та ітератори - особливості Rust, які походять з функціональних мов програмування. У Розділі 14 ми детальніше розглянемо Cargo і поговоримо про кращі практики спільного використання ваших бібліотек. Розділ 15 обговорює розумні вказівники, надані стандартною бібліотекою, та трейти, що забезпечують їхній функціонал.

У Розділі 16 ми розглянемо різні моделі конкурентного програмування та поговоримо про те, як Rust допомагає вам програмувати декілька потоків без остраху. У Розділі 17 розглядається порівняння ідіом Rust із об'єктноорієнтованими принципами програмування, які вам можуть бути знайомі.

Розділ 18 - це посібник зі шаблонів та зіставлення з шаблонами, які є потужними способами вираження ідей у ​​програмах Rust. Розділ 19 містить вінегрет із просунутих цікавих тем, включно з небезпечним Rust, макросами та різним про час існування, трейти, типи, функції та замикання.

In Chapter 20, we’ll complete a project in which we’ll implement a low-level multithreaded web server!

Нарешті, додатки містять корисну інформацію про мову у більш довідковому форматі. Додаток А охоплює ключові слова Rust, Додаток B охоплює оператори та символи Rust, Додаток C охоплює трейти, що їх можна успадковувати, надані стандартною бібліотекою, Додаток D охоплює деякі корисні інструменти розробки, а Додаток E пояснює видання Rust. У Додатку F ви можете знайти переклади книги, а в Додатку G ми висвітлюємо, як робиться Rust, і що таке нічний Rust.

Немає неправильного способу прочитати цю книгу: якщо ви хочете пропустити щось - вперед! Можливо, вам доведеться повернутися до попередніх розділів, якщо ви відчуєте, що заплуталися. Але робіть все, що вам підходить.

<span id="ferris"></span>

Важливою складовою у навчанні Rust є навчання читати повідомлення про помилки, які відображає компілятор: вони спрямовують вас до робочого коду. Відтак, ми наведемо багато прикладів, які не компілюються, разом із повідомленням про помилку, яке компілятор покаже вам у кожній ситуації. Знайте, що якщо ви введете та запустите випадковий приклад, він може не скомпілюватись! Не забудьте прочитати навколишній текст, щоб побачити, чи приклад, який ви намагаєтесь запустити, мав на меті помилку. Ferris також допоможе вам розрізнити код, що не має працювати:

| Ферріс                                                                                                                                          | Значення                                        |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ферріс зі знаком питання" />                            | Цей код не компілюється!                        |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ферріс розводить руками" />                                       | Цей код призводить до паніки!                   |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ферріс з однією піднятою клешнею знизує плечима" /> | Цей код не робить того, що від нього очікували. |

У більшій частині ситуацій, ми направимо вас до правильної версії коду, що не компілюється.

## Початковий Код

Файли з початковим кодом, з яких створено цю книжку, можна знайти на [GitHub][book].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

[book]: https://github.com/rust-lang/book/tree/main/src
