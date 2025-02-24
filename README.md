# Лабораторна робота 7: Лямбда-вирази. Параметризація поведінки

## Цілі лабораторної роботи

- вивчити процес створення та сценарії використання анонімних об'єктів;
- навчитися створювати анонімні класи;
- розібратися зі створенням одиночних та блокових лямбда-виразів.

## Теоретичний матеріал

### 1. Функціональний інтерфейс `Predicate` 

 `Predicate` - вбудований функціональний інтерфейс, доданий до Java 8 до пакету `java.util.function`. Інтерфейс описує один метод `test()`, який приймає значення на вході, перевіряє його стан і повертає `boolean` у якості результату. Інтерфейс `Predicate` повертає `true` або `false` відповідно до деякого значення.

Інтерфейс виглядає наступним чином

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

> Поки що не звертайте увагу на запис `<T>` и `(T t)`, вважайте, що там `Object`.

Функціональний дескріптор інтерфейсу

```java
T -> boolean
```

Наступний код

```java
Predicate negative = i -> (int)i < 0;

System.out.println(negative.test(2));
System.out.println(negative.test(-2));
System.out.println(negative.test(0));
```

Виведе в консоль

```
false
true
false
```

Розглянемо реальний приклад, коли нам може знадобитися фунцкіональний інтерфейс `Predicate`.

Уявіть, що нам потрібно реалізувати метод фільтрації масиву цілих чисел. Якщо число задовольняє умові, то воно потрапляє до вихідного масиву, а числа, які не пройшли фільтр, відсіваються. Код цього методу може виглядати наступним чином

```java
public int[] filter(int[] input) {
    int[] result = new int[input.length];

    int counter = 0;
    for (int i : input) {
        if ( [перевірка_умови] ) {
            result[counter] = i;
            counter++;
        }
    }

    return Arrays.copyOfRange(result, 0, counter);
}
```

Особливість цього завдання полягає в тому, що нам надзвичайно бажано описати лише один метод, який міг би фільтрувати дані по-різному при тому чи іншому виклику методу.

Ми можемо реалізувати це за допомогою поліморфізму та механізму перевизначення методу, але такий код буде занадто громіздким і не забезпечить належної гнучкості коду.

Для нас було б надзвичайно доцільно передати в метод логіку, за допомогою якої необхідно фільтрувати вхідний масив. Оскільки логіка визначається блоком коду, то нам фактично потрібно передати у якості аргумента методу блок коду, який би визначав, чи задовольняє умові фільтра елемент масиву.

**Механизм функціональних інтерфейсів та лямбда-виразів дозволяє легко передавати поведінку як аргумент методу. За допомогою передачі поведінки, мі можемо писати гнучкі методи, робота яких може змінюватися в залежності від поведінки, яка передається до методу.**

Таку задачу можна реалізувати за допомогою функціонального інтерфейсу `Predicate`. Додамо посилання на об'єкт типу `Predicate` у якості вхідного аргументу функції та будемо викликати у цього об'єкту метод `test()`, передаючи йому елемент для фільтрації. Якщо метод поверне `true`, то елемент проходить крізь фільтр та додається до вихідного масиву.

```java
public int[] filter(int[] input, Predicate p) {
    int[] result = new int[input.length];

    int counter = 0;
    for (int i : input) {
        if (p.test(i)) {
            result[counter] = i;
            counter++;
        }
    }

    return Arrays.copyOfRange(result, 0, counter);
}
```

Тепер ми можемо передавати логіку порівняння за допомогою лямбда-виразу.

```java
int[] test = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Знаходимо парні числа
Predicate pr1 = o -> {
    int value = (int) o;
    return value % 2 == 0;
};

int[] res = filter(test, pr1);
System.out.println(Arrays.toString(res));

// Знаходимо числа, які діляться на 3 без остачі
Predicate pr2 = o -> {
    int value = (int) o;
    return value % 3 == 0;
};

res = filter(test, pr2);
System.out.println(Arrays.toString(res));
```

Додаток виведе в консоль наступне

```
[2, 4, 6, 8, 10]
[3, 6, 9]
```

### 2. Функціональний інтерфейс `Consumer`

`Consumer` - вбудований функціональний інтерфейс, доданий до Java 8 до пакету `java.util.function`. Він приймає деяке значення у якості аргументу та нічого не повертає

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

Функціональний дескріптор інтерфейсу

```java
T -> void
```

Інтерфейс `Consumer` використовується, якщо необхідно виконати певну дію з деяким об'єктом без повернення результату. Один з випадків використання цього інтерфейсу - вивід в поток виводу.

```java
Consumer consumer = (i) -> System.out.println(i);

void forEach(int[] input, Consumer action) {
    for (int i : input) {
        action.accept(i);
    }
}

...

int[] test = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
forEach(test, consumer);
```

### 3. Функціональний інтерфейс `Function`

`Function` - це вбудований функціональний інтерфейс, який був доданий у версії Java SE 8 до пакету `java.util.function`. Інтерфейс приймає значення одного типу у якості аргументу та повертає значення іншого типу.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

Функціональний дескріптор інтерфейсу

```java
T -> R
```

Даний інтерфейс часто використовується для перетворення одного значення в інше. В інтерфейсі описаний один абстрактний метод `apply()`, який приймає об'єкт одного типу та повертає об'єкт іншого (типи вхідного та вихідного об'єктів можуть бути різними або співпадати).

```java
int[] test = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

int[] result = processArray(test, function);

System.out.println(Arrays.toString(result));

...

Function function = o -> {
    int value = (int) o;
    return value * 10;
};

int[] processArray(int[] input, Function function) {
    int[] result = new int[input.length];

    for (int i = 0; i < input.length; i++)
        result[i] = (int) function.apply(input[i]);

    return result;
}
```

## Завдання на лабораторну роботу

### Завдання 1

1. Напишіть предикат, який повертає `true`, якщо число є простим.

### Завдання 2

1. Дан наступний клас `Student`. У випадку необхідності додайте потрібні гетери, сетери та конструктори.

```java
class Student {
    private String name;
    private String group;
    private int[] marks;
}
```

2. Напишіть метод фільтрації масиву студентів.
3. Перевірте роботу методу фільтрації на прикладі предикату, який відсіює студентів, які мають 1 та більше заборгованостей (оцінка менше 60 балів).

### Завдання 3

1. Напишіть метод фільтрації за двома умовами (два предикати). Елемент проходить через фільтр, якщо задовольняє двум умовам.

### Завдання 4

1. Напишіть інтерфейс `Consumer`, який приймає на вхід об'єкт типу `Student` та виводить в консоль рядок виду `ПРІЗВИЩЕ + ІМ'Я`. Створіть масив з кількох студентів та перевірте роботу функції `forEach()`

### Завдання 5

1. Напишіть метод, який приймає `Predicate` та `Consumer`. Дія в Consumer виконується тільки тоді, якщо умова в `Predicate` виконується. Створіть масив з цілих чисел, придумайте два лямбда-вирази та перевірте роботу цього методу.

### Завдання 6

1. Напишіть `Function`, який приймає на вхід ціле число n та повертає ціле число 2^n. Створіть масив з 10 цілих чисел та перевірте роботу методу.

### Завдання 7

1. Напишіть метод `stringify()`, який приймає на вхід масив цілих чисел від 0 до 9 та `Function`. Напишіть `Function`, який приймає на вхід ціле число від 0 до 9 та повертає його значення у вигляді рядку ("нуль", "один", "два", "три" і так далі). Створіть масив з 10 цілих чисел та перевірте роботу методу.

### Додаткове завдання

Перепишіть розроблений додаток `SortingList` з 6 лабораторної роботи таким чином, щоб він використовував механізм лямбда-виразів.