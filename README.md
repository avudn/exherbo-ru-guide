# Гайд на установку Exherbo GNU/Linux (2026)

# 1.0 Почему Exherbo GNU/Linux?
Exherbo Linux, это дистрибутив на базе Gentoo GNU/Linux. Но базирован он скорее не на коде - а на идеях. В Exherbo, ты компилируешь все с исходного кода, и имеешь возможность использовать use-флаги. <br>

Данный дистрибутив рассчитан на использование опытными пользователями, которые уже имеют довольно хорошее понимание о Linux, и так же готовы принять участие в его разработке. Но установить его, и использовать может каждый! В этом гайде будет расписан каждый шаг, и объяснения действий для успешной установки Exherbo GNU/Linux. <br>

В этом гайде я покажу установку glibc systemd Exherbo GNU/Linux

# 1.1 Что такое USE-флаги?
USE-флаги, это флаги которые ты выставляешь пакетам с целью кастомизации. <br>
Самый популярный пример дистрибутива с USE-флагами, это Gentoo GNU/Linux. Главная особенность Gentoo - это компиляция с исходников, а так же use-флаги. <br>

Пример использования:
Я хочу установить GRUB, с поддержкой UEFI! <br>

В /etc/paludis/options.conf: <br>
```sys-boot/grub efi``` <br>

Добавив флаг "efi", вы дали знать пакетнику что вы хотите скомпилировать grub с UEFI-поддержкой. Это лишь базовый пример, на самом деле применений у use-флагов очень много. Можно минимизировать пакеты, вырезая из них не нужное вам, и так далее. Коротко: use-флаги позволяют вам кастомизировать пакеты под ваш вкус и цвет.

# 1.2 Подготовка к установке.
У Exherbo GNU/Linux отсутствует Live-ISO. В моем случае, я использую CachyOS Live-ISO для установки. Но вы можете использовать любой, который имеет нужные инструменты. Используйте dd в Linux чтобы перезаписать данные на флешке с нужным Live-ISO или Rufus если вы на Windows. <br>

# 1.3 Разделы
Мы создадим 2 раздела, для простой установки. По желанию можно использовать и home раздел, про это написано в https://exherbo.org/docs/install-guide.html.

Для этого, мы используем команду:
``` cfdisk /dev/sda ``` <br>

uВаши разделы должны выглядеть примерно так: <br>
<img src="./examples/1.png" height=420> <br>
В моем случае, /dev/nvme0n1p1 - это будет ESP, а root будет /dev/nvme0n1p2. <br>

Теперь, мы отформатируем разделы. <br>

## UEFI:
```mkfs.vfat -F32 /dev/sda1``` <br>
```mkfs.ext4 /dev/sda2``` <br>

## Legacy
```mkfs.ext2 /dev/sda1``` <br>
```mkfs.ext4 /dev/sda2``` <br>

# 1.4 Установка базы
Мы должны создать директорию для монтирования, и вмонтировать наш root-раздел. Это всё делается за одну линию: <br>
```mkdir /mnt/exherbo && mount /dev/sda2 /mnt/exherbo && cd /mnt/exherbo``` <br>

После этого, мы скачиваем stage-файл Exherbo Linux. Стейдж файл содержит в себе базу системы. <br>
```curl -O https://stages.exherbo.org/x86_64-pc-linux-gnu/exherbo-x86_64-pc-linux-gnu-gcc-current.tar.xz``` <br>

После того как вы скачали этот файл, можно прописать ```tar xJpf exherbo*xz```, чтобы распаковать все. 

# 1.5 fstab

Чтобы наша система запустилась, нам нужно создать файл `fstab`. В нём расписано, что и как нужно монтировать при запуске системы.

```bash
vim /etc/fstab
```

Добавьте:

```fstab
# <fs>       <mountpoint>    <type>    <opts>      <dump/pass>
/dev/sda2    /               ext4      defaults    0 1
```

Если вы используете **UEFI booting**:

```bash
echo "/dev/sda1    /boot/efi    vfat    defaults    0 0" >> /mnt/exherbo/etc/fstab
```

Если используется **Legacy/BIOS**:

```bash
echo "/dev/sda1    /boot/efi       ext2    defaults    0 0" >> /mnt/exherbo/etc/fstab
```

# 1.6 Chroot, установка ядра.
Монтируем по очереди: <br>

```bash
mount -o rbind /dev /mnt/exherbo/dev/
``` <br>

```bash
mount -o rbind /sys /mnt/exherbo/sys/
mount -t proc none /mnt/exherbo/proc/
mkdir -p /mnt/exherbo/boot/efi 
mount /dev/sda1 /mnt/exherbo/boot/efi
``` <br>



