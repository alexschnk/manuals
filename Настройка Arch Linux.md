#технологии #информация #инструкция #arch #linux 
## Обязательные настройки
---
##### Редактрование /etc/fstab

- Ускорение SSD

```
rw,lazytime,ssd,space_cache=v2, max_inline=256,commit=600,subvol
```
- Кеш програм в ОЗУ

```
tmpfs    /путь    tmpfs    defaults,noatime,mode=1777    0 0

/tmp                                                           

/var/tmp

/var/log

/home/arch (имя пользователя)/.cache                       

/home/arch (имя пользователя)/Загрузки                     

/var/cache/pacman/pkg                   

/home/arch (имя пользователя)/.var/app/org.kde.kdenlive/cache           
                                     
/home/arch (имя пользователя)/.var/app/org.telegram.desktop/data/TelegramDesktop/tdata/user_data        
        
/home/arch (имя пользователя)/.var/app/org.telegram.desktop/data/TelegramDesktop/tdata/temp_data

/home/arch (имя пользователя)/.var/app/org.mozilla.firefox/cache
```
##### Ускорение загрузки пакетов и настройка Pacman


- Ускорение загрузки пакетов Arch linux
```
sudo pacman -S reflector

sudo nano /etc/xdg/reflector/reflector.conf

-- sort rate

sudo systemctl start reflector
```
- Настройка Pacman
```
sudo nano /etc/pacman.conf
ILoveCandy
```
##### Установка пакетов под версию процессора (производительность Gentoo без компиляции)


Проверка поколения процессора 

```
/lib/ld-linux-x86-64.so.2 --help | grep -B 3 -E "x86-64-v3"
```
Установка ключей и зеркал
```
yay - S alhp-keyring alhp-mirrorlist
```
Подключение репозиториев
```
sudo nano /etc/pacman.conf

[core-x86-64-v3]
Include = /etc/pacman.d/alhp-mirrorlist

[extra-x86-64-v3]
Include = /etc/pacman.d/alhp-mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

sudo pacman -Syyuu

reboot
```
##### Драйвера

- Ядро и драйвера CPU
```
sudo pacman -S base base-devel linux-zen linux-zen-headers linux-firmware dosfstools btrfs-progs intel-ucode amd-ucode iucode-tool archlinux-keyring bluez bluez-utils nano vim dhcpcd dhclient networkmanager
```
- Графические драйвера AMD
```
sudo pacman -S xf86-video-ati xf86-video-amdgpu mesa mesa-demos lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader amdvlk lib32-amdvlk network-manager-applet
```
- Графические драйвера Intel
```
sudo pacman -S mesa mesa-demos xf86-video-intel lib32-mesa vulkan-intel lib32-vulkan-intel vulkan-icd-loader lib32-vulkan-icd-loader network-manager-applet libva-intel-driver lib32-libva-intel-driver
```
##### Параметры ядра
```
sudo nano /etc/default/grub

GRUB_DEFAULT=0
GRUB_TIMEOUT=3
GRUB_SAVEDEFAULT=false
GRUB_TIMEOUT_STYLE=false
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 elevator=noop ipv6.disable=1 raid=noautodetect rootfstype=btrfs  page_alloc.shuffle=1 split_lock_detect=off nowatchdog mitigations=off"
GRUB_CMDLINE_LINUX="rootfstype=btrfs"
```
Конфигурирование 
```
sudo grub-mkconfig -o /boot/grub/grub.cfg

reboot
```
>**pti=on**: Эта опция включает изоляцию таблицы страниц ядра KPT

>**module.sig_enforce=1**: это позволяет загружать только модули ядра, подписанные  
действительным ключом, что повышает безопасность, затрудняя загрузку вредоносных модулей ядра

>**ipv6.disable=1**: опция полностью отключает IPV6 в системе, поскольку данный протокол сильно  
нарушает нашу безопасность

>**elevator=noop**: планировщик SSD

>**raid=noautodetect**: отключает проверку на RAID во время загрузки. Если вы его используете - НЕ прописывайте данный параметр.

>**rootfstype=btrfs**: здесь указываем название файловой системы в которой у вас отформатирован корень.

>**page_alloc.shuffle=1**: этот параметр рандомизирует свободные списки распределителя страниц. Улучшает производительность при работе с ОЗУ с очень быстрыми накопителями (NVMe, Optane). Подробнее тут.

>**split_lock_detect=off**: отключаем раздельные блокировки шины памяти. Одна инструкция с раздельной блокировкой может занимать шину памяти в течение примерно 1 000 тактов, что может приводить к кратковременным зависаниям системы.

>**pci=pcie_bus_perf**: увеличивает значение Max Payload Size (MPS) для родительской шины PCI Express. Это даёт лучшую пропускную способность, т. к. некоторые устройства могут использовать значение MPS/MRRS выше родительской шины. Больше подробностей здесь (англ.):

>**nowatchdog**: отключает сторожевые таймеры. Позволяет избавиться от заиканий в онлайн играх.

>**mitigations=off**: непосредственно отключает все заплатки безопасности ядра (включая Spectre и Meltdown)

>**threadirqs:** задействует многопоточную обработку IRQ прерываний

>**intel_idle.max_cstate=1**: только для процессоров Intel. Отключает энергосберегательные функции процессора, ограничивая его спящие состояния, не позволяя ему переходить в состояние глубокого сна. Увеличивает может значительно увеличить энергопотребление на ноутбуках. Помогает исправлять некоторые странные зависания и ошибки на многих системах

##### Настройка modprobe
```
lsmod | grep iwlwifi
```
Если в окно терминала будет выведена строка «iwlwifi», вы можете переходить к следующему шагу.
```
sudo nano /etc/modprobe.d/iwlwifi11n.conf
options iwlwifi 11n_disable=8
```
Если проблемы с сетью
```
sudo rm -v /etc/modprobe.d/iwlwifi11n.conf
```
##### Настройка sysctl
```
sudo modprobe tcp_bbr

sudo nano /etc/modules-load.d/tcp-bbr.conf
tcp_bbr

sudo nano /etc/sysctl.d/99-sysctl.conf
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_congestion_control=bbr
net.core.default_qdisc=cake
vm.vfs_cache_pressure=30
```
##### Установка служб оптимизации
```
yay -S ananicy-cpp
sudo systemctl enable --now ananicy-cpp

sudo pacman -S dbus-broker
sudo systemctl disable dbus
sudo systemctl enable --now dbus-broker 
```
##### Настройка переменных окружения

```
sudo nano /etc/environment
```
**Включение OpenGL**
```
MESA_GL_VERSION_OVERRIDE=4.5
MESA_GLSL_VERSION_OVERRIDE=450
__GL_THREADED_OPTIMIZATIONS=1
```
>**__GL_THREADED_OPTIMIZATIONS=1** (По умолчанию выключено) — Активируем многопоточную обработку OpenGL. Используете выборочно для нативных игр/приложений, ибо иногда может наоборот вызывать регрессию производительности. Некоторые игры и вовсе могут не запускаться с данной переменной (К примеру, некоторые нативно-запускаемые части Metro)

>**__GL_MaxFramesAllowed=3 **(По умолчанию — 2) — Задает тип буферизации кадров драйвером. Можете указать значение “3” (Тройная буферизация) для большего количества FPS и улучшения производительности в приложениях/играх с VSync. Мы рекомендуем задавать вовсе “1” (т.е. не использовать буферизацию, подавать кадры так как они есть). Это может заметно уменьшить значение FPS в играх, но взамен вы получите лучшие задержки отрисовки и реальный физический отклик, т.к. кадр будет отображаться вам сразу на экран без лишних этапов его обработки

>**__GL_YIELD=»NOTHING»** (По умолчанию без значения) — Довольно специфичный параметр, “USLEEP” — снижает нагрузку на CPU и некоторым образом помогает в борьбе с тирингом, а “NOTHING” дает больше FPS при этом увеличивая нагрузку на процессор

>COGL_ATLAS_DEFAULT_BLIT_MODE=framebuffer 

>**export vblank_mode=0** - отключает вертикальную синхронизацию по всей системе и для любого приложения , эта опция иногда выручает при проблемах с записью экрана и повышает фпс и отзывчивость всей системы , использовать вместе с параметром сжатия буфера кадров можно и ничего не сломается , но эффект лучше не будет , каждая опция выполняет свою конкретную работу и будут мешать друг другу , отключение вертикальной синхронизации не будет нормально работать , поэтому рекомендую использовать что то одно

>**#export i915_enable_rc6=7**- отвечает за алгоритм сжатия буфера кадров что увеличивает немного отзывчивость системы и FPS , однако не всегда работает корректно и может приводить к к тому что оболочка запускается в черный экран , кроме того влияет на запись экрана которая может быть некорректно записана поэтому в зависимости от того как ведет себя система можно включить данную опцию убрав решетку или отключить , вернув решетку на место

**Параметры связаны с включением стандартных опций драйвера i915 который работает для видеокарт intel по умолчанию**

>export i915

>export i915_enable_fbc=1

>export lvds_downclock=1

##### Отключение дампов ядра
```
sudo nano /etc/systemd/coredump.conf
```
в разделе [Coredump]
```
Storage = none

sudo systemctl daemon-reload

sudo nano /etc/security/limits.conf

* hard core 0

sudo nano /etc/sysctl.d/suid_dumpable.conf
fs.suid_dumpable=0
```
##### Повышение лимитов
```
sudo nano /etc/systemd/system.conf
sudo nano /etc/systemd/user.conf

DefaultLimitNOFILE=2000000

sudo systemctl daemon-reload

sudo nano /etc/security/limits.conf 
username hard nofile 2000000
```
##### Добавление важных модулей в образы initramfs
```
sudo nano /etc/mkinitcpio.conf

MODULES=(i915 crc32c-intel libcrc32c zlib_deflate btrfs) 

HOOKS=(base udev autodetect modconf kms keyboard keymap block filesystems fsck)

sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

##### Перенос кэша компиляции (yay) в tmpfs
```
sudo nano /etc/makepkg.conf 
BUILDDIR=/tmp/makepkg
```
##### Установка пакетов (pacman, yay, flatpak)
```
sudo pacman -S flatpak git cpupower p7zip f2fs-tools ntfs-3g kdiskmark iotop htop archiso zsh zsh-completions lrzip unrar unzip unace squashfs-tools spectacle android-tools gocryptfs unrar

yay -S timeshift sirikali cpupower-gui zulucrypt rpm dpkg

flatpak install flathub org.mozilla.firefox org.telegram.desktop io.github.mimbrero.WhatsAppDesktop org.kde.ark org.keepassxc.KeePassXC md.obsidian.Obsidian org.libreoffice.LibreOffice io.gitlab.librewolf-community
org.qbittorrent.qBittorrent com.github.tchx84.Flatseal
```
##### Увеличение частот CPU за счет включения  более производительного профиля процессора 
```
sudo cpupower frequency-set -g performance
 
sudo nano /etc/systemd/system/cpupower.service

[Unit]
Description=Set CPU governor to performance

[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower -c all frequency-set -g performance


[Install]
WantedBy=multi-user.target

sudo systemctl enable cpupower.service

yay -S cpupower-gui

sudo systemctl disable cpupower-gui
```
##### Оптимизация Firefox

- Кеш Firefox в ОЗУ
```
about:config

browser.cache.disk.enable - "false" 

browser.cache.disk.capacity - 0 

browser.cache.memory.enable - "true" 

browser.cache.memory.capacity - -1

browser.cache.disk_cache_ssl - false

browser.cache.offline.enable - false
```
- Ускорение интернета
```
about:config

network.http.max-persistent-connections-per-server - до 10 (но не больше) 

network.http.max-connections - 900

network.http.http3.enabled - false (ускорение Гугл и ютуб)

browser.sessionstore.interval - частота записи на диск (больше - лучше)

browser.sessionhistory.max_entries - 5 (количество ссылок назад/вперёд)
```
- Отключение телеметрии
```
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
```
- Отчеты корпорации Mozilla
```
datareporting.healthreport.uploadEnabled - false

datareporting.policy.dataSubmissionEnabled - false

datareporting.policy.firstRunURL - пусто
```
- Настройки плавного скрола
```
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
```
- Ускорение отрисовки
```
gfx.use_text_smoothing_setting - true

gfx.webrender.enabled - true

gfx.webrender.highlight-painted-layers - false

layers.acceleration.force-enabled - true

nglayout.initialpaint.delay - 0 (рендер без задержки)
```
- Настройки приватности
```
privacy.resistFingerprinting - true

privacy.resistFingerprinting.autoDeclineNoUserInputCanvasPrompts - false

extensions.pocket.enabled - false

media.peerconnection.enabled - false

Просмотреть все:
1) privacy.trackingprotection (включить все)
2) fingerprinting (включить)
```
- Расширения
```
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
```
- Профиль в ОЗУ

а) Установка службы
```
sudo pacman -S profile-sync-daemon
```
б) Генерация настроек
```
psd
```
в) Редактирование настроек

```sudo nano /home/facade/.config/psd/psd.conf
BROWSERS=(firefox)
```
г) После этого нужно закрыть файрфокс и запустить psd командой
```
systemctl --user start psd.service
```
д) После  посмотреть что с сервисом
```
systemctl --user status psd.service
psd preview
```
## Факультативные настройки
---
##### Установка QEMU / KVM

```sudo pacman -S qemu virt-manager libvirt virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat nftables libguestfs dmidecode

sudo pacman -S ebtables

sudo systemctl start libvirtd

sudo systemctl enable libvirtd

sudo systemctl status libvirtd

sudo nano /etc/libvirt/libvirtd.conf
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"

sudo usermod -a -G libvirt имя_пользователя

sudo nano /etc/group
libvirt:x:963:имя_пользователя

sudo nano /etc/libvirt/qemu.conf
user = "root" -> user = "имя_пользователя"
group = "root" -> group = "libvirt"

sudo systemctl restart libvirtd

sudo systemctl status libvirtd
```
##### Включение Iommu в Bios

##### Добавление параметров безопасности ядра 
```
sudo nano /etc/sysctl.d/
```
>**kptr_restrict.conf 
kernel.kptr_restrict=2**
Предотвращает утечку указателей ядра через (proc,  kallsyms или dmesg), тем самым защищает от эксплойтов ядра  
  
>**dmesg_restrict.conf 
kernel.dmesg_restrict=1**
Блокирует всем пользователям кроме root доступ к журналам ядра 
  
>**harden_bpf.conf 
kernel.unprivileged_bpf_disabled=1
net.core.bpf_jit_harden=2**
Разрешает доступ к динамическому JIT компилятору только учётной записи с привилегиями root 
  
>**ptrace_scope.conf  
kernel.yama.ptrace_scope=2**
Позволяет использовать ptrace  только для процессов CAP_SYS_PTRACE 
Ptrace - это системный вызов,  который позволяет программе  изменять и проверять запущенный процесс, что даёт возможность  злоумышленникам легко  скомпрометировать другие  запущенные программы
 
>**kexec.conf
kernel.kexec_load_disabled=1**
Отключает kexec, который злоумышленник может использовать для подмены работающего ядра
 
>**mmap_aslr.conf 
vm.mmap_rnd_bits=32 
vm.mmap_rnd_compat_bits=16**
Устанавливает максимальное  
значение для параметров ASLR 
  
>**sysrq.conf 
kernel.sysrq=0**
Отключает SysRq, тем самым  
запрещая непривилегированным 
пользователям доступ к отладке 

>**unprivileged_users_clone.conf kernel.unprivileged_userns_clone=0**
Отключает пространства имён непривилегированных  пользователей, тем самым  существенно снижает поверхность атак для повышения прав в системе

>**tcp_sack.conf 
net.ipv4.tcp_sack=0**
Отключает TCP SACK

##### Отключение небезопасных модулей ядра

```sudo nano /etc/modprobe.d/blacklist-dma.conf 
install firewire-core /bin/true 
install thunderbolt /bin/true
```
Данная команда создаст файл по пути /modprobe.d/ в который мы внесём название модулей, которые не будут загружаться при старте ядра. В данный файл нужно добавить две строки после чего сохранить и выйти из nano. Не забудем перезагрузить компьютер
 
```
modprobe --showconfig | grep "^install" | grep "/bin" 
```
после перезагрузки мы выполняем  
команду, которая отобразит список заблокированных модулей. Как видите модули, которые делают нас уязвимыми перед DMA атакой теперь заблокированы.
 
##### Отключение невостребованных файловых систем и сетевых протоколов 
``` 
sudo nano /etc/modprobe.d/uncommon-filesystems.conf
install cramfs /bin/true  
install freevxfs /bin/true  
install jffs2 /bin/true  
install hfs /bin/true  
install hfsplus /bin/true  
install squashfs /bin/true  
install udf /bin/true

sudo nano /etc/modprobe.d/uncommon-network-protocols.conf  
install dccp /bin/true  
install sctp /bin/true  
install rds /bin/true  
install tipc /bin/true  
install n-hdlc /bin/true  
install ax25 /bin/true  
install netrom /bin/true  
install x25 /bin/true  
install rose /bin/true  
install decnet /bin/true  
install econet /bin/true  
install af_802154 /bin/true  
install ipx /bin/true  
install appletalk /bin/true  
install psnap /bin/true  
install p8023 /bin/true  
install llc /bin/true  
install p8022 /bin/true
```
##### Подмена Mac-адреса

- Вариант 1```
```
sudo pacman -S net-tools

sudo ifconfig
```
Обратите внимание на строку ether - это и есть мак-адрес сетевого  
устройства
```
sudo nano /etc/NetworkManager/conf.d/00-macrandomize.conf
[device] 
wifi.scan-rand-mac-address=yes 
[connection] 
wifi.cloned-mac-address=random 
ethernet.cloned-mac-address=random 
connection.stable-id=${CONNECTION}/${BOOT}
 ```
>**wifi.scan-rand-mac-address=yes** - генерировать случайные MAC-адреса во время сканирования wifi
 
>**wifi.cloned-mac-address=random** - генерировать случайный MAC-адрес wifi при каждом новом подключении
 
>**ethernet.cloned-mac-address=random** - генерировать случайный MAC-адрес сетевой карты ethernet при каждом подключении

```
sudo systemctl restart NetworkManager
```
- Вариант 2
Параметры системы > Категория Сеть и связь > Соединения. В списке соединений wifi  
найти требуемое подключение и во вкладке WI-Fi найти строку клонированный MAC-адрес.  
Далее просто вводим MAC-адрес, который будет отличаться на один символ от легитимного.  
Таким образом вы сможете подключаться и работать на роутере, оставаться незамеченным и при этом не мешать людям пользоваться домашним интернетом

##### Подмена хоста

```
git clone github.com/red0xff/hostnamespoof.git
```
Теперь открываем папку hostnamespoof в пакетном менеджере, открываем вложенную папку hostname_generators > docker_container_names и переименовываем docker_container_names.sh в hostnamegen приставку .sh при этом мы убираем!!!

Создание рабочего каталога для скрипта
```
sudo mkdir /usr/share/hostnamespoof
```
Копируем словарь hostnamegen, который мы переименовали на прошлом шаге в рабочий каталог скрипт
```
sudo cp hostname_generators/docker_container_names
/hostnamegen /usr/share/hostnamespoof/
```
Копируем сам скрипт в рабочий каталог
```
sudo cp spoof_hostname.sh /usr/share/hostnamespoof/
```
Проверяем действительно ли скопировались два файла
```
ls /usr/share/hostnamespoof/
```
Даём право словарю hostnamegen на исполнение
```
sudo chmod +x /usr/share/hostnamespoof/hostnamegen
```
Копируем службу автозапуска 
hostnamespoof в рабочий каталог systemd
```
sudo cp hostnamespoof.service /etc/systemd/system
```
Проверяем что копирование произошло успешно
```
 ls /etc/systemd/system
```
Входим в рабочий каталог hostnamespoof
```
cd /usr/share/hostnamespoof/
```
Выполняем данную команду, чтобы узнать текущее имя компьютера в сети, как видим в моём случае это user
```
hostname
```
Запускаем скрипт подмены имени хоста и проверяем его на предмет ошибок. В нашем случае ошибок не возникло, проверяем следом ещё раз имя хоста и как видим оно теперь objective-gagarin. Значит скрипт полностью работает
```
sudo ./spoof_hostname.sh
```
В автозагрузку
```
sudo systemctl enable hostnamespoof.service
```
##### Фаервол
```
sudo pacman -S opensnitch
sudo systemctl start opensnitchd
sudo systemctl enable opensnitchd
```
##### Отключение Baloo в Plasma

Baloo - это файловый индекстор в Plasma, аналог Tracker в GNOME, который однако ОЧЕНЬ прожорливый, и ест довольно много ресурсов процессора и памяти, вдобавок фоном нагружая ваш диск, в отличии от того же Tracker 3.
```
systemctl --user mask kde-baloo.service           
systemctl --user mask plasma-baloorunner.service
```
##### Отключение отладочной информации в KDE5
```
kdebugdialog5
```
##### Отключение проверки ключей в /etc/pacman.conf
```
sudo nano /etc/pacman.conf
раздел [options]
SigLevel = Never
```
##### Установка Zen- Hardened- ядра
```
sudo pacman -S linux-zen linux-zen-headers
sudo mkinitcpio -p linux-zen
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot

sudo pacman -S linux-hardened linux-hardened-headers

sudo mkinitcpio -p linux-hardened
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot
```
##### Настройка ZRAM 
```
$ su

$ nano /etc/modules-load.d/zram.conf

zram

$ nano /etc/modprobe.d/zram.conf

options zram num_devices=8


$ nano /etc/udev/rules.d/99-zram.rules

KERNEL=="zram0", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram0", TAG+="systemd"
KERNEL=="zram1", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram1", TAG+="systemd"
KERNEL=="zram2", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram2", TAG+="systemd"
KERNEL=="zram3", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram3", TAG+="systemd"
KERNEL=="zram4", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram4", TAG+="systemd"
KERNEL=="zram5", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram5", TAG+="systemd"
KERNEL=="zram6", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram6", TAG+="systemd"
KERNEL=="zram7", ATTR{disksize}="512Mb" RUN="/usr/bin/mkswap /dev/zram7", TAG+="systemd"

$ nano /etc/fstab

/dev/zram0 none swap defaults 0 0
/dev/zram1 none swap defaults 0 0
/dev/zram2 none swap defaults 0 0
/dev/zram3 none swap defaults 0 0
/dev/zram4 none swap defaults 0 0
/dev/zram5 none swap defaults 0 0
/dev/zram6 none swap defaults 0 0
/dev/zram7 none swap defaults 0 0

$ echo "vm.swappiness=40" > /etc/sysctl.d/swap.conf

reboot
```
##### Расширения Gnome

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
##### Редактирование файла hosts
##### Включить bluetooth
```
sudo pacman -S bluez bluez-utils pulseaudio-bluetooth blueman

sudo systemctl start bluetooth  

sudo systemctl enable bluetooth
```