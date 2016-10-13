Сборка для Android
==================

Существуют 2 версии: с использованием QT и без него

Необходимые пакеты
------------------

Требуются Android SDK, NDK и QT с поддержкой андроида (для QT).  

- [SDK](https://developer.android.com/studio/index.html) (выбрать command line tools only)  
- [NDK](https://developer.android.com/ndk/downloads/index.html)  
- [QT](https://www.qt.io/download-open-source/)(for QT only). Выбрать версию для андороида для вашей платформы. Например QT 5.6 под лиункс будет [this file](http://download.qt.io/official_releases/qt/5.6/5.6.1-1/qt-opensource-linux-x64-android-5.6.1-1.run  )

Также потребуется  Java JDK и Ant.

QT-Creator (только для QT)
------------------------
Запустить QT-creator, который должен быть установлен вместе с  QT.  
Идти в  Settings/Anndroid и указать пути к SDK and NDK.
Если все сделано правильно то должно появиться:  
Android for armeabi-v7a (gcc, qt) и Android for x86 (gcc, qt).

Библиотеки
--------------
Следущие собранные библиотеки следует взять и репозитария PurpleI2P.  
```bash
git clone https://github.com/PurpleI2P/Boost-for-Android-Prebuilt.git  
git clone https://github.com/PurpleI2P/OpenSSL-for-Android-Prebuilt.git  
git clone https://github.com/PurpleI2P/MiniUPnP-for-Android-Prebuilt.git  
git clone https://github.com/PurpleI2P/android-ifaddrs.git  
```


Сборка с  QT
------------------------
- Открыть qt/i2pd_qt/i2pd_qt.pro в QT-creator   
- Поменять строчку MAIN_PATH = /path/to/libraries на директорию где лежать зависимости из предыдущего пункта   
- Выбрать проект для целевой платформы (как правило armeabi-v7a) и собрать    
- .apk файл появится в android-build/bin     

Сборка без QT
---------------------------
- Поменять строку I2PD_LIBS_PATH в android/jni/Application.mk на директорию с зависимостями  
- Запустить команду 'ndk-build -j4' из директории andorid  
- Поправить файл 'local.properties'. Строчки 'sdk.dir=<путь к SDK>' и 'ndk.dir=<путь к NDK>'  
- Run 'ant clean debug'

Релизный .apk
--------------
Релизный .apk следует подписать своей подписью для чего требуется Java keystore file(.jks). Либо взять уже существующий, либо сделать новый с помощью keytool либо сконвертировать какой нибудь из своих сертификатов. Например i2pd пидписан этим [сертификатом](https://github.com/PurpleI2P/i2pd/blob/openssl/contrib/certificates/router/orignal_at_mail.i2p.crt).  
Создать файл 'ant.propeties'  
key.store='путь к keystore file'  
key.alias='alias name'  
key.store.password='keystore password'  
key.alias.password='alias password'   
Запустить 'ant clean release'
