Сборка на Unix системах
=======================

Этот документ описывает сборку на системах:
* [Debian/Ubuntu](#debian-ubuntu) (содержит инструкцию для сборки пакетов)
* [Fedora/Centos](#fedora-centos)
* [MAC OS X](#mac-os-x)
* [FreeBSD](#freebsd)

Убедитесь, что все зависимости в вашей системе удовлетворены. Смотрите [эту](https://i2pd.readthedocs.io/en/latest/devs/building/requirements/ "Build requirements") страницу с общими требованиями.

Если требования удовлетворены, приступаем к сборке i2pd!
Клонируем репозиторий и начинаем сборку:
```bash
git clone https://github.com/PurpleI2P/i2pd.git
```
В общем случае, процесс сборки выглядит так (с cmake):
```bash
cd i2pd/build
cmake <cmake options> . # смотрите раздел "Опции CMake" ниже
make                    # можно добавить опцию VERBOSE=1 для отладки
```
...или быстрый и грязный путь:
```bash
cd i2pd/
make
```
После успешной сборки i2pd можно установить в систему следующей командой:
```bash
make install
```

Опции СMake
-----------

Доступные опции CMake (каждая форма имеет форму: `-D<key>=<value>`, подробности смотрите в `man 1 cmake`):

* `CMAKE_BUILD_TYPE` профиль сборки, отладочный или релиз (Debug/Release, по умолчанию: без оптимизации и отладочных символов)
* `WITH_BINARY`          сборка самого i2pd (по-умолчанию: включено (ON))
* `WITH_LIBRARY`         сборка библиотеки libi2pd (по-умолчанию: включено (ON))
* `WITH_STATIC`          сборка статических версий библиотеки и самого i2pd (по-умолчанию: отключено (OFF))
* `WITH_UPNP`            сборка с поддержкой UPnP (нужна библиотека libupnp, по-умолчанию: отключено (OFF))
* `WITH_AESNI`           сборка с поддержкой AES-NI (ON/OFF, по-умолчанию: отключено (OFF))
* `WITH_HARDENING`       включить Hardending (ON/OFF) (только с gcc, по-умолчанию: отключено (OFF))
* `WITH_PCH`             использовать pre-compiled header (экспериментально, ускоряет процесс сборки, по-умолчанию: отключено (OFF))
* `WITH_I2LUA`           используется для сборки i2lua (по-умолчанию: отключено (OFF))
* `WITH_WEBSOCKETS`      задействовать websocket server (по-умолчанию: отключено (OFF))
* `WITH_AVX`             сборка с поддержкой AVX (по-умолчанию: отключено (OFF))
* `WITH_MESHNET`         сборка для сети cjdns (по-умолчанию: отключено (OFF))
* `WITH_ADDRSANITIZER`   сборка с Address Sanitizer (по-умолчанию: отключено (OFF))
* `WITH_THREADSANITIZER` сборка с Thread Sanitizer (по-умолчанию: отключено (OFF))

Так же у CMake есть флаг -L, который может быть использован для отображения списока текущих установленных опций:
```bash
cmake -L
```

Debian/Ubuntu
-------------

Устанавливаем компилятор и прочие средства для сборки:
```bash
sudo apt-get install build-essential
```

Так же вам понядобятся библиотеки разработчиков:
```bash
sudo apt-get install \
    libboost-date-time-dev \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-system-dev \
    libssl-dev \
    zlib1g-dev
```

Если нужна поддержка UPnP, должна быть установлена библиотека miniupnpc (не забудьте потом запустить CMake с параметром `WITH_UPNP=ON`):
```bash
sudo apt-get install libminiupnpc-dev
```

Вы можете собрать deb-пакет следующим образом:
```bash
sudo apt-get install fakeroot devscripts dh-apparmor
cd i2pd
debuild --no-tgz-check -us -uc -b
```

Fedora/Centos
-------------

Устанавливаем компилятор и прочие средства для сборки:
```bash
sudo yum install make cmake gcc gcc-c++
```

*В новых версиях Fedora используется [DNF](https://en.wikipedia.org/wiki/DNF_(software)) вместо YUM by по-умолчанию, можете использовать его*

> *В Centos 7 есть только CMake 2.8.11 в официальных репозиториях, это слишком старая версия для i2pd. Нужен CMake версии 2.8.12 или новее.*
> 
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

Необходим [homebrew](https://brew.sh).

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

Для 10.X используйте clang. Вам так же понабятся порты devel/boost-libs, security/openssl и devel/gmake.
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
