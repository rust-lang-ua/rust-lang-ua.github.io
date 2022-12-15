## Організація тестів

Як зазначено на початку розділу, тестування є складною дисципліною, і різні люди використовують різну термінологію та організацію. Спільнота Rust думає про тести з точки зору двох основних категорій: модульні тести та інтеграційні тести. *Модульні тести* є невеликими та більш сфокусованими, ізольовано тестують один модуль за один раз, і можуть тестувати приватні інтерфейси. *Інтеграційні тести* є повністю зовнішніми до вашої бібліотеки та використовують ваш код так само як будь-який інший зовнішній код, використовуючи тільки публічний інтерфейс і потенційно випробовуючи багато модулів під час тесту.

Написання обох типів тестів є важливим, щоб переконатися, що частини вашої бібліотеки роблять те, на що ви очікували від них, окремо і разом.

### Модульні тести

Мета модульних тестів — перевірити кожну одиницю коду ізольованою від решти коду, щоб швидко визначити точку, де код не працює як очікувалося. Модульні тести розташовуються в теці *src* в кожному файлі коду, який вони тестують. За домовленістю, у кожному файлі, що містить функції для тестування, створюється модуль з назвою `tests`, анотований `cfg(test)`.

#### Модуль tests і `#[cfg(test)]`

Анотація модуля tests `#[cfg(test)]` каже Rust компілювати і виконувати тестовий код лише коли ви запускаєте `cargo test`, а не `cargo build`. Це зберігає час компіляції, коли ви хочете зібрати бібліотеку, і зберігає місце у отриманому скомпільованому артефакті, бо тести не до нього не включені. Як ви побачите, оскільки інтеграційні тести розміщуються в іншій теці, вони не потребують анотації `#[cfg(test)]`. Однак, оскільки модульні тести розміщуються у тих самих файлах, що й код, вам треба вказувати `#[cfg(test)]`, щоб позначити, що їх не треба включати у результат компіляції.

Згадайте, що коли ми створили новий проєкт `adder` у першому підрозділу цього розділу, Cargo згенерував для нас цей код:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Цей код є автоматично згенерованим модульним тестом. Атрибут `cfg` означає *конфігурація* і каже Rust, що наступний елемент має включатися лише з певною опцією конфігурації. У цьому випадку опцією конфігурації є `test`, що надається Rust для компіляції і запуску тестів. Використовуючи атрибут `cfg`, ми вказуємо Cargo компілювати наш тестовий код лише коли ми явно запускаємо тести за допомогою `cargo test`. Це стосується і будь-яких допоміжних функцій, що можуть бути в цьому модулі, на додачу до функцій, анотованих `#[test]`.

#### Тестування приватних функцій

У тестовій спільноті є дискусія про те, чи мають приватні функції тестуватися безпосередньо, і інші мови ускладнюють або унеможливлюють тестування приватних функцій. Незалежно від того, якої тестової ідеології ви дотримуєтеся, правила приватності Rust дозволяють вам тестувати приватні функції. Розгляньте код у Блоці коду 11-12 з приватною функцією `internal_adder`.

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

<span class="caption">Блок коду 11-12: тестування приватної функції</span>

Зверніть увагу, що функція `internal_adder` не позначена як `pub`. Тести - це просто код Rust, а модуль `tests` - це просто ще один модуль. Як ми вже говорили в підрозділі [“Способи звернутися до елементу в дереві модулів”][paths]<!-- ignore -->
, елементи дочірніх модулів можуть використовувати елементи своїх батьківських модулів. У цьому тесті, ми вводимо всі елементи батьківського для `test` модуля в область видимості за допомогою `use super::*`, і тоді тест може викликати `internal_adder`. Якщо ви не вважаєте, що приватні функції мають бути протестовані, немає нічого в Rust, що змусить вас це робити.

### Інтеграційні тести

У Rust, інтеграційні тести є цілковито зовнішніми відносно до вашої бібліотеки. Вони використовують вашу бібліотеку так само як це робив би будь-який інший код, що означає, що вони можуть викликати лише функції, які є частиною публічного API вашої бібліотеки. Їхнє призначення - перевірити, чи правильно різні частини вашої бібліотеки працюють разом. Фрагменти коду, які правильно самі по собі працюють, можуть мати проблеми при інтеграції, тому покриття інтегрованого коду тестами також важливе. Для створення інтеграційних тестів вам знадобиться для початку тека *tests*.

#### Тека *tests*

Ми створимо теку *tests* на верхньому рівні тек нашого проєкту, поруч із *src*. Cargo знає, що файли інтеграційних тестів треба шукати в цій теці. Ми можемо зробити стільки тестових файлів, скільки захочемо, і Cargo скомпілює кожен з файлів як окремий крейт.

Створімо інтеграційний тест. Поки у файлі *src/lib.rs* все ще код з Блоку коду 11-12, створіть теку *tests*, а в ній - новий файл, з назвою *tests/integration_test.rs*. Структура вашої теки має виглядати ось так:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
        └── integration_test.rs
```

Введіть код з Блоку коду 11-13 у файл *tests/integration_test.rs*:

<span class="filename">Файл: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```


<span class="caption">Блок коду 11-13: інтеграційний тест функції з крейту `adder`</span>

Кожен файл у теці `tests` є окремим крейтом, тож нам потрібно ввести нашу бібліотеку до області видимості кожного тестового крейту. Саме тому ми додаємо `use adder` на початку коду, чого не робили в модульних тестах.

Нам не треба додавати до коду у *tests/integration_test.rs* анотацію `#[cfg(test)]`. Cargo розглядає теку `tests` окремо і компілює файли у цій теці лише коли ми запускаємо `cargo test`. Запустімо зараз `cargo test`:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Три секції виводу містять модульні тести, інтеграційні тести та документаційні тести. Зверніть увагу, що якщо будь-який тест у секції провалиться, наступна секція не буде запущена. Наприклад, якщо провалиться модульний тест, для інтеграційних і документаційних тестів не буде виведено нічого, бо ці тести будуть запущені лише якщо всі модульні тести пройдуть.

Перша секція для модульних тестів така сама, яку ми вже бачили: по рядку для кожного модульного тесту (один, що зветься `internal`, який ми додали у Блоці коду 11-12) і далі рядок підсумку для модульних тестів.

Секція інтеграційних тестів починається рядком `Running
tests/integration_test.rs`. Далі по рядку для кожної тестової функції у інтеграційному тесті і рядок підсумку для результатів інтеграційних тестів прямо перед початком секції `Doc-tests adder`.

Кожен файл інтеграційного тесту має свою власну секцію, тому якщо ми додамо більше файлів до теки *tests*, буде більше секцій інтеграційних тестів.

Ми все ще можемо запустити певну функцію інтеграційного тесту, вказавши назву тестової функції як аргумент до `cargo test`. Щоб запустити всі тести з певного файлу інтеграційних тестів, вкажіть `cargo test` аргумент `--test` із назвою файлу:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Ця команда виконає лише тести у файлі *tests/integration_test.rs*.

#### Підмодулі у інтеграційних тестах

При додаванні інтеграційних тестів для кращої організації ви можете захотіти створити більше файлів у теці *tests*; наприклад, ви можете згрупувати тестові функції за функціоналом, який вони тестують. Як згадувалося раніше, кожен файл у теці *tests* компілюється як окремий крейт, що є корисним для створення окремих областей видимості для більш ретельного наслідування того, як кінцеві користувачі будуть використовуючи ваш крейт. Проте це означає, що файли в теці *tests* не виявляють таку ж поведінку як файли у *src*, як ви дізналися в Розділі 7 щодо того, як відокремити код в модулі та файли.

Відмінна поведінка каталогу *tests* є найбільш помітною, коли ви маєте набір допоміжних функцій, які використовуються в декількох файлах інтеграційних тестів і ви намагаєтесь слідувати крокам з підрозділу ["Розподіл модулів на різні файли"]()<!-- ignore --> Розділу 7, щоб винести їх у спільний модуль. Наприклад, якщо ми створимо *tests/common.rs* і розмістимо там функцію з назвою `setup`, ми можемо додати в цю функцію код, що ми хочемо викликати з декількох тестових функцій у декількох тестових файлах:

<span class="filename">Файл: src/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Коли ми знову запустимо тести, то побачимо нову секцію у виведенні тестів для файлу *common.rs*, хоча цей файл не містить жодних тестових функцій і ми нізвідки не викликали функцію `setup`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Побачити `common` серед результатів тестів з уточненням `running 0 tests` - ми не цього хотіли. Ми хотіли лише мати код, спільний для кількох файлів інтеграційних тестів.

Щоб `common` не з'являвся в результатах тестів, замість створення *tests/common.rs* ми створимо *tests/common/mod.rs*. Тека проєкту тепер виглядає так:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│    └── lib.rs
└── tests
     ├── common
     │    └── mod.rs
     └── integration_test.rs
```

Це давніше правило іменування, яке Rust також розуміє, про яке ми згадували у підрозділі ["Альтернативні шляхи файлів"][alt-paths]<!-- ignore --> Розділу 7. Те, що файл названо у цей спосіб, каже Rust не розглядати модуль `common` як файл інтеграційного тесту. Коли ми перемістимо код функції `setup` до *tests/common/mod.rs* і видалимо файл *tests/common.rs*, секція для цього файлу більше не показуватиметься. Файли в підтеках теки *tests* не компілюються як окремі крейти і не мають секції в виведенні тестів.

Після того, як ми створили *tests/common/mod.rs*, ми можемо використовувати його з будь-якого з тестових файлів як модуль. Ось приклад виклику функції `setup` з тесту `it_adds_two` в *tests/integration_test.rs*:

<span class="filename">Файл: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Зверніть увагу, що проголошення `mod common;` - те саме, що й проголошення модуля, продемонстроване в Блоці коду 7-21. Тоді з тестової функції ми можемо викликати функцію `common::setup()`.

#### Інтеграційні тести для двійкових крейтів

Якщо наш проєкт є двійковим крейтом, що містить лише файл *src/main.rs* і не має файлу *src/lib.rs*, ми не можемо створювати інтеграційні тести у теці *tests* і вводити в область видимості функції, визначені у файлі *src/main.rs*, за допомогою інструкції `use`. Лише бібліотечні крейти надають функції для використання в інших крейтах; двійкові крейти призначені лише для запуску.

Це - одна з причин, чому проєкти Rust, що створюють двійковий файл, мають простий файл *src/main.rs*, що викликає логіку з файлу *src/lib.rs*. За такої структури інтеграційні тести *можуть* тестувати бібліотечний крейт, використовуючи `use`, щоб дістатися до важливого функціоналу. Якщо важливий функціонал працює, невеликий код у файлі *src/main.rs* також працюватиме, і цей невеликий код не треба тестувати.

## Підсумок

Можливості тестування Rust надають можливість вказати, як код має працювати, щоб переконатися, що він і надалі працює як очікувалося, навіть якщо ви його зміните. Модульні тести випробовують різні частини бібліотеки окремо і можуть тестувати приватні деталі реалізації. Інтеграційні тести перевіряють, що різні частини бібліотеки коректно працюють разом, і вони використовують публічний API бібліотеки для тестування коду, так само, як це робитиме сторонній код. Попри те, що система типів Rust і правила власності допомагають уникнути деяких видів помилок, тести все одно є важливими для зменшення логічних помилок, які стосуються очікуваної поведінки вашого коду.

Застосуймо усі знання, отримані в цьому та попередніх розділах, щоб попрацювати над проєктом!
ch07-05-separating-modules-into-different-files.html

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths