## Коментарі

Всі програмісти прагнуть зробити свій код зрозумілішим, та деколи не завадить додаткове пояснення. В таких випадках програмісти лишають в початковому коді *коментарі*, які ігнорує компілятор, але які можуть бути корисними людям, що читатимуть цей код.

Ось простий коментар:

```rust
// hello, world
```

У Rust найтиповіший стиль коментарів починається з двох знаків дробу і продовжується до кінця рядка. Для коментарів, що займають більше одного рядка, вам доведеться ставити `//` у кожному рядку, ось так:

```rust
// Тут ми робимо щось складне, досить довге, щоб нам знадобилося кілька рядків
// коментаря! Хух! Сподіваюся, цей коментар достатньо детально пояснює, що 
// тут відбувається.
```

Коментарі також можна розміщувати в кінці рядків, що містять код:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Та частіше ви бачитимете їх у форматі, де коментар знаходиться в окремому рядку перед кодом, якого він стосується:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust також має інший вид коментарів, документаційні коментарі, які ми обговоримо в підрозділі ["Публікація крейта на Crates.io"][publishing]<!-- ignore -->
Розділу 14.

[publishing]: ch14-02-publishing-to-crates-io.html
