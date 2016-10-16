Сборка на Unix системах
=======================

Этот документ описывает сборку на системах:
* [Debian/Ubuntu](#debianubuntu) (содержит инструкцию для сборки пакетов)
* [Fedora/Centos](#fedoracentos)
* [FreeBSD](#freebsd)

Убедитесь, что все зависимости в вашей системе удовлетворены.

Если это так, то приступаем к сборке i2pd.
Клонируем репозиторий и собираем:
```bash
git clone https://github.com/PurpleI2P/i2pd.git
cd i2pd/build
cmake -DCMAKE_BUILD_TYPE=Release # more options could be passed, see "CMake Options"
make                             # you may add VERBOSE=1 to cmdline for debugging
```

После сборки i2pd можно установить в систему следующей командой:
```bash
make install
```

Вы так же можете использовать упрощенный вариант сборки:
```bash
git clone https://github.com/PurpleI2P/i2pd.git
cd i2pd
make
```

Debian/Ubuntu
-------------

Устанавливаем компилятор и прочие программы для сборки:
```bash
sudo apt-get install build-essential
```

Устанавливаем библиотеки разработчиков для сборки:
```bash
sudo apt-get install \
    libboost-chrono-dev \
    libboost-date-time-dev \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libssl-dev
```

Если нужна поддержка UPnP (не забудьте потом запустить CMake с параметром `WITH_UPNP=ON`):
```bash
sudo apt-get install libminiupnpc-dev
```

Вы можете собрать пакет .deb следующим образом:
```bash
sudo apt-get install fakeroot devscripts
cd i2pd
debuild --no-tgz-check
```

Fedora/Centos
-------------

Устанавливаем компилятор и прочие программы для сборки:
```bash
sudo yum install make cmake gcc gcc-c++
```

*В новых версиях Fedora используется [DNF](https://en.wikipedia.org/wiki/DNF_(software)) вместо YUM by по-умолчанию, можете использовать его*

> *В Centos 7 есть только CMake 2.8.11 в официальных репозиториях, это слишком старая версия для i2pd. Нужен CMake версии 2.8.12 или новее*
> Вы можете собрать CMake для Centos вручную (ВНИМАНИЕ, там много зависимостей для сборки):
> ```bash
> wget https://kojipkgs.fedoraproject.org/packages/cmake/2.8.12/3.fc21/src/cmake-2.8.12-3.fc21.src.rpm
> yum-builddep cmake-2.8.12-3.fc21.src.rpm
> rpmbuild --rebuild cmake-2.8.12-3.fc21.src.rpm
> yum install ~/rpmbuild/RPMS/x86_64/cmake-2.8.12-3.el7.centos.x86_64.rpm
> ```

Устанавливаем библиотеки разработчиков для сборки:
```bash
sudo yum install boost-devel openssl-devel
```

Если нужна поддержка UPnP (не забудьте потом запустить CMake с параметром `WITH_UPNP=ON`):
```bash
sudo yum install miniupnpc-devel
```

MAC OS X
--------

Необходим homebrew.

Устанавливаем библиотеки разработчиков для сборки:
```bash
brew install libressl boost
```

Собираем:
```bash
make HOMEBREW=1
```


FreeBSD
-------

Для 10.X используйте clang. Вам так же понабятся порты boost и openssl.
Запустите gmake, он прочитает Makefile.bsd и сделает необходимые изменения.

Ветка 9.X использует gcc v4.2 который не поддерживает необходимый стандарт c++11.

Необходимые порты:

* `devel/cmake`
* `devel/boost-libs`
* `lang/gcc47`(или более новые версии)

Чтобы использовать более новый компилятор, установите следующие переменные (замените "47" своей версией gcc):
```bash
export CC=/usr/local/bin/gcc47
export CXX=/usr/local/bin/g++47
```

Опции CMake
-----------

Доступные опции CMake (подробности смотрите в `man 1 cmake`):

* `CMAKE_BUILD_TYPE` профиль сборки, отладочный или релиз (Debug/Release)
* `WITH_BINARY`      сборка самого i2pd
* `WITH_LIBRARY`     сборка библиотеки libi2pd
* `WITH_STATIC`      сборка статических версий библиотеки и самого i2pd
* `WITH_UPNP`        сборка с поддержкой UPnP (нужна библиотека libupnp)
* `WITH_AESNI`        сборка с поддержкой AES-NI (ON/OFF)
* `WITH_HARDENING`   включить Hardending (ON/OFF) (только с gcc)
* `WITH_PCH`         использовать pre-compiled header (экспериментально, ускоряет процесс сборки)

Так же у CMake есть -L флаг, который показывает список текущих установленных опций:
```bash
cmake -L
```
