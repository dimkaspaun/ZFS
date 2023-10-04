## 1) Что нужно сделать?

### Определить алгоритм с наилучшим сжатием. Зачем: отрабатываем навыки работы с созданием томов и установкой параметров. Находим наилучшее сжатие.

**Смотрим список всех дисков, которые есть в виртуальной машине: lsblk**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_1.png)


**Создадим ещё 3 пула: 
Смотрим информацию о пулах: zpool list**


![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_2.png)



**Команда zpool status показывает информацию о каждом диске, состоянии сканирования и об ошибках чтения, 
записи и совпадения хэш-сумм. Команда zpool list показывает информацию о размере пула, количеству 
занятого и свободного места, дедупликации и т.д.**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_3.png)

**Добавим разные алгоритмы сжатия в каждую файловую систему** 

**Проверим, что все файловые системы имеют разные методы сжатия**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_4.png)

**Скачаем один и тот же текстовый файл во все пулы:**

    [root@zfs vagrant]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done 
    
    --2023-10-04 01:42:02--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log 
    
    Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47 
    
    Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected. HTTP request sent, awaiting response... 200 OK 
    
    Length: 40979797 (39M) [text/plain] Saving to: '/otus1/pg2600.converter.log' 
    
    100%[======================================>] 40,979,797  1.27MB/s   in 23s  
    
    2023-10-04 01:42:28 (1.69 MB/s) - '/otus1/pg2600.converter.log' saved [40979797/40979797] 
    
    --2023-10-04 01:42:28--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log Resolving gutenberg.org (gutenberg.org)... 
    
    152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47 
    
    Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected. 
    
    HTTP request sent, awaiting response... 200 OK 
    
    Length: 40979797 (39M) [text/plain] Saving to: '/otus2/pg2600.converter.log' 
    
    100%[======================================>] 40,979,797  1.70MB/s   in 23s     
    
    2023-10-04 01:42:52 (1.72 MB/s) - '/otus2/pg2600.converter.log' saved [40979797/40979797] 
    
    --2023-10-04 01:42:52--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log Resolving gutenberg.org (gutenberg.org)... 
    
    152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47 Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected. 
    
    HTTP request sent, awaiting response... 200 OK 
    
    Length: 40979797 (39M) [text/plain] Saving to: '/otus3/pg2600.converter.log' 
    
    100%[======================================>] 40,979,797  1.45MB/s   in 32s    
    
    2023-10-04 01:43:24 (1.24 MB/s) - '/otus3/pg2600.converter.log' saved [40979797/40979797] 
    
    --2023-10-04 01:43:24--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log 
    
    Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47 
    
    Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected. 
    
    HTTP request sent, awaiting response... 200 OK Length: 40979797 (39M) [text/plain] Saving to: '/otus4/pg2600.converter.log' 

    100%[======================================>] 40,979,797  2.00MB/s   in 21s     
    
    2023-10-04 01:43:46 (1.86 MB/s) - '/otus4/pg2600.converter.log' saved [40979797/40979797] 

**Проверим, что файл был скачан во все пулы:** 

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_5.png)

**Проверим, сколько места занимает один и тот же файл в разных пулах и проверим степень сжатия файлов:**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/1_6.png)

**Таким образом, у нас получается, что алгоритм gzip-9 самый эффективный по сжатию.**

## 2.	 Определение настроек пула

**Скачиваем архив в домашний каталог:**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/2_1.png)

**Сделаем импорт данного пула к нам в ОС:**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/2_2.png)

**Запрос сразу всех параметром файловой системы: zfs get all otus**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/2_3.png)
![](https://github.com/dimkaspaun/ZFS/blob/master/screen/2_4.png)

**Размер: zfs get available otus**

**Тип: zfs get readonly otus**

**Значение recordsize: zfs get recordsize otus**

**Тип сжатия (или параметр отключения): zfs get compression otus**

**Тип контрольной суммы: zfs get checksum otus**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/2_5.png)


## 3.	Работа со снапшотом, поиск сообщения от преподавателя

**Скачаем файл, указанный в задании:**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/3_1.png)

**Восстановим файловую систему из снапшота: zfs receive otus/test@today < otus_task2.file**

**Далее, ищем в каталоге /otus/test файл с именем “secret_message”:**

![](https://github.com/dimkaspaun/ZFS/blob/master/screen/3_2.png)
