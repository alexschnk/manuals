## Обязательные настройки 
---
##### Редактрование /etc/fstab

1) Ускорение SSD

rw,lazytime,ssd,space_cache=v2, max_inline=256,commit=600,subvol

2) Кеш програм в ОЗУ

tmpfs    /путь    tmpfs    defaults,noatime,mode=1777    0 0

/tmp                                                           

/var/tmp

/var/log                                

/home/arch (имя пользователя)/.cache                       

/home/arch (имя пользователя)/Загрузки                     

/var/cache/dnf         

/home/arch (имя пользователя)/.var/app/org.kde.kdenlive/cache              
                                     
/home/arch (имя пользователя)/.var/app/org.telegram.desktop/data/TelegramDesktop/tdata/user_data        
        
/home/arch (имя пользователя)/.var/app/org.telegram.desktop/data/TelegramDesktop/tdata/temp_data

/home/arch (имя пользователя)/.var/app/org.mozilla.firefox/cache

##### Настройка переменных окружения
sudo nano /etc/environment

Включение OpenGL
MESA_GL_VERSION_OVERRIDE=4.5
MESA_GLSL_VERSION_OVERRIDE=450
__GL_THREADED_OPTIMIZATIONS=1

##### Отключение дампов ядра

/etc/systemd/coredump.conf

в разделе [Coredump]
Storage = none

sudo systemctl daemon-reload

##### Повышение лимитов

sudo nano /etc/systemd/system.conf
sudo nano /etc/systemd/user.conf

DefaultLimitNOFILE=2000000

sudo nano /etc/security/limits.conf 
username hard nofile 2000000

sudo systemctl daemon-reload

##### Настройка sysctl

sudo modprobe tcp_bbr

sudo nano /etc/modules-load.d/tcp-bbr.conf
tcp_bbr

sudo nano /etc/sysctl.d/99-sysctl.conf
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=cake
vm.vfs_cache_pressure=30

##### Добавление параметров ядра

sudo grubby --update-kernel=ALL --args="loglevel=3 ipv6.disable=1 raid=noautodetect rootfstype=btrfs  page_alloc.shuffle=1 split_lock_detect=off nowatchdog mitigations=off pci=pcie_bus_perf"

sudo grubby --update-kernel=ALL --remove-args="foo=bar"

reboot

elevator=noop: планировщик SSD

ipv6.disable=1: отключить ipv6

lpj=: уникальный параметр для каждой системы. Его значение автоматически определяется во время загрузки, что довольно трудоемко, поэтому лучше задать вручную. Определить ваше значение для lpj можно через следующую команду: sudo dmesg | grep "lpj="

raid=noautodetect: отключает проверку на RAID во время загрузки. Если вы его используете - НЕ прописывайте данный параметр.

rootfstype=btrfs: здесь указываем название файловой системы в которой у вас отформатирован корень.

page_alloc.shuffle=1: этот параметр рандомизирует свободные списки распределителя страниц. Улучшает производительность при работе с ОЗУ с очень быстрыми накопителями (NVMe, Optane). Подробнее тут.

split_lock_detect=off: отключаем раздельные блокировки шины памяти. Одна инструкция с раздельной блокировкой может занимать шину памяти в течение примерно 1 000 тактов, что может приводить к кратковременным зависаниям системы.

pci=pcie_bus_perf: увеличивает значение Max Payload Size (MPS) для родительской шины PCI Express. Это даёт лучшую пропускную способность, т. к. некоторые устройства могут использовать значение MPS/MRRS выше родительской шины. Больше подробностей здесь (англ.):

nowatchdog - отключает сторожевые таймеры. Позволяет избавиться от заиканий в онлайн играх.

mitigations=off: - непосредственно отключает все заплатки безопасности ядра (включая Spectre и Meltdown)

https://unix.stackexchange.com/questions/684623/pcie-bus-perf-understanding-the-capping-of-mrrs

https://www.programmersought.com/article/74187399630/

##### Отключение ZRAM 

sudo dnf remove zram-generator zram-generator-defaults

reboot

##### Подключение репозиториев

1) RPM Fusion

sudo dnf install --nogpgcheck https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

2) Подключение Flathub

sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

##### Установка пакетов 

- Драйвера
sudo dnf install libva-intel-driver intel-media-driver intel-gpu-firmware

- Пакеты rpm
sudo dnf install git p7zip f2fs-tools ntfs-3g kdiskmark  htop unrar unzip unace squashfs-tools spectacle android-tools unrar timeshift

- Пакеты flatpak
flatpak install flathub  org.telegram.desktop org.libreoffice.LibreOffice io.gitlab.librewolf-community
org.qbittorrent.qBittorrent com.github.tchx84.Flatseal

## Факультативные настройки 
---
##### Установка KVM

sudo dnf group install Virtualization

reboot

##### Рандомизация MAC адреса

1) Генерирование уникального псевдослучайного MAC адреса для каждого соединения при загрузке системы (параметр stable). Это избавит от проблем с переподключением к публичным хот-спотам и небходимости повторно проходить аутентификацию в captive-порталах;

2) Генерирование уникального псевдослучайного MAC адреса для каждого соединения при каждом переподключении (параметр random). Наиболее безопасно, но может вызывать описанные выше проблемы.

Профиль stable. Файл sudo nano /etc/NetworkManager/conf.d/00-macrandomize-stable.conf:

[device]
wifi.scan-rand-mac-address=yes

[connection]
wifi.cloned-mac-address=stable
ethernet.cloned-mac-address=stable
connection.stable-id=${CONNECTION}/${BOOT}

Профиль random. Файл sudo nano /etc/NetworkManager/conf.d/00-macrandomize-random.conf

[device]
wifi.scan-rand-mac-address=yes

[connection]
wifi.cloned-mac-address=random
ethernet.cloned-mac-address=random

sudo systemctl restart NetworkManager

Для отключения рандомизации и возвращения настроек по умолчанию достаточно просто удалить созданный файл и перезапустить Network Manager.

##### Отключение  автоматических запросов к домену fedoraproject.org с целью проверки соединения

sudo dnf remove NetworkManager-config-connectivity-fedora

sudo systemctl restart NetworkManager.service

##### Настройка CloudFlare DNS over TLS

Дать systemd-resolved приявлегия основного резолвера

Важно: Начиная с Fedora 33, systemd-resolved уже используется в качестве основного системного DNS-резолвера.

Удалим существующую символическую ссылку, указывающую на Network Manager:

sudo rm -f /etc/resolv.conf

Установим systemd-resolved основным резолвером:

sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

Изменения вступят в силу немедленно.

Откроем файл конфигурации /etc/systemd/resolved.conf:

sudoedit /etc/systemd/resolved.conf

Внесём следующие правки:

[Resolve]
DNS=1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001
FallbackDNS=8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
#Domains=
#LLMNR=yes
MulticastDNS=yes
DNSSEC=allow-downgrade
DNSOverTLS=opportunistic
Cache=yes
DNSStubListener=yes
ReadEtcHosts=yes

Здесь используются серверы CloudFlare с поддержкой DNS-over-TLS.

Сохраним изменения в файле и перезапустим systemd-resolved:

sudo systemctl restart systemd-resolved.service

Теперь в информации об используемых DNS должна отображаться информация об использовании этой технологии

##### Настройка оболочки

Расширения Gnome

1) CPU Power Manager
2) Removable Drive Menu
3) GSConnect
4) Dash to Dock
5) Blur my Shell
6) Compiz windows effect
7) Burn My Windows (Эффект ТВ - 400)
8) Compiz alike magic lamp effect
9) Internet Speed Monitor 
10) Caffeine
11) Transparent window moving
12) Resource Monitor

##### Оптимизация Firefox


1) Кеш Firefox в ОЗУ

about:config

browser.cache.disk.enable - "false" 

browser.cache.disk.capacity - 0 

browser.cache.memory.enable - "true" 

browser.cache.memory.capacity - -1

browser.cache.disk_cache_ssl - false

browser.cache.offline.enable - false

2) Ускорение интернета

about:config

network.http.max-persistent-connections-per-server - до 10 (но не больше) 

network.http.max-connections - 900

network.http.http3.enabled - false (ускорение Гугл и ютуб)

browser.sessionstore.interval - частота записи на диск (больше - лучше)

browser.sessionhistory.max_entries - 5 (количество ссылок назад/вперёд)


3) Отключение телеметрии

browser.ping-centre.telemetry - false

security.ssl.errorReporting.automatic - true

toolkit.identity.enabled - false

toolkit.telemetry.archive.enabled - false

toolkit.telemetry.bhrPing.enabled - false

toolkit.telemetry.coverage.opt-out - false

toolkit.telemetry.enabled - false

toolkit.telemetry.firstShutdownPing.enabled - false

toolkit.telemetry.hybridContent.enabled - false

toolkit.telemetry.infoURL - пусто

toolkit.telemetry.newProfilePing.enable - false

toolkit.telemetry.reportingpolicy.firstRun - false

toolkit.telemetry.server - пусто

toolkit.telemetry.shutdownPingSender.enabled - false

toolkit.telemetry.unified - false

toolkit.telemetry.updatePing.enabled - false


4) Отчеты корпорации Mozilla

datareporting.healthreport.uploadEnabled - false

datareporting.policy.dataSubmissionEnabled - false

datareporting.policy.firstRunURL - пусто


5) Настройки плавного скрола

general.smoothScroll.currentVelocityWeighting - 0

general.smoothScroll.durationToIntervalRatio - 1000

general.smoothScroll.lines.durationMaxMS -150

general.smoothScroll.lines.durationMinMS - 0

general.smoothScroll.mouseWheel.durationMaxMS - 150

general.smoothScroll.mouseWheel.durationMinMS - 0

general.smoothScroll.mouseWheel.migrationPercent - 0

general.smoothScroll.msdPhysics.continuousMotionMaxDeltaMS - 250

general.smoothScroll.msdPhysics.enabled - true

general.smoothScroll.msdPhysics.motionBeginSpringConstant - 450

general.smoothScroll.msdPhysics.regularSpringConstant - 450

general.smoothScroll.msdPhysics.slowdownMinDeltaMS - 50

general.smoothScroll.msdPhysics.slowdownSpringConstant - 5000

general.smoothScroll.other - true

general.smoothScroll.other.durationMaxMS - 150

general.smoothScroll.other.durationMinMS - 0

general.smoothScroll.pages.durationMaxMS - 150

general.smoothScroll.pages.durationMinMS - 0

general.smoothScroll.pixels - true

general.smoothScroll.pixels.durationMaxMS -150

general.smoothScroll.pixels.durationMinMS - 0

general.smoothScroll.scrollbars.durationMaxMS - 600

general.smoothScroll.scrollbars.durationMinMS - 0

general.smoothScroll.stopDecelerationWeighting - 0.2

mousewheel.acceleration.factor - 5


6) Ускорение отрисовки

gfx.use_text_smoothing_setting - true

gfx.webrender.enabled - true

gfx.webrender.highlight-painted-layers - false

layers.acceleration.force-enabled - true

nglayout.initialpaint.delay - 0 (рендер без задержки)


7) Настройки приватности

privacy.resistFingerprinting - true

privacy.resistFingerprinting.autoDeclineNoUserInputCanvasPrompts - false

extensions.pocket.enabled - false

media.peerconnection.enabled - false

Просмотреть все:
1) privacy.trackingprotection (включить все)
2) fingerprinting (включить)

8) Расширения

1) No script 
2) Canvas defender 
3) Ublock origin + фильтр IP-логеров
4) Dark reader
5) Clear URLs 
6) Chameleon
7) TWP - translate web page 
8) Firefox multi-account Containers 
9) Decentraleyes
10) Temporary container 
11) Disconnect