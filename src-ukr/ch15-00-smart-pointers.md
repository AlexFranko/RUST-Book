# Розумні Вказівники

*Вказівник* - загальна концепція для змінної, що містить адресу в пам'яті. Ця адреса посилається, або "вказує", на певну інформацію. Найбільш розповсюджений вид вказівника в Rust - це посилання, яке ви вивчили в Розділі 4. Посилання позначені символом `&` та позичають значення, на яке вказують. Вони не мають інших можливостей, крім посилання на дані, та не мають накладних витрат.

*Розумні вказівники*, з іншої сторони, є структурами даних що поводять себе як вказівник, але також мають додаткові метадані та можливості. Концепція розумних вказівників не унікальна для Rust: розумні вказівники виникли в C++ та також існують в інших мовах програмування. Rust має різні розумні вказівники, визначені стандартною бібліотекою, які надають додатковий функціонал, крім наданого посиланнями. Для роз'яснення основної концепції ми подивимось на різні приклади розумних вказівників, включно з типом розумних вказівників, що *підраховує посилання*. Цей вказівник дозволяє даним мати декілька володільців завдяки відстежуванню кількості володільців. Коли володільців не залишається, інформація буде видалена.

Оскільки Rust має власну концепцію володіння та позичання, є додаткова різниця між посиланнями та розумними вказівниками: посилання лише позичають значення, але в багатьох випадках розумні вказівники *володіють* значенням, на яке вказують.

Хоча ми й не називати їх так на той момент, ми вже зустрілись з деякими розумними вказівниками в цій книзі, включно зі `String` та `Ve<T>` у Розділі 8. Обидва типи вважаються розумними вказівниками, оскільки вони володіють деякою пам'яттю і дозволяють вам маніпулювати нею. Вони також мають метадані та додаткові можливості чи гарантії. `String`, наприклад, зберігає свою місткість як метадані та має додаткову спроможність гарантувати, що його дані будуть валідним UTF-8.

Розумні вказівники зазвичай реалізуються за допомогою структур. На відміну від звичайних структур, розумні вказівники реалізують трейти `Deref` та `Drop`. Трейт `Deref` дозволяє екземпляру структури розумного вказівника поводитись, як посилання, тож ви можете писати свій код, що працюватиме як із посиланнями, так і з розумними вказівниками. Трейт `Drop` дозволяє змінити код, що виконується при виході екземпляра розумного вказівника з області видимості. У цьому розділі буде розглянуто обидва трейта та продемонстровано, чому вони важливі для розумних вказівників.

Оскільки паттерн розумного вказівника є загальним паттерном проєктування, що часто використовується в Rust, цей розділ не розглядає усі можливі розумні вказівники. Багато бібліотек мають власні розумні вказівки, і ви навіть можете написати свої власні. Ми розглянемо найпоширеніші розумні вказівники стандартної бібліотеки:

* `Box<T>` для розміщення значень в heap
* `Rc<T>`, тип з підрахунком посилань, який уможливлює множинне володіння
* `Ref<T>` та `RefMut<T>`, accessed through `RefCell<T>`, тип що забезпечує виконання правил запозичення під час виконання, а не компіляції

Додатково ми розглянемо паттерн *внутрішньої мутабельності*, де немутабельний тип надає API для зміни внутрішнього значення. Також будуть розглянуті *зациклювання посилань*: як вони можуть розтратити пам'ять та як цьому запобігти.

Отже, вперед!
