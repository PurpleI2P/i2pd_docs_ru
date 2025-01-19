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
$BOOST_LIB;\
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

# Сборка i2pd через собственный feed в OpenWrt

Данный способ позволяет интегрировать сборку i2pd в стандартный процесс сборки OpenWrt и получить итоговый пакет `.ipk`. После установки на роутер вы сможете управлять i2pd через `opkg`.

## 1. Подготовка окружения

1. **Загрузите исходники OpenWrt** (или используйте [OpenWrt SDK](https://openwrt.org/docs/guide-developer/using_the_sdk)).  
   Пример клонирования всего репозитория OpenWrt:
   ```bash
   git clone https://github.com/openwrt/openwrt.git /path/to/openwrt
   ```
   или же скачайте соответствующий SDK с [официального сайта](https://downloads.openwrt.org/).

2. **Перейдите в директорию с OpenWrt**:
   ```bash
   cd /path/to/openwrt
   ```

## 2. Создание структуры собственного фида

1. **Создайте папку для своего локального фида**. Это может быть, к примеру, `feeds/custom`. Внутри неё обычно делают ту же структуру, что и в стандартных фидах OpenWrt (в частности, пакетные директории, такие как `net`, `libs`, `utils` и т.д.):

   ```bash
   mkdir -p feeds/custom/net
   ```

2. **Создайте директорию для i2pd**:
   ```bash
   mkdir -p feeds/custom/net/i2pd
   ```

3. **Подготовьте Makefile пакета**. Основная “магия” сборки i2pd для OpenWrt происходит именно в `Makefile` пакета, который описывает, где взять исходники i2pd и как их собрать.  
   Пример (минимально необходимый) `feeds/custom/net/i2pd/Makefile`:

   ```makefile
   include $(TOPDIR)/rules.mk

   # Указываем, что для сборки используем CMake
   PKG_NAME:=i2pd
   PKG_VERSION:=2.45.0
   PKG_RELEASE:=1

   # Источник – официальный репозиторий на GitHub (тег или commit).
   # Можно также использовать MASTER_SITES и другие механизмы, но
   # GitHub-пример – наиболее распространён.
   PKG_SOURCE_PROTO:=git
   PKG_SOURCE_URL:=https://github.com/PurpleI2P/i2pd.git
   PKG_SOURCE_VERSION:=v2.45.0
   PKG_MIRROR_HASH:=skip

   # Говорим, что хотим собрать через cmake
   PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
   PKG_FIXUP:=cmake
   CMAKE_INSTALL:=1

   include $(INCLUDE_DIR)/package.mk
   include $(INCLUDE_DIR)/cmake.mk

   # Описываем, какие пакеты появятся в итоге
   define Package/i2pd
     SECTION:=net
     CATEGORY:=Network
     TITLE:=i2p Daemon (i2pd)
     URL:=https://github.com/PurpleI2P/i2pd
     DEPENDS:=+libstdcpp +libpthread +librt +boost +libopenssl +zlib
     # Если требуется MiniUPnP, добавляем:
     # DEPENDS+= +libminiupnpc
   endef

   define Package/i2pd/install
     $(INSTALL_DIR) $(1)/usr/sbin
     # Если i2pd собирается в bin/, нужно скопировать соответствующий бинарник:
     $(INSTALL_BIN) $(PKG_BUILD_DIR)/i2pd $(1)/usr/sbin/i2pd

     # Пример: скопировать пример конфигов (опционально), если нужно
     $(INSTALL_DIR) $(1)/etc/i2pd
     # Для простоты можно скопировать лишь базовый i2pd.conf
     # $(INSTALL_DATA) $(PKG_BUILD_DIR)/pkgdocs/i2pd.conf $(1)/etc/i2pd/
   endef

   $(eval $(call BuildPackage,i2pd))
   ```

   Важные моменты:
   - В `DEPENDS` указываем библиотеки, которые нужны i2pd (Boost, OpenSSL, Zlib и пр.).
   - Если планируется использовать UPnP, добавляем `+libminiupnpc` и при необходимости прописываем опцию `-DWITH_UPNP=ON` в `CMAKE_OPTIONS`.
   - Версия `PKG_VERSION` и `PKG_SOURCE_VERSION` (тег или commit) должны совпадать.

4. **Подключаем свой локальный фид** в `feeds.conf` или `feeds.conf.default`. Для этого добавляем строку:
   ```bash
   src-link custom /path/to/openwrt/feeds/custom
   ```
   или указываем полный путь к вашему внешнему репозиторию, если вы его храните отдельно.

## 3. Обновление фидов и установка i2pd

1. **Обновите фиды**:
   ```bash
   ./scripts/feeds update -a
   ```
2. **Установите пакет i2pd** (добавьте его в список сборки):
   ```bash
   ./scripts/feeds install i2pd
   ```
   Если всё прописано правильно, в выводе вы увидите, что пакет i2pd добавлен.

3. **Запустите `menuconfig`**:
   ```bash
   make menuconfig
   ```
   - Перейдите в раздел `Network` и найдите `i2pd`.
   - Нажмите `Y` или `M` (в зависимости от того, хотите ли вы встроить i2pd в прошивку или собрать как отдельный `.ipk`).

4. **Скомпилируйте i2pd**:
   ```bash
   make package/i2pd/compile V=s
   ```
   - При необходимости можно поставить галочку на “Verbose build” в `menuconfig` для более детализированного лога.
   - Если сборка пройдёт успешно, пакет i2pd появится в `bin/packages/<архитектура>/custom/` (либо в `bin/packages/<arch>/base/` — зависит от настройки `SECTION`).

## 4. Установка собранного пакета на устройство

1. **Скопируйте `.ipk` на роутер** (или другое устройство с OpenWrt):
   ```bash
   scp bin/packages/<arch>/custom/i2pd_<версия>_<arch>.ipk root@192.168.1.1:/tmp
   ```

2. **Установите через opkg**:
   ```bash
   ssh root@192.168.1.1
   opkg install /tmp/i2pd_<версия>_<arch>.ipk
   ```

3. **Проверьте**:
   ```bash
   i2pd --version
   ```
   Должно корректно отобразить версию i2pd.

## 5. Настройка и запуск

После установки i2pd через пакетный менеджер:
1. Создайте директории для данных, если это не делается автоматически:
   ```bash
   mkdir -p /var/lib/i2pd
   ```
2. Скопируйте или настройте конфигурационные файлы (при желании):
   ```bash
   mkdir -p /etc/i2pd
   # Можно перенести i2pd.conf, tunnels.conf и т.п. из репо i2pd
   # или использовать готовый init-скрипт, если он включён в пакет.
   ```
3. Запустите i2pd (или используйте init-скрипты, если вы их добавили):
   ```bash
   /usr/sbin/i2pd --conf=/etc/i2pd/i2pd.conf
   ```
   Либо:
   ```bash
   /etc/init.d/i2pd start
   ```
