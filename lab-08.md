<div align="center">

**МІНІСТЕРСТВО ОСВІТИ І НАУКИ УКРАЇНИ**  
Заклад фахової передвищої / вищої освіти  
Циклова комісія / кафедра інформаційних технологій

# ЛАБОРАТОРНА РОБОТА № 8

**Запис і читання файлів системними викликами `open()`, `read()`, `write()`, `close()`**

Тема: Створення програми для запису в файл та читання з файла з використанням `open()`, `read()`, `write()`, `close()`

Виконав: студент групи ІПЗ-33  
**Юрдамян Артур Торосович**  
Перевірив: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

2026

</div>

---

## Мета роботи

Навчитися відкривати, читати, записувати та закривати файли за допомогою системних викликів `open()`, `read()`, `write()`, `close()`, контролюючи значення їх повернення.

## Теоретичні відомості

`open()` повертає файловий дескриптор або -1 при помилці. `read()` повертає кількість прочитаних байтів, 0 означає кінець файла, а -1 — помилку. `write()` повертає кількість записаних байтів або -1 при помилці. `close()` закриває дескриптор. Права доступу при створенні файла задаються вісімковим числом і обмежуються поточною маскою umask.

## Завдання 1. Створення файла та запис тексту

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/stat.h>

int main(void) {
    const char text[] = "Hello, world!\n";
    const size_t length = sizeof(text) - 1;

    umask(0);
    int fd = open("myfile", O_WRONLY | O_CREAT | O_TRUNC, 0666);
    if (fd < 0) {
        perror("open");
        return EXIT_FAILURE;
    }

    ssize_t written = write(fd, text, length);
    if (written < 0 || (size_t)written != length) {
        perror("write");
        close(fd);
        return EXIT_FAILURE;
    }

    if (close(fd) < 0) {
        perror("close");
        return EXIT_FAILURE;
    }

    printf("У файл myfile записано %zd байтів.\n", written);
    return EXIT_SUCCESS;
}
```

## Завдання 2. Читання файла myfile та виведення на екран

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void) {
    char buffer[256];
    int fd = open("myfile", O_RDONLY);
    if (fd < 0) {
        perror("open");
        return EXIT_FAILURE;
    }

    ssize_t count;
    while ((count = read(fd, buffer, sizeof(buffer))) > 0) {
        ssize_t offset = 0;
        while (offset < count) {
            ssize_t n = write(STDOUT_FILENO, buffer + offset,
                              (size_t)(count - offset));
            if (n < 0) {
                perror("write");
                close(fd);
                return EXIT_FAILURE;
            }
            offset += n;
        }
    }

    if (count < 0) {
        perror("read");
        close(fd);
        return EXIT_FAILURE;
    }
    if (close(fd) < 0) {
        perror("close");
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

## Творче завдання 3. Копіювання myfile у myfile1

```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void) {
    char buffer[256];
    int in = open("myfile", O_RDONLY);
    if (in < 0) {
        perror("open myfile");
        return EXIT_FAILURE;
    }

    int out = open("myfile1", O_WRONLY | O_CREAT | O_TRUNC, 0666);
    if (out < 0) {
        perror("open myfile1");
        close(in);
        return EXIT_FAILURE;
    }

    ssize_t count;
    while ((count = read(in, buffer, sizeof(buffer))) > 0) {
        ssize_t offset = 0;
        while (offset < count) {
            ssize_t n = write(out, buffer + offset, (size_t)(count - offset));
            if (n < 0) {
                perror("write");
                close(in);
                close(out);
                return EXIT_FAILURE;
            }
            offset += n;
        }
    }

    if (count < 0) {
        perror("read");
        close(in);
        close(out);
        return EXIT_FAILURE;
    }

    if (close(in) < 0) {
        perror("close input");
        close(out);
        return EXIT_FAILURE;
    }
    if (close(out) < 0) {
        perror("close output");
        return EXIT_FAILURE;
    }

    puts("Файл myfile успішно скопійовано в myfile1.");
    return EXIT_SUCCESS;
}
```

## Порядок виконання та перевірка

1. Зберегти код завдання 1 у файл `write_file.c`.

2. Скомпілювати: `gcc write_file.c -o write_file`, потім запустити `./write_file`.

3. Перевірити запис командою `cat myfile`.

4. Зберегти код завдання 2 у файл `read_file.c`, скомпілювати `gcc read_file.c -o read_file` та запустити `./read_file`.

5. Зберегти код творчого завдання у файл `copy_file.c`, скомпілювати `gcc copy_file.c -o copy_file` та запустити `./copy_file`.

6. Перевірити результат копіювання командою `cat myfile1` або `cmp myfile myfile1`.

Очікуваний результат: після завдання 1 файл `myfile` містить рядок Hello, world!; завдання 2 виводить цей рядок на екран; завдання 3 створює `myfile1` з таким самим вмістом. Фактичний вивід варто підтвердити запуском програм у Linux.

## Контрольні запитання та відповіді

**1. Яке призначення `open()` і `close()`?**

`open()` відкриває файл і повертає файловий дескриптор; `close()` звільняє дескриптор і завершує роботу з ним.

**2. Які особливості `read()` і `write()`?**

`read()` копіює байти з файла у буфер, `write()` — з буфера до файла або іншого дескриптора. Обидві функції повертають фактичну кількість байтів, тому результат потрібно перевіряти.

**3. Які прапори використовують з `open()`?**

Зокрема `O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_TRUNC` та `O_APPEND`. Прапори режиму доступу взаємовиключні, інші прапори комбінуються побітовим OR.

**4. Що повертає `open()`?**

Невід’ємний файловий дескриптор у разі успіху або -1 у разі помилки.

**5. Що повертають `read()` і `write()`?**

Кількість фактично прочитаних або записаних байтів; -1 означає помилку. Для `read()` нуль означає кінець файла.

**6. Чи можна використовувати `sizeof()` для визначення розміру буфера?**

Так. sizeof(array) повертає розмір масиву в байтах у межах того контексту, де це справді масив; для вказівника `sizeof()` повертає розмір самого вказівника.

**7. Для чого потрібна `fcntl.h`?**

Заголовок містить оголошення та константи для керування файловими дескрипторами, зокрема прапори `open()`.

**8. Який параметр `read()` визначає адресу буфера?**

Другий параметр — вказівник на буфер, у який `read()` записує прочитані дані.

**9. Який параметр `write()` визначає адресу даних?**

Другий параметр — вказівник на буфер, з якого `write()` бере дані для запису.

**10. Скільки параметрів має `close()`?**

Один параметр — файловий дескриптор.

**11. Для чого використовують umask(0)?**

umask(0) обнуляє маску створення файлів для поточного процесу. Фактичні права все одно залежать від файлової системи та інших обмежень.

**12. Які прапори відкриття та створення файла відомі?**

`O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_EXCL`, `O_TRUNC`, `O_APPEND`.

**13. Як формуються права доступу?**

Права задаються трьома групами: власник, група та інші користувачі; у кожній групі біти читання, запису та виконання мають значення 4, 2 і 1.

**14. Якою операцією комбінують вісімкові коди прав доступу?**

Вісімковий запис — це подання бітової маски прав. Для комбінування окремих прапорів у коді застосовують побітове OR (|).

**15. Як комбінуються прапори `open()`?**

Прапори комбінують оператором побітового OR, наприклад `O_WRONLY` | `O_CREAT` | `O_TRUNC`.

## Висновок

У роботі реалізовано створення та запис файла, читання даних і копіювання вмісту в інший файл через `open()`, `read()`, `write()` і `close()`. Перевірка значень повернення дає змогу виявляти помилки й коректно завершувати роботу з файлами.
