## Публікація Крейта на Crates.io

Ми використовували пакети з [crates.io](https://crates.io/)<!-- ignore --> як залежності проекту, але ви також можете поділитися своїм кодом з іншими людьми, опублікувавши ваші власні пакети. Реєстр крейтів на [crates.io](https://crates.io/)<!-- ignore --> поширює початковий код ваших пакетів, тому він в першу чергу розміщує open-source код.

Rust і Cargo мають функціонал, який полегшує пошук та використання вашого опублікованого пакета. Далі ми поговоримо про деякі з цих можливостей і потім пояснимо, як опублікувати пакет.

### Робимо Корисні Коментарі в Документації

Якісне документування ваших пакетів допоможе іншим користувачам знати, як і коли їх використовувати, тому варто вкладати час на написання документації. У Розділі 3, ми обговорювали як коментарі Rust коду використовують подвійний слеш, `//`. Rust також має особливий вид коментарів для документації, зручно відомий як *документаційні коментарі*, які також будуть створювати HTML документацію. HTML покаже зміст документаційних коментарів для елементів публічного API, розрахованих на програмістів, які зацікавлені в *використанні* вашого крейту на відміну від того, як ваш крейт *імплементовано*.

Документаційні коментарі використовують три слеші, `///`, замість двох та підтримують Markdown для форматування тексту. Розміщуйте документаційні коментарі безпосередньо перед елементом, який вони документують. Блок коду 14-1 показує документаційні коментарі для функції `add_one` крейту з назвою `my_crate`.

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```


<span class="caption">Listing 14-1: A documentation comment for a function</span>

Тут ми дамо опис того, що робить функція `add_one`, почнемо розділ з заголовком `Приклади`, і надамо код, який продемонструє, як використовувати функцію `add_one`. Ми можемо створити HTML документацію з документаційних коментарів, запустивши `cargo doc`. Ця команда запускає інструмент `rustdoc`, який поширюється з Rust і кладе згенеровану HTML документацію в директорії *target/doc*.

Для зручності, запуск `cargo doc --open` збере HTML для Вашої поточної документації (а також документації для всіх залежностей вашого крейту) і відкриє результат у браузері. Перейдіть до функції `add_one` та ви побачите, як текст коментарів документації відтворюється, як показано на Малюнку 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: HTML documentation for the `add_one` function</span>

#### Часто Вживані Розділи

Ми використовували Markdown заголовок `# Examples` в Блоці Коду 14-1 для створення секції в HTML з назвою “Examples.” Ось ще кілька секцій, які автори крейтів зазвичай використовують у своїх документаціях:

* **Паніки**: Сценарії, в яких документована функція може запанікувати. Користувачі, які будуть використовувати ці функції і які не хочуть, щоб їх програма панікувала, повинні бути впевнені, що вони не викликають функції в цих ситуаціях.
* **Помилки**: Якщо функція повертає `Result`, який описує різновиди можливих помилок та які умови можуть призвести до повернення цих помилок, користувачам функції може бути корисно, щоб вони могли написати код для обробки різних помилок різними способами.
* **Безпека**: Якщо функція `unsafe` (ми обговоримо небезпечність в Розділі 19), то має бути секція, в якій пояснюється, чому функція небезпечна та її інваріанти, які мають дотримуватися користувачі функції.

Most documentation comments don’t need all of these sections, but this is a good checklist to remind you of the aspects of your code users will be interested in knowing about.

#### Коментарі Документації як Тести

Додавання прикладу блоків коду в ваші коментарі документації може допомогти продемонструвати, як ви використовуєте вашу бібліотеку, і в цьому є додатковий бонус: запуск `cargo test` запускатиме приклади коду в вашій документації як тести! Немає нічого приємнішого ніж документація з прикладами. Але немає нічого гіршого ніж приклади, які не працюють, бо код змінився з моменту написання документації. Якщо ми запустимо `cargo test` із документацією для функції `add_one` з Блока Коду 14-1, ми побачимо секцію результатів тестів наступним чином:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Now if we change either the function or the example so the `assert_eq!` in the example panics and run `cargo test` again, we’ll see that the doc tests catch that the example and the code are out of sync with each other!

#### Коментування Присутніх Елементів

Стиль документаційних коментарів `//!` додає документацію до елемента, який містить коментарі, аніж до елементів, що слідують за коментарями. Як правило, ми використовуємо документаційні коментарі всередині кореневого файлу крейта (*src/lib.rs* за домовленістю) або всередині модуля для документування крейта або модуля в цілому.

For example, to add documentation that describes the purpose of the `my_crate` crate that contains the `add_one` function, we add documentation comments that start with `//!` to the beginning of the *src/lib.rs* file, as shown in Listing 14-2:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```


<span class="caption">Listing 14-2: Documentation for the `my_crate` crate as a whole</span>

Зауважте, що тут немає коду після останнього рядку, який починається з `//!`. Оскільки ми почали коментар з `//!` замість `///`, ми документуємо предмет який міститься в коментарі, замість предмета, який слідує за коментарем. У цьому випадку, цей елемент це файл *src/lib.rs*, який містить кореневий каталог. Ці коментарі описують увесь крейт.

When we run `cargo doc --open`, these comments will display on the front page of the documentation for `my_crate` above the list of public items in the crate, as shown in Figure 14-2:

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figure 14-2: Rendered documentation for `my_crate`, including the comment describing the crate as a whole</span>

Документаційні коментарі всередині елементів корисні для опису крейтів і особливо модулів. Використовуйте їх, щоб пояснювати загальну мету контейнера, щоб допомогти вашим користувачам зрозуміти організацію крейту.

### Експорт Зручного Публічного API з `pub use`

Структура вашого публічного API має вирішальне значення, коли ви публікуєте крейт. Користувачі вашого крейту менш знайомі зі структурою ніж ви та можуть мати труднощі у пошуку бажаних частин, якщо у вашому крейті велика ієрархія модулів.

У Розділі 7 ми розглянули, як робити елементи публічними за допомогою ключового слова `pub` та вносити елементи в область видимості з ключовим словом `use`. Однак, структура, яка має сенс під час розробки крейту, може бути не дуже зручна для ваших користувачів. Ви можливо захочете організувати ваші структури в багаторівневій ієрархії, але потім люди які захочуть використати визначений глибоко в ієрархії тип можуть мати проблеми з з'ясуванням, що цей тип існує. Вони також можуть бути роздратованими через необхідність писати `use` `my_crate::some_module::another_module::UsefulType;` замість `use` `my_crate::UsefulType;`.

Хороші новини полягають в тому, що якщо іншим користувачам *не* зручно використовувати її з іншої бібліотеки, вам не потрібно переробляти вашу внутрішню організацію: натомість, ми можете повторно експортувати елементи, щоб зробити публічну структуру, яка відрізняється від вашої приватної структури використанням `pub use`. Повторне експортування бере публічний елемент з одного місця і робить його публічним в іншому місці, ніби це було визначено в іншій локації.

Скажімо, ми зробили бібліотеку з назвою `art` для моделювання художніх концепцій. Всередині цієї бібліотеки є два модулі: модуль `kinds`, який містить два енуми із назвами `PrimaryColor` та `SecondaryColor` і модуль `utils`, який містить функцію `mix`, як показано в Блоці Коду 14-3:

<span class="filename">Файл: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```


<span class="caption">Listing 14-3: An `art` library with items organized into `kinds` and `utils` modules</span>

Figure 14-3 shows what the front page of the documentation for this crate generated by `cargo doc` would look like:

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figure 14-3: Front page of the documentation for `art` that lists the `kinds` and `utils` modules</span>

Зауважте, що типи `PrimaryColor` та `SecondaryColor` не вказані на головній сторінці, так само як і функція `mix`. Нам потрібно натиснути на `kinds` та `utils`, щоб побачити їх.

Іншому залежному від цієї бібліотеки крейту знадобиться інструкція `use`, яка приносить елементи з `art` в область видимості, вказавши визначену зараз структуру модуля. Блок коду 14-4 показує приклад крейта, який використовую елементи `PrimaryColor` та `mix` з крейту `art`:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```


<span class="caption">Listing 14-4: A crate using the `art` crate’s items with its internal structure exported</span>

Автор коду в Блоці Коду 14-4, який використовує `art` крейт, має з'ясувати, що `PrimaryColor` в модулі `kinds`, а `mix` в модулі `utils`. Структура модуля `art` крейту є більш актуальною для розробників `art` крейту, ніж для його користувачів. Внутрішня структура не містить жодної корисної інформації для когось, хто намагається зрозуміти, як використовувати `art` крейт, радше викликає плутанину, бо розробники, які використовують цей крейт мають з'ясовувати, куди дивитися та мають вказувати назву модуля в інструкції `use`.

To remove the internal organization from the public API, we can modify the `art` crate code in Listing 14-3 to add `pub use` statements to re-export the items at the top level, as shown in Listing 14-5:

<span class="filename">Файл: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```


<span class="caption">Listing 14-5: Adding `pub use` statements to re-export items</span>

The API documentation that `cargo doc` generates for this crate will now list and link re-exports on the front page, as shown in Figure 14-4, making the `PrimaryColor` and `SecondaryColor` types and the `mix` function easier to find.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figure 14-4: The front page of the documentation for `art` that lists the re-exports</span>

The `art` crate users can still see and use the internal structure from Listing 14-3 as demonstrated in Listing 14-4, or they can use the more convenient structure in Listing 14-5, as shown in Listing 14-6:

<span class="filename">Файл: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```


<span class="caption">Listing 14-6: A program using the re-exported items from the `art` crate</span>

У випадках, коли є багато вкладених модулів, повторне експортування типів на вищий рівень із `pub use` може зробити суттєву різницю в досвіді використання цього крейта. Ще одним поширеним використанням `pub use` є повторне експортування визначень залежностей в поточному крейті, щоб зробити визначення цього крейту частиною публічного API.

Створення корисної публічної структури API є скоріше мистецтвом ніж наукою, і ви можете ітерувати, щоб знайти API яке найкраще підходить для ваших користувачів. Вибір `pub use` дає вам гнучкість у тому, як ви внутрішньо структуруєте свій крейт та відділяє внутрішню структуру від того, що ви представляєте своїм користувачам. Подивімося на деякий код з встановлених вами крейтів, щоб побачити, чи їх структура відрізняється від їх публічного API.

### Налаштування Облікового Запису Crates.io

Перш ніж ви зможете опублікувати якісь крейти, вам потрібно створити обліковий запис на [crates.io](https://crates.io/)<!-- ignore --> і отримати API токен. Для цього, відвідайте домашню сторінку на [crates.io](https://crates.io/)<!-- ignore --> і увійдіть за допомогою вашого облікового запису Github. (Обліковий запис на GitHub наразі є вимогою, але сайт може підтримувати інші способи створення облікового запису в майбутньому.) Після входу перейдіть до налаштувань облікового запису на [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> і отримайте ваш API ключ. Потім запустить команду `cargo login` із вашим API токеном наступним чином:

```console
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

Ця команда повідомить Cargo про ваш API токен та збереже його локально в *~/.cargo/credentials*. Зауважте, що токен це *секрет*: не діліться ним з будь-ким іншим. Якщо ви поділитесь ним з будь-ким задля будь-якої причини, ви можете відкликати його та створити новий токен на [crates.io](https://crates.io/)<!-- ignore
-->.

### Додавання Метаданих до Нового Крейту

Скажімо ви маєте крейт який ви хочете опублікувати. Перед публікацією, вам буде потрібно додати деякі метадані в секції `[package]` файлу *Cargo.toml* вашого крейту.

Вашому крейту знадобиться унікальне ім'я. Поки ви працюєте над крейтом локально, ви можете називати його як завгодно. Однак, назва крейту на [crates.io](https://crates.io/)<!-- ignore --> виділяється в порядку живої черги(перший прийшов - перший отримав). Як тільки назва крейту обрана, ніхто інший не може опублікувати крейт із цією назвою. Перед спробою опублікувати крейт, пошукайте назву, яку ви бажаєте використовувати. Якщо назва зайнята, вам знадобиться обрати іншу назву та редагувати поле `name` в файлі *Cargo.toml* в секції `[package]`, щоб використати нову назву для публікації наступним чином:

<span class="filename">Файл: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Even if you’ve chosen a unique name, when you run `cargo publish` to publish the crate at this point, you’ll get a warning and then an error:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
Перегляньте https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata для додатковох інформації.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Будь ласка перегляньте https://doc.rust-lang.org/cargo/reference/manifest.html щодо того, як завантажити метадані
```

Це помилка, оскільки у вас відсутня деяка вирішальна інформація: опис та ліцензія які необхідні для того, щоб люди знали, що ваш крейт робить та на яких умовах вони можуть його використовувати. У *Cargo.toml*, додайте опис розміром з речення або два, оскільки воно з'явиться з вашим крейтом в результаті пошуку. Для поля `license` вам потрібно надати *значення ідентифікатора ліцензії*. [Linux Foundation’s Software Package Data Exchange (SPDX)][spdx] перелічує ідентифікатори які ви можете використати як це значення. Наприклад, щоб вказати, що ваш крейт використовує ліцензію MIT, додайте ідентифікатор `MIT`:

<span class="filename">Файл: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

If you want to use a license that doesn’t appear in the SPDX, you need to place the text of that license in a file, include the file in your project, and then use `license-file` to specify the name of that file instead of using the `license` key.

Супровід щодо того, яка ліцензія підійде вашому проєкту, поза рамками цієї книги. Багато людей у спільноті Rust ліцензує їх проєкти так само як і Rust, використовуючи подвійну ліцензію `MIT OR Apache-2.0`. Ця практика демонструє, що ви також можете вказати декілька ідентифікаторів ліцензій, які відокремлені `OR`, щоб використовувати декілька ліцензій в вашому проєкті.

With a unique name, the version, your description, and a license added, the *Cargo.toml* file for a project that is ready to publish might look like this:

<span class="filename">Файл: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) describes other metadata you can specify to ensure others can discover and use your crate more easily.

### Публікація на Crates.io

Тепер, коли ви створили обліковий запис, зберегли ваш API токен, обрали назву вашого крейту та вказали необхідні метадані, ви готові до публікації! Публікація крейту завантажує конкретну версію на [crates.io](https://crates.io/)<!-- ignore --> для використовування іншими.

Будьте обережні, оскільки публікація *перманентна*. Версія ніколи не може бути перезаписана, а код ніколи не може бути видалений. Одна з основних цілей [crates.io](https://crates.io/)<!-- ignore --> це діяти як перманентний архів коду, щоб збірка кожного проекту залежного від крейту на [crates.io](https://crates.io/)<!-- ignore --> продовжувала працювати. Дозволяючи видалення версій ми робимо виконання цієї цілі неможливим. Однак, немає обмежень на кількість версій крейтів, які ви можете опублікувати.

Запустимо знову команду `cargo publish`. Тепер воно має бути вдалим:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Вітаємо! Ви зараз поділилися вашим кодом із Rust спільнотою та кожен може легко додати ваш крейт як залежність до його проєкту.

### Публікація Нової Версії Існуючого Крейту

Коли ви внесли зміни в ваш крейт та готові випустити нову версію ви змінюєте значення `version`, яке вказане в вашому файлі *Cargo.toml* та знову публікуйте. Використовуйте [правила Семантичного Версіонування][semver] для вирішення відповідного наступного номера версії, на основі зроблених вами змін. Потім запускайте `cargo publish` для завантаження нової версії.

<!-- Old link, do not remove -->
<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### Вилучаємо Старі Версії з Crates.io з `cargo yank`

Хоча ми не можемо видалити попередні версії крейта, ми можемо запобігти будь-яким майбутнім проєктам додавання цих версій як нової залежності. Це корисно, коли версія крейту зламана по тій чи іншій причині. У таких ситуаціях Cargo підтримує *висмикування або yanking* версії крейту.

Висмикування версії перешкоджає залежність від цієї версії новими проєктами, дозволяючи усім чинним проєктам, які залежать від неї, продовжувати її використовувати. По суті, смик означає, що всі проєкти з *Cargo.lock* не зламаються та будь-який наступний створений файл *Cargo.lock* не буде використовувати висмикану версію.

Щоб висмикнути версію крейту в директорії, яку ви раніше публікували запустіть команду `cargo yank` та зазначте яку версію ви хочете висмикнути. Наприклад, якщо ви опублікуєте крейт з назвою `guessing_game` версії 1.0.1 та захочете висмикнути її, в теці вашого проєкту для `guessing_game` ви б виконали:

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game:1.0.1
```

By adding `--undo` to the command, you can also undo a yank and allow projects to start depending on a version again:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game_:1.0.1
```

Висмикування *не* видаляє жодного коду. Воно не може, наприклад, випадково видалити завантажені секрети. Якщо це станеться, вам буде потрібно негайно відновити ці секрети.

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
