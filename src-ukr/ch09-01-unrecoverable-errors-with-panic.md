## Невідновні Помилки із `panic!`

Іноді погані речі трапляються у вашому коді з якими ви нічого не можете зробити. У цих випадках Rust має макрос `panic!`. Є два практичних способи викликати паніку: зробивши дію, яка призведе до паніки (наприклад отримати доступ до елемента масиву за його межами) або явно викликавши макрос `panic!`. В обох випадках ми викличемо паніку в нашій програмі. За замовчуванням, ці паніки виведуть в консолі повідомлення про помилку, розгорнуть та очистять стек та закриють програму. За допомогою змінної середовища ви також можете скерувати Rust показати стек викликів, коли виникне паніка, щоб полегшити відстеження її джерела.

> ### Розгортання Стека або Переривання у Відповідь на Паніку
> 
> За замовчуванням, коли виникає паніка, програма запускає *розгортання*. Це означає, що Rust проходиться по стеку та очищає дані всіх зустрічних функцій. Проте ця розгортка та очищення це багато роботи. Отже, Rust дозволяє вибрати альтернативу: негайно завершувати програму без її очищення.
> 
> Пам'ять, яку використовувала програма, тоді буде очищена операційною системою. Якщо у вашому проєкті вам потрібно зробити кінцевий бінарний файл якомога менше, ви можете змінити поведінку програми при паніці з розгортки стеку на негайне переривання додавши `panic = 'abort'` у відповідну секцію `[profile]` у вашому файлі *Cargo.toml*. Наприклад, якщо ви хочете негайне переривання паніки у режимі збірки, додайте наступне:
> 
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Спробуймо викликати `panic!` у простій програмі:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

Коли ви запускаєте програму, ви побачите щось на зразок цього:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Виклик `panic!` призвів до повідомлення про помилку, що міститься в останніх двох рядках. Перший рядок показує наше повідомлення про паніку і місце в нашому початковому коді, де вона сталася: *src/main.rs:2:5* вказує, що це другий рядок, п'ятий символ нашого файлу *src/main.rs*.

У цьому випадку зазначений рядок є частиною нашого коду, і якщо ми перейдемо до цього рядка, ми побачимо виклик макроса `panic!`. В інших випадках виклик `panic!` може бути в коді, який викликає наш код, а назва файлу і номер рядка, звітований повідомленням про помилку, буде чужим кодом, де викликається макрос `panic!`, а не рядок нашого коду, який зрештою призвів до `panic!`. Ми можемо використати бектрейс функції з якої прийшов виклик `panic`, щоб дізнатися, яка частина нашого коду викликає проблему. Ми обговоримо бектрейс у деталях пізніше.

### Використання Бектрейсу `panic!`

Розглянемо ще один приклад, коли виклик `panic!` йде з бібліотеки через помилку в нашому коді, а не через прямий виклик макроса нашим кодом. Блок коду 9-1 намагається отримати доступ до елемента вектора поза меж діапазону припустимих індексів.

<span class="filename">Файл: src/main.rs</span>

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```


<span class="caption">Блок коду 9-1: Спроба отримати доступ до елемента вектора за межами його кінця викличе `panic!`</span>

Тут ми намагаємося отримати доступ до 100-го елемента нашого вектора (який знаходиться за індексом 99, бо індексування починається з нуля), але вектор має всього 3 елементи. В цій ситуації Rust панікуватиме. Використання `[]` повинно повернути елемент, але якщо передати не валідний індекс, то не буде елементу, який Rust може повернути, що було б правильним.

У C спроба прочитати за межами кінця структури даних це не визначена поведінка або undefined behaviour. Ви можете отримати те, що розташоване в місці в пам'яті та відповідає цьому елементу структури даних, навіть якщо пам'ять не належить цій структурі. Це називається *читання поза межами буфера або buffer overread* і може стати причиною появи уразливостей в безпеці, якщо нападник здатен маніпулювати індексом таким чином, щоб прочитати дані які зберігаються поза структурою даних до яких він не має права на доступ.

Щоб захистити вашу програму від такого роду уразливості, при спробі прочитати елемент за індексом, якого не існує, Rust зупинить виконання програми. Спробуймо:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Ця помилка вказує на рядок 4 нашого файлу `main.rs`, де ми намагаємося отримати доступ до елемента за індексом
99. Наступний рядок каже нам, що ми можемо встановити змінну середовища `RUST_BACKTRACE` для отримання бектрейсу того, що саме стало причиною помилки. *Бектрейс* це список усіх функцій які були викликані до появи помилки. Бектрейси в Rust працюють так само як і в інших мовах: Читати бекстрейс потрібно зверху вниз й читати доти, доки ви не побачите назви ваших файлів. Ось місце, де виникла проблема. Рядки зверху це те, що викликано вашим кодом; рядки знизу це код, який викликає код зверху. Ці "до-та-після" рядки можуть включати код ядра Rust, код стандартної бібліотеки, або крейтів, що ви використовуєте. Спробуймо отримати бектрейс встановивши змінну середовища `RUST_BACKTRACE` будь-яке значення окрім 0. Блок коду 9-2 показує вивід схожий на те, що ви побачите.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/std/src/panicking.rs:483
   1: core::panicking::panic_fmt
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:85
   2: core::panicking::panic_bounds_check
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:62
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:255
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:15
   5: <alloc::vec::Vec<T> as core::ops::index::Index<I>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/alloc/src/vec.rs:1982
   6: panic::main
             at ./src/main.rs:4
   7: core::ops::function::FnOnce::call_once
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/ops/function.rs:227
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```


<span class="caption">Блок коду 9-2: Бектрейс утворений викликом <0>panic!</0> показує, коли змінна середовища <0>RUST_BACKTRACE</0> була встановлена</span>

Це багато виводу! Точний вивід може відрізнятися в залежності від версії операційної системи та версії Rust. Для того, щоб отримати бектрейс із цією інформацією, мають бути увімкнені дебаг символи. Дебаг символи увімкнуті за замовчуванням коли ви використовуєте `cargo build` або `cargo run` без позначки `--release`, як ми зробили тут.

У виводі Блока коду 9-2, рядок 6 бектрейсу вказує на наш рядок який спричиняє проблему: рядок 4 файлу *src/main.rs*. Якщо ми не хочемо, щоб наша програма панікувала, ми повинні почати наше розслідування з першого рядка, де згадується написаний нами файл. В Блоці коду 9-1, де ми навмисно написали код, який викличе паніку, спосіб виправлення паніки це не запитувати елемент поза межами діапазону індексів вектора. Надалі коли ваш код панікуватиме, вам потрібно буде з'ясовувати, які дії виконує код із якими значеннями щоб спричинити паніку і що код повинен робити натомість.

Ми ще повернемося до `panic!` і до того, коли нам слід і не слід використовувати `panic!` для обробки умов помилок у секції ["To `panic!`or Not to `panic!`”]()<!-- ignore --> пізніше у цьому розділі. Далі ми розглянемо, як відновляти помилки за допомогою `Result`.
ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic