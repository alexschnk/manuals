#инструкция #linux #производительность #void
### Подготовка к установке 
```
root
voidlinux
bash
```
### Создание и монтирование разделов

```
lsblk

cfdisk

mkfs.vfat -F32 /dev/sdb1
mkfs.btrfs -f /dev/sdb2

lsblk -f

mount /dev/sdb2 /mnt
btrfs su cr /mnt/@
btrfs su cr /mnt/@home
btrfs su list /mnt

umount -R /mnt

mount -o rw,lazytime,compress=zstd,max_inline=256,space_cache=v2,ssd,commit=300,subvol=@ /dev/sdb2 /mnt

df -h

mkdir /mnt/home

ls /mnt

mount -o rw,lazytime,compress=zstd,max_inline=256,space_cache=v2,ssd,commit=300,subvol=@home /dev/sdb2 /mnt/home


mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/sdb1 /mnt/boot/efi

lsblk
```
### Обновление XBPS
```
xbps-install -Su xbps
```
### Подключение репозитория
```
REPO=https://repo-default.voidlinux.org/current
```
### Выбор архитектуры 
```
ARCH=x86_64
```
### Копирование ключей XBPS
```
mkdir -p /mnt/var/db/xbps/keys
cp /var/db/xbps/keys/* /mnt/var/db/xbps/keys/
```
### Установка базовой системы 
```
XBPS_ARCH=$ARCH xbps-install -S -r /mnt -R "$REPO" base-system base-devel linux nano bash-completion 
```
### CHROOT
```
xchroot /mnt /bin/bash
```
### Настройка временной зоны
```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```
### Настройка клавиатуры
```
nano /etc/rc.conf
KEYMAP="ru"
FONT=cyr-sun16
```
### Имя хоста
```
nano /etc/hostmane
voidlinux
```
### Настройка локалей
```
nano /etc/default/libc-locales
en_US.UTF-8 UTF-8
ru_RU.UTF-8 UTF-8
xbps-reconfigure -f glibc-locales

nano /etc/locale.conf
LANG=ru_RU.UTF-8
LC_COLLATE=C
```
### Генерация FSTAB 
```
cp /proc/mounts /etc/fstab
nano /etc/fstab
```
### Задание пароля ROOT
```
passwd
```
### Создание пользователя
```
useradd -m -U -G wheel,audio,video,plugdev -s /bin/bash void
passed void
```
### Добавление пользователя в группу WHEEL
```
visudo
%wheel ALL=(All:ALL) ALL
void ALL=(ALL:ALL) ALL
```
### Установка загрузка 
```
xbps-install grub-x86_64-efi efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```
### Реконфигурирование системы
```
xbps-reconfigure -fa
```
### Перезагрузка 
```
exit

umount -R /mnt
```