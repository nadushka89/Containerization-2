# Контейнеризация семинары №2


![](source\hero.avif)

## Урок 2. Механизмы контрольных групп

Задание 1:
1) запустить контейнер с ubuntu, используя механизм LXC
2) ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает

Задание 2*: настроить автоматическую маршрутизацию между контейнерами. Адреса можно взять: 10.0.12.0/24 и 10.0.13.0/24. e.x. sudo lxc-create -n test123 -t ubuntu -f /usr/share/doc/lxc/examples/lxc-veth.conf

Задание со звездочкой - повышенной сложности, это нужно учесть при выполнении (но сделать его необходимо).

**Задание:**

*  Установим cgroup-tools, вводим команду:
```
sudo apt install cgroup-tools
```

![cgroup-tools](source\cgroup-tools.png)

Для демонстрации пространства имен PID - используем утилиту unshare Linux для запуска программы в новом наборе пространств имен.

Запуск 
```
sudo unshare --pid --fork --mount-proc /bin/bash
```
предоставит нам оболочку bash в новом пространстве имен PID.

![unshare](source\unshare.png) 

Cоздадим группу под названием mytestgroup и посмотрим какие там файлы:

```
cgcreate -a $USER -g memory:mytestgroup -g cpu:mytestgroup
ls /sys/fs/cgroup/mytestgroup
```

![mytestgroup](source\mytestgroup.png)

Считываем данные из файла, в котором уграничивается потребление памяти контейнером:
```
cat /sys/fs/cgroup/mytestgroup/memory.max
```
```
cgexec -g memory:mytestgroup bash
```

Установливаем необходимые пакеты через пакетный менеджер, вводим команду:

```
sudo apt-get install lxc debootstrap bridge-utils lxc-templates
```
![templates](source\templates.png)

Cоздает контейнер с именем test123 из образа ubuntu:
```
sudo lxc-create -n test-gb-2 -t ubuntu
```
![test-gb-2](source\test-gb-2.png)

Запускает контейнер:

```
sudo lxc-start -n test-gb-2
```

Чтобы убедиться, что контейнер действительно создался и работает, подключаемся к нему:
```
sudo lxc-attach -n test-gb-2
```

Вводим следующую команду для просмотра выделенной и свободной памяти:
```
free -m
```
Выходим из контейнера:
```
exit
```
![free](source\free-m.png)

Для того, что бы ограничить потребление памяти контейнером, необходимо:

```
sudo lxc-cgroup -n test-gb-2 memory.max 256M
```
Таким образом мы ограничили потребление памяти на более 256 мб.
Останавливаем контейнер и снова запускаем его, что ограничение работает:
![memory_max_](source\memory_max_.png)

Все работает.


* Далее просмотрим список имеющихся в нашей системе контейнеров:
```
sudo lxc-ls
```

* Для удаления контейнера необходмио воспользоваться следующей командой:
```
lxc-destroy -n test-gb-2
```

