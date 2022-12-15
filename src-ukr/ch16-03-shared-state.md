## Паралелізм із спільним станом

Обмін повідомленнями - чудовий, але не єдиний спосіб роботи з конкурентністю. Іншим способом можу бути доступ декількох потоків до спільних даних. Розглянемо наступну частину слогану з документації мови програмування Go ще раз: "не комунікуйте за допомогою спільної памʼяті."

Як би виглядала комунікація за допомогою спільної памʼяті? Окрім того, чому ентузіасти обміну повідомленнями застерігають від використання спільної памʼяті?

У певному сенсі, канали в будь-якій мові програмування схожі на одноособове володіння, тому що як тільки ви передали значення по каналу, ви не повинні більше використовувати таке значення. Конкурентність із спільною памʼяттю нагадує множинне володіння: декілька потоків одночасно мають доступ до однієї і тієї ж області памʼяті. Як ви могли бачити в Розділі 15, де розумні вказівники робили множинне володіння можливим, таке володіння може додати програмі складності, оскільки потрібно управляти різними власниками (owners). Система типів Rust та правила володіння дуже допомагають здійснювати таке управління коректно. Наприклад, давайте розглянемо мʼютекси, один з найпоширеніших примітивів конкурентності для роботи із спільною памʼяттю.

### Використання мʼютексів для доступу до даних з лише з одного потоку в момент часу

*Mutex (мʼютекс)* - це абревіатура для *mutual exclusion (взаємне виключення)*, оскільки мʼютекс дозволяє лише одному потоку отримувати доступ до даних в будь-який момент часу. Для того, щоб отримати доступ до даних у мʼютексі, потік має спочатку повідомити, що він бажає отримати доступ, запросивши отримати *блокування (lock)* мʼютексу. Блокування - це структура даних, що є частиною мʼютексу і відстежує хто саме має ексклюзивний доступ до даних. Саме тому, мʼютекс описують як *захист* даних, які він в собі зберігає, за допомогою системи блокування.

Мʼютекси мають репутацію складного в використанні механізму, оскільки ви маєте памʼятати два правила:

* Ви повинні спробувати отримати блокування перед використанням даних.
* Коли ви закінчите працювати з даними, що захищає мʼютекс, ви маєте розблокувати дані, щоб інші потоки могли отримати блокування.

Метафорою для мʼютексу можна вважати панельну дискусію на конференції лише з одним мікрофоном. Перед тим як інший учасник дискусії зможе говорити, він повинен попросити або показати, що він хоче скористатись мікрофоном. Коли він отримає мікрофон, він може говорити стільки, скількі вважає за потрібне, а потім передати мікрофон наступному учаснику дискусії, який просить слово. Якщо учасник дискусії забуває передати мікрофон після того, як він закінчив, то ніхто інший не матиме змоги говорити. Якщо управління спільним мікрофоном піде неправильно, то панельна дискусія не працюватиме так, як заплановано!

Правильне управління мʼютексами може бути неймовірно складним, ось чому так багато людей з ентузіазмом ставиться до каналів. Однак, завдяки системі типів Rust та правилам володіння, ви не можете помилитись при блокуванні та розблокуванні.

#### API `Mutex<T>`

Щоб продемонструвати як використовувати мʼютекс, давайте почнемо з використання мʼютексу в однопоточному контексті, як показано в Блоці коду 16-12:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```


<span class="caption">Блок коду 16-12: Експерименти з API `Mutex<T>` в однопоточному контексті для простоти</span>

Як і з багатьма типами, ми створюємо `Mutex<T>`, використовуючи функцію `new`. Для доступу до даних всередині мʼютекса, ми використовуємо метод `lock` для отримання блокування. Цей виклик заблокує поточний потік, щоб він не міг виконувати жодну роботу до моменту поки не настане наша черга отримувати блокування.

Виклик `lock` завершиться неуспішно, якщо інший потік, котрий тримав блок, запанікував (panicked). В такому випадку, ніхто ніколи не зможе отримати блок, тому ми вирішили використати `unwrap` і змусити потік запанікувати, якщо ми опинимось в такій ситуації.

Після того, як ми отримали блокування, ми можемо розглядати повернуте значення, яке в даному випадку називається `num`, як мутабельне посилання на дані всередині. Система типів гарантує, що ми отримуємо блокування перед тим як використати значення в `m`. Тип `m` - `Mutex<i32>`, а не `i32`, тому ми *зобовʼязані* викликати `lock` щоб мати змогу використовувати значення `i32`. Ми не можемо забути про це; інакше система типів не дозволить нам отримати доступ до внутрішнього `i32`.

Як ви могли запідозрити, `Mutex<T>` є розумним вказівником. Точніше, виклик `lock` *повертає* розумний покажчик, котрий називається `MutexGuard`, загорнутий в `LockResult`, який ми обробили за допомогою виклика `unwrap`. `MutexGuard` - це розумний вказівник, що реалізує `Deref`, щоб вказувати на внутрішні дані; розумний вказівник такж має реалізацію `Drop`, котра вивільняє блок автоматично, коли `MutexGuard` виходить за межі області видимості, що відбувається в кінці внутрішньої області видимості. Як наслідок, ми не ризикуємо забути розблокувати блок і заблокувати використання мʼютексу іншими потоками, оскільки розблокування блоку відбувається автоматично.

Після видалення блоку, ми можемо вивести на екран значення мʼютексу і побачити, що ми змогли змінити внутрінє `i32` на 6.

#### Спільне використання `Mutex<T>` декількома потоками

Тепер давайте спробуємо, використати значення з декількох різних потоків за допомогою `Mutex<T>`. Ми запустимо 10 потоків і кожен з них буде збільшувати значення лічильника на 1, таким чином лічильник змінюватиме значення від 0 до 10. Наступний приклад в Блоці коду 16-3 містить помилку компіляції і ми використаємо цю помилку щоб дізнатися більше про використання `Mutex<T>` і як Rust допомагає нам правильно його використовувати.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```


<span class="caption">Блок коду 16-13: Десять потоків по черзі інкрементують лічильник, захищений за допомогою `Mutex<T>`</span>

Ми створюємо змінну `counter`, що містить `i32` всередині `Mutex<T>`, так само як ми зробили в Блоці коду 16-12. Далі, ми створюємо 10 потоків, що ітеруються по діапазону (range) чисел. Ми використовуємо `thread::spawn` і передаємо кожному потоку одне й те саме замикання, котре переміщує лічильник всередину потоку, отримує блокування `Mutex<T>`, викликаючи метод `lock`, а потім додає 1 до значення всередині мʼютексу. Коли потік завершує виконання замикання, `num` виходить з області видимості, звільняє блок (lock), щоб інший потік міг його отримати.

В основному потоці, ми збираємо (collect) всі обробники (join handles). Після цього, так само як і в Блоці коду 16-2, ми викликаємо `join` на кожному обробнику, щоб впевнитись, що всі потоки завершуються. В цей момент основний потік отримає блокування і виведе на екран результат виконання цієї програми.

Ми натякнули, що цей приклад не скомпілюється. А тепер давайте дізнаємось чому!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

У повідомленні про помилку вказано, що значення `counter` вже було переміщено в попередній ітерації циклу. Rust говорить нам, що ми не можемо перемістити володіння блококуванням `counter` в декілька потоків. Виправимо помилку компіляції за допомогою множинного володіння, про яке ми говорили в Розділі 15.

#### Множинне володіння і декілька потоків

В Розділі 15, ми надали значення декільком власникам, використовуючи розумний вказівник `Rc<T>` щоб створити значення з підрахунком посилань. Зробімо тут те саме і подивимось, що станеться. Ми загорнемо `Mutex<T>` в `Rc<T>` в Блоці коду 16-14 і склонуємо `Rc<T>` перед переміщенням володіння всередину потоку.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```


<span class="caption">Блок коду 16-14: Спроба використати `Rc<T>` щоб дозволити потокам володіти `Mutex<T>`</span>

Компілюємо знов і отримуємо... інші помилки! Компілятор нас багато чому вчить.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Ох, це повідомлення про помилку доволі багатослівне! Ось важлива частина, на яку треба звернути увагу: `` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. Компілятор також повідомляє нам чому: `` the trait `Send` is not implemented for
`Rc<Mutex<i32>>` ``. Ми поговоримо про `Send` в наступній секції: це один з трейтів, що гарантують, що типи, котрі ми використовуємо в потоках, призначені для використання в конкурентних ситуаціях.

На жаль, `Rc<T>` небезпечно спільно використовувати в декількох потоках. Коли `Rc<T>` керує підрахунком посилань, він додає одиницю до лічильника за кожен виклик `clone` і віднімає одиницю від лічильника, кожного разу коли значення клону видаляється. Проте він не використовує жодних примітивів конкурентності, щоб переконатися, що зміни лічильника не будуть перервані іншим потоком. Це може призвести до неправильного підрахунку посилань - проблем, які дуже важко помітити й ідентифікувати, і можуть призвести до витоків памʼяті (memory leaks) або ж значення може бути видалене, до того як ми з ним закінчимо. Нам потрібен тип, ідентичний `Rc<T>`, але такий, що робить зміни до лічильника підрахунку посилань в потокобезпечний (thread-safe) спосіб.

#### Атомарний підрахунок посилань із `Arc<T>`

На щастя, `Arc<T>` *є* типом, схожим на `Rc<T>`, але який безпечно використовувати в конкурентних ситуаціях. Літера *a* означає *atomic*, тобто це тип *з атомарним підрахуванням посилань*. Атоміки - це додатковий вид примітивів конкурентності, які ми не будемо тут детально розглядати: див. документацію стандартної бібліотеки для [`std::sync::atomic`][atomic]<!-- ignore --> для більш докладної інформації. На даному етапі вам лише необхідно знати, що атоміки працюють як примітивні типи, але безпечні для спільного використання декількома потоками.

Ви можете запитати, чому всі примітивні типи не є атомариними і чому типи стандартної бібліотеки не використовують `Arc<T>` за замовчуванням. Причиною є те, що безпека потоків супроводжується зниженням швидкості виконання, а це штраф, який ви хочете заплатити лише тоді, коли це дійсно необхідно. Якщо ви просто виконуєте операції над значеннями в межах одного потоку, ваш код може працювати швидше, якщо йому не потрібно застосовувати гарантії, котрі надають атоміки.

Давайте повернемось до нашого прикладу: `Arc<T>` і `Rc<T>` мають однаковий API, тому ми просто виправляємо нашу програму змінюючи рядок з `use`, виклик `new`, а також виклик `clone`. Код в Блоці коду 16-15 нарешті скомпілюється й виконається:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```


<span class="caption">Блок коду 16-15: Використання `Arc<T>` для обгортання `Mutex<T>` щоб мати можливіть поділитися володінням між кількома потоками</span>

Цей код виводить на екран наступне:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Ми зробили це! Ми рахували від 0 до 10, що може здатися не дуже вражаючим, але це навчило нас багато чому про `Mutex<T>` та безпеку потоків. Ви також можете використовувати структуру цієї програми для виконання більш складних операцій, ніж просто збільшення лічильника. Використовуючи цю стратегію, ви можете розділити обчислення на незалежні частини, потім розділити ці частини між потоками, а потім використати `Mutex<T>`, щоб кожен потік оновив кінцевий результат своєю частиною.

Завважте, що якщо ви виконуєте прості числові операції, є типи простіші за `Mutex<T>`, що визначені в молдулі [`std::sync::atomic` стандартної бібліотеки][atomic]. Згадані типи забезпечують безпечний, конкурентний, атомарний доступ до примітивних типів. Для цього прикладу ми вирішили використовувати `Mutex<T>` із примітивним типом щоб ми могли зосередитися на тому, як працює `Mutex<T>`.

### Подібності між `RefCell<T>`/`Rc<T>` і `Mutex<T>`/`Arc<T>`

Ви могли помітити, що `counter` є імутабельним, але ми могли б отримати мутабельне посилання на значення в ньому; це означає, що `Mutex<T>` забезпечує внутрішню мутабельність (interior mutability), як це робить `Cell`. Таким же чином ми використовували `RefCell<T>` у Розділі 15, щоб дозволити нам змінювати контент всередині `Rc<T>`, ми використовуємо `Mutex<T>` щоб змінити вміст у `Arc<T>`.

Ще одна деталь, яку слід зазначити, полягає в тому, що Rust не може захистити вас від усіх видів логічних помилок під час використання `Mutex<T>`. Згадайте, що в Розділі 15 ми обговорювали, що використання `Rc<T>` супроводжується ризиком створення циклічних посилань, де два значення `Rc<T>` посилаються один на одного, спричиняючи витоки памʼяті (memory leaks). Подібним чином, використання `Mutex<T>` несе з собою ризик створення *взаємних блокувань*. Це відбувається, коли операція потребує блокування двох ресурсів і кожен з двох потоків отримав оне з блокувань, таким чином змушуючи їх вічно чекати один одного. Якщо вас цікавлять взаємні блокування, спробуйте створити Rust програму, яка має взаємне блокування; потім пошукайте стратегії вирішення проблеми взаємних блокувань для мʼютексів в будь-якій мові та спробуйте реалізувати їх на Rust. API документація стандартної бібліотеки для `Mutex<T>` і `MutexGuard` надає корисну інформацію.

Ми завершимо цей розділ розповіддю про трейти `Send` і `Sync` і те, як ми можемо їх використовувати разом з власними типами.

[atomic]: ../std/sync/atomic/index.html

[atomic]: ../std/sync/atomic/index.html