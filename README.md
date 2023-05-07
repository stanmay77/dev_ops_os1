# Домашнее задание к занятию «Операционные системы. Лекция 1»

1. Какой системный вызов делает команда cd?

```
chdir("/tmp")
```

2. Попробуйте использовать команду file на объекты разных типов в файловой системе. Используя strace, выясните, где находится база данных file, на основании которой она делает свои догадки.

```
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
```

Команда file предназначена для определения типа файла. При вызове команды file происходит чтение первых нескольких байтов файла и определяет, соответствуют ли эти байты какому либо формату файла. Для определения типа файла используется файл магических чисел, который содержит информацию о типах файлов. Этот файл обычно находится в директории /usr/share/file/magic или /etc/magic.

3. Предположим, приложение пишет лог в текстовый файл. 

Этот файл оказался удалён (deleted в lsof), но сказать сигналом приложению переоткрыть файлы или просто перезапустить приложение возможности нет. Так как приложение продолжает писать в удалённый файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков, предложите способ обнуления открытого удалённого файла, чтобы освободить место на файловой системе.

Для примера:

```
exec 10> test.log
ping ya.ru >&10
rm test.log
```

Чтобы проверить какой PID использует удаленный файл используем команду:
```
sudo lsof | grep deleted
```

Чтобы отобразить все процессы, пишущие в 5 дескриптор можно использовать команду:

```
sudo lsof -w -d 5
```

И наконец обнуляем файл:

```
echo "" | tee /proc/<PID нашего процесса>/fd/5
```

4. Занимают ли зомби-процессы ресурсы в ОС (CPU, RAM, IO)?

Зомби-процессы не потребляют ресурсы ЦП, ОЗУ или ввода-вывода (IO), так как они не выполняются. Они являются процессами-оболочками, которые уже завершили свою работу, но не были полностью удалены из системы.

5. В IO Visor BCC есть утилита opensnoop:

root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc

На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools для Ubuntu 20.04

```
/var/run/utmp, /usr/local/share/dbus-1/system-services, /usr/share/dbus-1/system-services, /lib/dbus-1/system-services, /var/lib/snapd/dbus-1/system-services/
```

6. Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc и где можно узнать версию ядра и релиз ОС.

```
man 2 uname

Part of the utsname information is also accessible  via  /proc/sys/ker‐
       nel/{ostype, hostname, osrelease, version, domainname}.
```

7. Чем отличается последовательность команд через ; и через && в bash? Например:

root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#

; - последовательноe выполенение
&& - выполнение только если  команда до && завершилась успешно

В примере, если каталог /tmp/some_dir не существует, то в первом случае выполнится вывод "Hi", а во втором вывод не появится.

set -e позволяет немедленно завершить скрипт, если в нем произошла ошибка. Использование "&&" тогда часто не имеет смысла. Для примера, конструкция "command1 && command2 && command3" в скрипте с set -е, и cmd1 завершается неудачно, то команды cmd2 и cmd3 не будут выполнены.

8. Из каких опций состоит режим bash set -euxo pipefail, и почему его хорошо было бы использовать в сценариях?

Опции в Bash:
```
-e - останавливает выполнение скрипта, если любая команда возвращает ненулевой код возврата.
-u - останавливает выполнение скрипта, если была попытка использовать неопределенную переменную.
-x - выводит на экран каждую выполняемую команду перед ее выполнением.
-o pipefail - возвращает код ошибки последней неудачной команды в цепочке команд, объединенных оператором |.
```

Позволяет отлаживать скрипты - останавливает скрипт в случае ошибок и выводит инфо для отладки

9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps изучите (/PROCESS STATE CODES), что значат дополнительные к основной заглавной букве статуса процессов. Его можно не учитывать при расчёте (считать S, Ss или Ssl равнозначными).
```
38 I - в режиме ожидания ввода/вывода, не заблокированы и не выполняются
66 I< - в режиме ожидания ввода/вывода, заблокированы и не выполняются
1 R+ - в состоянии запуска или остановки
109 S - выполняющиеся
2 S+ - выполняющиеся в привилегированном режиме
2 S< - в режиме ожидания ввода/вывода, заблокированы и выполняются
2 SN - приостановлены, могут продолжить выполнение
```