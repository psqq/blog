Информация о посте:
* Id поста: `432`.
* Чтобы обсудить пост [создайте issue](https://github.com/psqq/blog/issues) с заголовком, который должен начинаться с id этого поста: `432: ...`.

# My experience using Arch Linux

<!-- begin post preview -->

В этом посте я делюсь опытом использования `Arch Linux`. Пишу о том как я устанавливаю и настраиваю `Arch Linux`, а также даю решения для некоторых проблем, с которыми я столкнулся в ходе использования этой ОС. Пишу только основное, чтобы не растягивать пост, но даю ссылки, по которым можно получить детальную информацию.

<!-- end post preview -->

Инструкции могут устареть поэтому пользуйтесь с осторожностью.

# Особенности

Из особенностей `Arch Linux` я бы выделил следующие:

* `Arch Linux` придерживается модели [роллинг-релизов](https://ru.wikipedia.org/wiki/Rolling_release).
  * Устанавливаете один раз, а дальше просто обновляете установленные пакеты в операционной системе.
* Оптимизирован под архитектуру `x86-64`.
* Пакетный менеджер `pacman`, который понравился мне своей быстротой по сравнению с `apt` из `Ubuntu`.
* Есть огромный пользовательский репозиторий пакетов [AUR](https://aur.archlinux.org/).

Об особенностях можете также почитать краткий обзор с вики: [https://wiki.archlinux.org/title/Arch_Linux](https://wiki.archlinux.org/title/Arch_Linux).

# Установка

Гайд по установке с вики: [https://wiki.archlinux.org/title/Installation_guide](https://wiki.archlinux.org/title/Installation_guide).

Опишу, просто в качестве примера, как обычно я устанавливаю `Arch Linux`.

Для начала [скачиваем](https://archlinux.org/download/) и монтируем `ISO` образ на флешку. Например, с помощью [Rufus](https://github.com/pbatard/rufus) (только для Windows) или с помощью [UNetbootin](https://unetbootin.github.io/) (для всех ос).

*Советую всегда иметь флешку с `Arch Linux` (или возможность быстро такую флешку сделать), чтобы при возникновении проблем их можно было решить с её помощью через `arch-chroot`.*

Далее загружаемся с флешки и выбираем пункт меню для установки (должно быть что-то вроде `Arch Linux install medium`).

В установочном образе есть `tmux`.

Сначала нужно убедится, что интернет работает:

```bash
ping archlinux.org
```

Проводной интернет должен подключится сам, `Wi-Fi` нужно [настроить](https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet), например, с помощью [iwctl](https://wiki.archlinux.org/title/Iwd#iwctl).

Далее идут команды для разметки диска. Вам нужно их скорректировать под ваши диск и ваши нужды. Размечаем диск, например, так для `GPT` (взято из [манула nixos](https://nixos.org/nixos/manual/index.html#sec-installation-partitioning-UEFI)):

*Само собой, пути до дисков нужно указать ваши и выполнять только те команды для разметки диска, которые нужны вам.*

```bash
parted /dev/sda -- mklabel gpt
parted /dev/sda -- mkpart ESP fat32 1MiB 512MiB
parted /dev/sda -- set 3 boot on
parted /dev/sda -- mkpart primary linux-swap 512MiB 2GiB
parted /dev/sda -- mkpart primary 2GiB 100%
```

Форматируем разделы:

```bash
mkswap -L swap /dev/sda1
mkswap /dev/sda1
swapon /dev/sda1
mkfs.ext4 -L arch /dev/sda2
mkfs.fat -F 32 -n boot /dev/sda1
```

Перед установкой следует выбрать список серверов-зеркал и прописать их в файл `/etc/pacman.d/mirrorlist`. В `Приложении ru mirrorlist` есть пример списка `ru` серверов-зеркал. Подробности в [гайде по установке](https://wiki.archlinux.org/title/Installation_guide#Installation).

Далее идет собственно установка `Arch Linux` в смонтированный раздел диска. `base linux linux-firmware` - это необходимый минимум пакетов для начальной установки. Я обычно устанавливаю также следующие пакеты:

```bash
mount /dev/sdX1 /mnt
pacstrap -K /mnt base linux linux-firmware vim networkmanager vi sudo zsh grub efibootmgr
```

Дополнительные пакеты:

* `vim` - редактор
* `networkmanager` - программа для управления сетевыми соединениями
* `sudo` - для выполнения команд как рут пользователь из под обычного пользователя
* `vi` - для `visudo`
* `zsh` - использую из-за `Oh My Zsh`
* `grub` - загрузчик ОС
* `efibootmgr` - потребуется для установки загрузчика, если у вас `EFI`

Далее монтируем `efi` раздел и создаем `/etc/fstab` файл:

```bash
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
genfstab -U /mnt >> /mnt/etc/fstab
```

Заходим в установленную ос:

```bash
arch-chroot /mnt
```

С помощью `arch-chroot` можно открыть `shell` для ос даже когда она не загружается обычным способом. Именно поэтому полезно иметь флешку с `Arch Linux`.

Теперь мы находимся под `root` в установленной ОС. Делаем завершающие настройки и перезагружаемся.

Настраиваем часовой пояс и время:

```bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```

Настраиваем локали:

```bash
vim /etc/locale.gen
# Раскомментировать нужное, например:
# en_US.UTF-8 UTF-8
# ru_RU.UTF-8 UTF-8
locale-gen
vim /etc/locale.conf
# Выбрать нужную локаль по умолчанию, например:
# LANG=en_US.UTF-8
```

Устанавливаем [обновления стабильности и безопасности для микрокода процессора](https://wiki.archlinux.org/title/Microcode):

```bash
pacman -S amd-ucode # для AMD
pacman -S intel-ucode # для Intel
```

Включаем сервис `NetworkManager`, если вы его устанавливали. Вместе с пакетом `networkmanager` также идет удобная утилита с консольным `GUI`: `nmtui`. С помощью нее легко настроить `Wi-Fi` и другие сетевые подключения.

```bash
systemctl enable NetworkManager
```

Для определения других ОС загрузчиком `grub` нужно сделать [следующее](https://wiki.archlinux.org/title/GRUB#Detecting_other_operating_systems). Установить паке `os-prober` и в файле `/etc/default/grub` раскомментировать `GRUB_DISABLE_OS_PROBER=false`.

Устанавливаем загрузчик ОС:

```bash
grub-install
grub-mkconfig -o /boot/grub/grub.cfg
```

Не знаю почему, но `os-prober` у меня не находит `Windows` из `Live USB`. Только повторный запуск команд выше после перезагрузки из установленной ОС добавляет `Windows` в список выбора для запуска.

Завершаем установку:

```bash
# Установка пароля для root
passwd

# Добавляем рядового пользователя
useradd -m -G wheel -s /bin/zsh USER_NAME
# Раскомментировать строку `%wheel ALL=(ALL:ALL) ALL`
visudo

# Устанавливаем пароль для пользователя
passwd USER_NAME

# Выходим из arch-chroot, размонтируем рекурсивно /mnt и перезагружаем компьютер
exit
umount -R /mnt
reboot
```

После перезагрузки ОС должна работать и можно продолжить её настройку по вашему вкусу. Ниже в разделе `Настройка ОС` есть некоторые идеи. Подразделы устроены так, что по ним можно идти последовательно.

# Обновление

Вообще обновление системы делается довольно просто командой: `sudo pacman -Syu`, но есть нюансы.

Во-первых, нужно читать новости с официального сайта, они прямо на [главной странице](https://archlinux.org/). Из новостей можно узнать важную информацию или рекомендации для выбора пакетов во время обновления.

Во-вторых, бывают ситуация когда после обновления ОС не запускается `:)`. У меня такое возникало 2 раза за несколько лет, но когда к такому не готов можно потратить много времени на поиск и решение проблемы. Если ОС перестала запускаться именно после обновления, то это должно решиться просто откатом назад. Не знаю есть ли прямо команда "откатись на состояние до обновления", но есть возможность откатиться на определенную дату. Делается это следующим образом (подробности на [хабре](https://habr.com/ru/articles/344000/)). Загружаемся с флешки с `Arch Linux`. Монтируем раздел диска в `/mnt` и делаем `arch-chroot /mnt`. Редактируем файл `/etc/pacman.d/mirrorlist` так, чтобы в нем была только одна строка вида `Server=https://archive.archlinux.org/repos/2024/05/10/$repo/os/$arch`. Нужно указать только нужную дату вместо `2024/05/10`. Пример этого файла в `Приложении mirrorlist`.

Я на данный момент обновляю систему только на определенную дату, чтобы можно было легко переключится на дату когда ОС точно работала.

Один раз у меня перестал запускаться `Arch Linux` из-за того, что я отключил компьютер во время обновления. В таком случае может помочь переустановка всех пакетов через `arch-chroot` ([https://wiki.archlinux.org/title/pacman/Tips_and_tricks](https://wiki.archlinux.org/title/pacman/Tips_and_tricks)):

```bash
pacman -Qqn | pacman -S -
```

Если система не обновлялась долгое время, то нужно обновить пакет `archlinux-keyring` (информация взята с [https://wiki.archlinux.org/title/Pacman/Package_signing](https://wiki.archlinux.org/title/Pacman/Package_signing)):

```bash
sudo pacman -Sy --needed archlinux-keyring && sudo pacman -Su
```

# Настройка ОС

## Xfce

Установка `Xfce4` в качестве среды рабочего стола:

```bash
sudo pacman -S xorg xfce4 xfce4-goodies
# Шрифты для нормальной работы браузера
sudo pacman -S ttf-liberation ttf-dejavu ttf-hack noto-fonts ttf-opensans ttf-roboto
```

Добавляем в `.xinitrc`:

```bash
exec startxfce4
```

Запуск иксов:

```bash
startx
```

После установки `Xfce`, вы можете установить `sudo pacman -S firefox` и открыть этот гайд в браузере. Практически всегда, когда вы встречаете при установке пакетов предложения для выбора какой пакет установить, выбор по умолчанию является рекомендуемым выбором.

## Windows

`Arch Linux` может быть установлен бок о бок с `Windows`. Но следует (отключить)[] гибернацию и быстрый старт.

### Монтирование раздела с Windows

Для монитрования `NTFS` разделов с `Windows` понадобится пакет [ntfs-3g](https://wiki.archlinux.org/title/NTFS-3G):

```bash
sudo pacman -S ntfs-3g
```

Чтобы смонтировать `NTFS` раздел наберите:

```bash
mount /dev/your_NTFS_partition /mount/windows
```

Для автоматического монтирования при запуске системы нужно добавить в файл `/etc/fstab`:

```conf
UUID=0C12EF3A23FF133A /mnt/windows ntfs-3g defaults 0 0
```

### Время

Детали [тут](https://wiki.archlinux.org/title/System_time#UTC_in_Microsoft_Windows).

Если `Arch Linux` установлен вместе с `Windows`, то при переходе от одной ОС к другой у вас будет сбиваться время. Это связано с тем как ОС смотрят на время из аппаратных часов. `Arch Linux` считаем, что там `UTC`, а `Windows` локальное время. Нужно настроить операционные системы так, чтобы они воспринимали время из аппаратных часов одинаково. Рекомендуют делать так, чтобы `Windows` воспринимал время в `UTC`. Делается это запуском этой команды в коммандной строке `Windows` с правами администратора:

```cmd
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```

### Шрифты

О шрифтах есть страница на [вики](https://wiki.archlinux.org/title/fonts).

Если `Windows` установлена рядом с `Arch Linux`, то шрифты с `Windows` можно [установить](https://wiki.archlinux.org/title/Microsoft_fonts#Using_fonts_from_a_Windows_partition) с помощью символической ссылки на папку:

```bash
sudo ln -s /windows/Windows/Fonts /usr/local/share/fonts/WindowsFonts
sudo fc-cache --force
```

либо скопировать:

```bash
sudo mkdir /usr/local/share/fonts
sudo mkdir /usr/local/share/fonts/WindowsFonts
sudo cp /windows/Windows/Fonts/* /usr/local/share/fonts/WindowsFonts/
sudo chmod 644 /usr/local/share/fonts/WindowsFonts/*
sudo fc-cache --force
sudo fc-cache-32 --force
```

## Oh My Zsh

Установка [Oh My Zsh](github.com/ohmyzsh/ohmyzsh) фреймворка для конфигурации `zsh`:

```bash
sudo pacman -S git
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Одна из тем для `Oh My Zsh` - [powerlevel10k](https://github.com/romkatv/powerlevel10k).

Тема требует [специальные шрифты Meslo Nerd Font](https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#meslo-nerd-font-patched-for-powerlevel10k), которые можно установить в систему с помощью `gnome-font-viewer`, либо воспользоваться инструкцией с [вики](https://wiki.archlinux.org/title/fonts#Installation). На странице `powerlevel10k` есть инструкции как установить этот шрифт для разных эмуляторов терминала.

```bash
sudo pacman -S gnome-font-viewer
```

После установки `gnome-font-viewer` шрифты должны открываться по двойному клику и устанавливаться по нажатию кнопки `Install`.

## yay

[yay](https://github.com/Jguer/yay) - это [AUR Helper](https://wiki.archlinux.org/title/AUR_helpers), программа для автоматизации работы с [AUR](https://wiki.archlinux.org/title/Arch_User_Repository). `yay` также является оберткой над `pacman` и может устанавливать и обновлять обычные пакеты.

Для установки потребуются следующие пакеты:

```bash
sudo pacman -S fakeroot base-devel
```

Установка `yay`:

```bash
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

Установка пакетов с помощью `yay`:

```bash
yay package-name
```

Будет выведен список пакетов, которые подходят по названию `package-name`. Выбираете и устанавливаете нужный пакет.

Команда для обновление системы и всех пакетов из `AUR`:

```bash
yay
```

## Автоматический логин и запуск X после запуска системы

```bash
sudo systemctl edit getty@tty1
```

Написать это:

```conf
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin USER_NAME --noclear %I $TERM
```

В `.zprofile` это:

```bash
if [ "$(tty)" = /dev/tty1 ]; then
    exec startx
fi
```

## Сон и гибернация

Автоматический сон и гибернацию из-за бездействия я предпочитаю отключать, т.к. у меня обычно ОС из этих состояний не выходит и приходится перезагружать компьютер.

Отключается это [так](https://www.tecmint.com/disable-suspend-and-hibernation-in-linux/):

```bash
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

# Приложения

## mirrorlist

Мой `/etc/pacman.d/mirrorlist` (на 2024.05.13):

```conf
#Server=https://archive.archlinux.org/repos/2023/10/01/$repo/os/$arch
#Server=https://archive.archlinux.org/repos/2023/11/01/$repo/os/$arch
#Server=https://archive.archlinux.org/repos/2023/12/01/$repo/os/$arch
#Server=https://archive.archlinux.org/repos/2024/02/01/$repo/os/$arch
#Server=https://archive.archlinux.org/repos/2024/04/20/$repo/os/$arch
Server=https://archive.archlinux.org/repos/2024/05/10/$repo/os/$arch
```

## ru mirrorlist

[ru-mirrors.txt](https://gist.githubusercontent.com/psqq/9dc4e869b2200dc009821b08c99c866e/raw/aa9a19effd730c65bcb0cdc44e6810e2a5b3848e/ru-mirrors.txt) для начальной установки:

```conf
Server = http://mirror.yandex.ru/archlinux/$repo/os/$arch
Server = https://mirror.yandex.ru/archlinux/$repo/os/$arch
Server = http://mirror.surf/archlinux/$repo/os/$arch
Server = https://mirror.surf/archlinux/$repo/os/$arch
Server = http://mirror.nw-sys.ru/archlinux/$repo/os/$arch
Server = https://mirror.nw-sys.ru/archlinux/$repo/os/$arch
Server = http://mirrors.powernet.com.ru/archlinux/$repo/os/$arch
Server = http://mirror.rol.ru/archlinux/$repo/os/$arch
Server = https://mirror.rol.ru/archlinux/$repo/os/$arch
Server = http://mirror.truenetwork.ru/archlinux/$repo/os/$arch
Server = https://mirror.truenetwork.ru/archlinux/$repo/os/$arch
Server = http://archlinux.zepto.cloud/$repo/os/$arch
```

Загрузка по короткой ссылке: `curl -L bit.ly/30Lhmdf`.

## Ресурсы

При возникновении проблем и вопросов пользуемся [вики](https://wiki.archlinux.org/) (для большинства статей есть русский язык), [форумом](https://bbs.archlinux.org/), [lor](https://www.linux.org.ru/) и гуглом.