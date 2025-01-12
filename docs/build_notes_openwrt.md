Сборка для OpenWrt
==================

Данная инструкция описывает процесс сборки i2pd для платформы OpenWrt с использованием OpenWrt SDK. Убедитесь, что ваша целевая платформа и версия OpenWrt соответствуют требованиям.

Необходимые пакеты
------------------

Для сборки необходимы следующие инструменты и зависимости:

- **OpenWrt SDK** для вашей версии и архитектуры
- **Компиляторы и инструменты сборки** (gcc, g++, make, cmake и т.д.)
- **Библиотеки**:
  - [Boost 1.83.0](https://www.boost.org/)
  - [OpenSSL 3.4.0](https://www.openssl.org/)
  - [zlib 1.3.1](https://zlib.net/)
  - [MiniUPnP](https://github.com/miniupnp/miniupnp)

Подготовка исходников
---------------------

### 0. Подготовка и загрузка исходных кодов

1. Подключитесь к вашему устройству OpenWrt по SSH:
    ```bash
    ssh root@192.168.0.1
    ```

2. Проверьте информацию о версии OpenWrt и архитектуре:
    ```bash
    cat /etc/openwrt_release
    ```
    Пример вывода:
    ```
    DISTRIB_ID='OpenWrt'
    DISTRIB_RELEASE='22.03.3'
    DISTRIB_REVISION='r20028-43d71ad93e'
    DISTRIB_TARGET='ramips/mt7620'
    DISTRIB_ARCH='mipsel_24kc'
    DISTRIB_DESCRIPTION='OpenWrt 22.03.3 r20028-43d71ad93e'
    DISTRIB_TAINTS='no-all'
    ```

3. Загрузите OpenWrt SDK соответствующей версии и архитектуры:
    ```bash
    wget https://downloads.openwrt.org/releases/22.03.3/targets/ramips/mt7620/openwrt-sdk-22.03.3-ramips-mt7620_gcc-11.2.0_musl.Linux-x86_64.tar.xz
    tar xf openwrt-sdk-22.03.3-ramips-mt7620_gcc-11.2.0_musl.Linux-x86_64.tar.xz
    ```

4. Настройте переменные окружения для SDK:
    ```bash
    export PATH="$PWD/staging_dir/toolchain-mipsel_24kc_gcc-11.2.0_musl/bin/:$PATH"
    export STAGING_DIR="$PWD/staging_dir"
    export CXX=mipsel-openwrt-linux-g++
    export CC=mipsel-openwrt-linux-gcc
    export LD=mipsel-openwrt-linux-ld
    export AR=mipsel-openwrt-linux-ar
    export AS=mipsel-openwrt-linux-as
    export RANLIB=mipsel-openwrt-linux-ranlib
    ```

Сборка зависимостей
-------------------

### 1. Сборка Boost 1.83.0

**Примечание:** Убедитесь, что версия Boost соответствует требованиям i2pd. В случае несовместимости необходимо собрать i2pd со статическими библиотеками.

```bash
./bootstrap.sh
./b2 toolset=gcc-mipsel \
    link=static runtime-link=shared \
    --prefix=./install-mips \
    --with-system --with-filesystem --with-program_options \
    --layout=tagged \
    install \
    threading=multi \
    target-os=linux \
    address-model=32 architecture=mips \
    abi=sysv
```

### 2. Сборка OpenSSL 3.4.0

```bash
wget https://github.com/openssl/openssl/releases/download/openssl-3.4.0/openssl-3.4.0.tar.gz
tar xf openssl-3.4.0.tar.gz
cd openssl-3.4.0
./Configure linux-mips32 --prefix=$PWD/install-mips --openssldir=$PWD/install-mips '-Wl,-rpath,$(LIBRPATH)'
make -j$(nproc)
make install
```

### 3. Сборка zlib 1.3.1

```bash
wget https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
tar xf zlib-1.3.1.tar.gz
cd zlib-1.3.1
cmake . -DCMAKE_INSTALL_PREFIX='./install-mips'
make -j$(nproc)
make install
```

### 4. Сборка MiniUPnP

```bash
git clone https://github.com/miniupnp/miniupnp.git --depth 1
cd miniupnp
cmake . -DCMAKE_INSTALL_PREFIX='./install-mips'
make
make install
```

Сборка i2pd
-----------

1. Загрузите исходный код i2pd:
    ```bash
    git clone https://github.com/PurpleI2P/i2pd.git --depth 1
    cd i2pd/build
    ```

2. Настройте переменные окружения для сборки:
    ```bash
    export OPENWRT_SDK="<путь_к_openwrt_sdk>"
    export OPENSSL_LIB="<путь_к_openssl>"
    export MINIUPNP_LIB="<путь_к_miniupnp>"
    export BOOST_LIB="<путь_к_boost>"
    export ZLIB="<путь_к_zlib>"
    ```

3. Запустите CMake с необходимыми параметрами:
    ```bash
    cmake . \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_SYSTEM_NAME=Linux \
        -DCMAKE_C_COMPILER=mipsel-openwrt-linux-gcc \
        -DCMAKE_CXX_COMPILER=mipsel-openwrt-linux-g++ \
        -DCMAKE_FIND_ROOT_PATH="\
$OPENSSL_LIB;\
$OPENWRT_SDK;\
$BOOST_LIB/;\
$MINIUPNP_LIB" \
        -DCMAKE_SYSROOT="$STAGING_DIR/toolchain-mipsel_24kc_gcc-11.2.0_musl" \
        -DBOOST_ROOT=$BOOST_LIB/install-mips \
        -DZLIB_INCLUDE_DIR=$ZLIB/include \
        -DZLIB_LIBRARY=$ZLIB/lib/libz.a \
        -DCMAKE_FIND_ROOT_PATH_MODE_INCLUDE=ONLY \
        -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY \
        -DCMAKE_FIND_ROOT_PATH_MODE_PACKAGE=ONLY \
        -DWITH_BINARY=ON \
        -DWITH_LIBRARY=OFF \
        -DWITH_STATIC=ON \
        -DWITH_UPNP=ON \
        -DWITH_HARDENING=ON \
        -DOPENSSL_ROOT_DIR=$OPENSSL_LIB \
        -DOPENSSL_INCLUDE_DIR=$OPENSSL_LIB/include \
        -DOPENSSL_USE_STATIC_LIBS=TRUE
    ```

4. Запустите сборку:
    ```bash
    make -j$(nproc)
    ```

Установка
---------

После успешной сборки необходимо установить i2pd на ваше устройство OpenWrt.

1. Создайте необходимые директории и скопируйте конфигурационные файлы:
    ```bash
    ssh root@192.168.0.1 'mkdir /var/lib/i2pd/ -p'
    scp -r ../contrib/{i2pd.conf,tunnels.d,tunnels.conf,subscriptions.txt} root@192.168.0.1:/etc/i2pd/
    ```

2. Скопируйте скомпилированный бинарный файл i2pd на устройство:
    ```bash
    scp ./i2pd root@192.168.0.1:/bin/
    ```

3. Запустите i2pd на устройстве:
    ```bash
    ssh root@192.168.0.1 '/bin/i2pd'
    ```
