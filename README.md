# Что это?

protonk - это скрипт, который позволяет использовать `proton` без steam. Например, с помощью этой утилиты можно установить игру из GOG или использовать уже распакованные приложения для Windows.

protonk не реализует никаким образом функционал `proton`. Лишь настраивает и использует `proton` так, чтобы можно было запускать игры/приложения без steam.

Большинство строчек кода взяты [отсюда](https://github.com/Francesco149/protonfit/blob/master/protonfit).

Дисклеймер: скрипт носит экспериментальный характер. Использование `proton` вне среды Steam крайне нежелательно по целому ряду причин (*со слов многоуважаемого GloriousEggroll*):

- В `proton` отсутствует компонент Wine `wineboot`, необходимый для корректного создания префиксов.

- Не создаётся параметр среды со `Steam Game ID`, из-за чего фиксы, имплементированные для конкретной игры не будут работать. Но можно попробовать поменять переменную среды `SteamGameId` на ID нужной игры.

- То же самое для компонента `gstreamer`. Некоторые видеозаписи (игровые ролики) не будут воспроизводиться.

- `proton` процесс запускается не от стандартного имени пользователя, что влечёт за собой определённые последствия.

- `proton` опирается на библиотеки, предоставленные в `Steam Runtime`. Если их не будет хватать в системе — будут сбои. 

Лучше использовать другие решения. Например, [lutris](https://github.com/lutris/lutris) + [wine-ge](https://github.com/GloriousEggroll/wine-ge-custom). Вы были предупреждены.

# Перед использованием

На самом деле, все эти действия (кроме установки steam) всё равно придётся проводить, даже если не будет использоваться protonk.

## Убедитесь, что ваша видеокарта поддерживает `Vulkan API`

`DXVK`, который активно используется в `proton`, требует наличия поддержки `Vulkan API` со стороны видеокарты. Если ваша видеокарта не поддерживает `vulkan`, то все дальнейшие манипуляции, вероятнее всего, будут непродуктивны. Работать, может быть, будет, но вот производительность сильно упадёт.

## Подготовка системы

Здесь предлагается вам определиться с тем, какое ядро (см. дальше) вы собираетесь использовать.
А также вам нужно удостовериться, что вы используете самые *свежие стабильные* (если такие найдёте=/) драйвера на видеокарту.

Есть 2 варианта установки:

### Стандартное ядро linux

Стандартное ядро будет, конечно, работать в играх, но при игре могут наблюдаться некоторые проблемы: микрофризы и зависания системы. Это связанно с тем, что ядро Linux по умолчанию подстраивается под нормальную работу с системой, а игры уже не являются для него приоритетом.

Поскольку оно уже у вас установленно, перейдём сразу к обновлению драйверов. Установите драйвера на вашу видеокарту. В зависимости от дистрибутива и производителя установка будет отличаться. Например, для `Xubuntu 20.04 LTS` *(лучше, конечно, ставить не LTS вариант системы)* с видеокартой от **nvidia** драйвера рекомендуется установить проприетарный драйвер:

``` bash
# Посмотреть какая у вас видеокарта:
> sudo lspci -vnn | grep -i VGA -A 12

# Посмотреть какие драйвера доступны:
> ubuntu-drivers devices

# И установка, например, видеодрайвера версии 460:
> sudo apt install nvidia-driver-460
```

В случае если вы гордый пользователь карты от **AMD**, то рекомендуется использовать связку драйверов `amdgpu + mesa` для лучшей производительности. Они включены прямо в ядро Linux.

### Улучшенное ядро linux-tkg

`linux-tkg` - это ядро Linux, которое идёт вместе с набором патчей, направленных на увеличение производительности в играх. Поддерживает такие технологии как `ESYNC`, `FSYNC` и `FUTEX2`, так и патчи ядра, направленные на улучшение поддержки системы с архитектурой процессора. Всё это приносит ощутимый прирост производительности как в играх, так и прибавку в общей отзывчивости системы. Дополнительную информацию вы можете найти [на этой странице.](https://github.com/Frogging-Family/linux-tkg).

Предупреждаем, что процесс компиляции может занять некоторое время (1-3 ч.).

К примеру, шаги компиляция ядра для `Xubuntu 20.04 LTS` выглядят так:

``` bash
> git clone https://github.com/Frogging-Family/linux-tkg

> cd linux-tkg

> bash ./install.sh install
```
После этого выбирайте пункты среди предложенных, исходя из вашего оборудования и цели. От себя добавлю, что рекомендуется выбрать среди доступных опций:

- Планировщик ЦПУ (CPU scheduler) - `PDS` (лучше подходит для игр).

- Обязательно включить в сборку поддержку `ESYNC`, `FSYNC` и `FUTEX2`.

- Корректно выбрать вашу архитектуру процессора. Например, могут возникнуть сложности с CoffeLake архитектурой, потому что вы не найдёте её в списке (оказывается, [CoffeLake относится к SkyLake](https://github.com/Frogging-Family/linux-tkg/issues/33)). Гугл вам в помощь.

В случае, если вы завистливый владелец [видеокарты от nvidia](https://www.youtube.com/watch?v=_36yNWw_07g), то не торопитесь перезагружать систему, ибо данное ядро не очень хорошо ладит со стандартным драйвером. Вам нужно будет поставить его [DKMS](https://ru.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) версию.
Например, для `Xubuntu 20.04 LTS` и видеокартой от **nvidia** это может делаться следующим образом:

``` bash
> sudo apt update && sudo apt upgrade # лучше выполнить эти команды, чтобы потом исключить лишние проблемы

# Посмотрите, какие варианты есть
> apt search "nvidia-dkms"

# Удалите все текущие драйвера
> sudo apt-get purge nvidia* # Да, здесь apt-get, а не просто apt

# Установите выбранный dkms драйвер. Например, версия 460
> sudo apt install nvidia-dkms-460 # Команда должна скомпилировать модули для каждого ядра

> sudo reboot now

# Если вам нужны nvidia-утилиты (nvidia-smi, nvidia-bug-report.sh и др), то можете установить их командой:
> sudo apt install nvidia-utils-460 # число 460 должно совпадать с версией драйвера
```

### arch

На арче всё будет попроще. Вообще считается, что на `arch` играть лучше, чем на Debian-like. Например, вот набор команд, которые необходимы, если у вас `arch` и видео-карта `nvidia`:

``` bash
# Компиляция и установка tkg-ядра:
> git clone https://github.com/Frogging-Family/linux-tkg
> cd linux-tkg
> makepkg -si

# Отредактируйте grub как вам необходимо. Например, с помощью этой утилиты:
> grub-customizer

# Перезагрузитесь
> reboot

# Убедитесь, что вы загрузились в правильное ядро:
> uname -r 
# или
> cat /proc/version

# Проверьте, может быть у вас уже установлены необхоимые драйвера:
> lspci -k | grep -A 2 -i "VGA" | grep "Kernel driver in use:"
# Если Kernel driver in use: nouveau, то продолжайте вводить команды ниже
# Если Kernel driver in use: nvidia, то в теории всё ок. Можно переходить к следующему пункту

# Время ставить дрова на видеокарту nvidia:
> git clone https://github.com/Frogging-Family/nvidia-all.git
> cd nvidia-all
> makepkg -si

# Перезагрузитесь
> reboot

# Надеюсь, получилось загрузиться в систему)
```

## Установите steam

Установите и запустите хотя бы раз до окошка с авторизацией steam. Авторизоваться НЕобязательно. На `Xubuntu 20.04 LTS` это можно сделать следующим образом:

``` bash
> sudo apt update && sudo apt install steam

> steam # подождите, пока появится приглашение авторизоваться,
        # дальше можете спокойно нажимать Ctrl+C, если не хотите авторизоваться
```

## Загрузите копию proton

Рекомендуется использовать модифицированную сборку `proton`, поскольку он включает в себя несколько важных фиксов, отсутствующих в официальном билде. На момент написания этого текста лучше всего использовать [proton-ge](https://github.com/GloriousEggroll/proton-ge-custom).

1. Перейти в раздел с релизами: https://github.com/GloriousEggroll/proton-ge-custom/releases

1. Загрузите самую свежую версию.

1. Разархивируйте, например, в эту директорию: `~/.local/share/proton-ge/`. Вы также можете использовать proton-ge вместо обычного протона для steam ([подробнее здесь](https://github.com/GloriousEggroll/proton-ge-custom#installation)).

## Укажите, где находится угодный вам proton

Создайте переменную среды `PROTON`, которая бы содержала путь до proton (именно "исполняемого" файла, а не директории):

``` bash
> echo -e "\nexport PROTON=/path/to/proton" >> ~/.bashrc
```

Например, если в предыдущем пункте была бы выбрана версия `Proton-6.15-GE-2 released`, то команда выглядела бы так:

``` bash
> echo -e "\nexport PROTON=$HOME/.local/share/proton-ge/Proton-6.15-GE-2/proton" >> ~/.bashrc
```

## Ещё пару пакетов

Ещё установите следующие пакеты: `wine`, `winetricks`. На `Xubuntu 20.04 LTS` это делается следующий образом:

``` bash
> sudo apt install wine winetricks

# или "нестабильная" самая новая версия
> wget -nc https://dl.winehq.org/wine-builds/winehq.key
> sudo apt-key add winehq.key
> sudo apt-add-repository 'https://dl.winehq.org/wine-builds/ubuntu/'
> sudo apt update
> sudo apt install --install-recommends winehq-staging
> sudo apt install winetricks
```

На `arch` так:

``` bash
> sudo pacman -Sy
> sudo pacman -S wine-staging winetricks
> sudo pacman -S giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo libxcomposite lib32-libxcomposite libxinerama lib32-libxinerama ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader cups samba dosbox
```

Также рекомендуется прочитать подробнее [тут](https://www.gloriouseggroll.tv/how-to-get-out-of-wine-dependency-hell/): https://www.gloriouseggroll.tv/how-to-get-out-of-wine-dependency-hell/

# Установка

Скачайте файл из репозитория protonk, сделайте его исполняемым и переместите в деректорию, которая есть в `$PATH` (выполните команду *«echo $PATH»*). Например, установка может выглядеть так:

``` bash
> wget -O protonk https://raw.githubusercontent.com/The220th/protonk/master/protonk

> chmod +x ./protonk

# Если ~/.local/bin/ есть в $PATH, иначе выберите другую директорию
> mv ./protonk ~/.local/bin/
```

# Использование

Перейдите в каталог с игрой/приложением:

``` bash
> cd /path/to/game
```

Затем "запустите" `exe файл` следующим образом:

``` bash
> protonk run ./file.exe
```

После этого в директории, где вы "находитесь", появится папка `./pfx`. [Это "бутылка" (или префикс) wine](https://bit.ly/3ysl6eO). Там будет среда вашей игры/программы. Рекомендуется создавать каждый раз такую "бутылку" для каждого приложения.

## winetricks

Чтобы запустить `winetricks` для префикса, то перейдите в директорию, где находится папка `./pfx`, и выполните команду:

``` bash
> protonk tricks
```

Вы можете использовать те же аргументы, что и для winetricks. Например, установка `directx9`, `vcrun2019`:

``` bash
> protonk tricks directx9
#и/или
> protonk tricks vcrun2019
```

## One prefix to rule them all

Если по какой-то причине нужен один префикс для всех приложений и игр (так делать не рекомендуется), то определите переменную среды `PROTONK_ONEPREFIX`, которая должна указывать на директорию, где будет этот самый префикс `pfx`. Также теперь команда `protonk tricks` будет менять именно этот самый один на всех префикс.

## Параметры запуска

Вы можете создать файл `protonk-settings` в той же директории, что и `.exe` файл. В этом файле могут быть параметры запуска для proton. Например, файл `protonk-settings` может выглядеть так:

``` bash
export PROTON_LOG_DIR="$PWD"
export PROTON_CRASH_REPORT_DIR="$PWD"
export PROTON_LOG=1
export WINEDEBUG=+cmd
#export WINEDEBUG=-all
export SteamGameId=0

export MANGOHUD=1 # enable mangohud https://github.com/flightlessmango/MangoHud

#export WINE_FULLSCREEN_FSR=1 # enable FSR

export DXVK_LOG_LEVEL="info"
#export PROTON_USE_WINED3D="1"
#export PROTON_USE_WINED3D11="1"
#export PROTON_NO_D3D11="1"
#export PROTON_NO_D3D10="1"
#export PROTON_NO_D9VK="1"
#export PROTON_NO_ESYNC="1"
#export PROTON_USE_VKD3D="1"       #<-----DX12 support
#export PROTON_NO_FSYNC="1"
#export PROTON_FORCE_LARGE_ADDRESS_AWARE="1"
#export PROTON_OLD_GL_STRING="1"
#export PROTON_USE_SECCOMP=1
#export RADV_PERFTEST=aco # enable ACO compiler for less shader stutter on amd cards
```

Посмотреть эти параметры можно, например, [тут](https://github.com/ValveSoftware/Proton#runtime-config-options).

Если не сработало и настройки не подтянулись, то попробуйте сделать файл исполняемым:

``` bash
chmod +x ./protonk-settings
```

# QA

## Как показывать информация по температуре, нагрузке CPU/GPU и т. п.

Хорошим решением будет установить [mangohud](https://github.com/flightlessmango/MangoHud). После установки устанавливайте меременную среды `MANGOHUD` в единичку:

``` bash
MANGOHUD=1 %command%
```

## Как лочить fps

Самый простой способ - это использовать опять-таки [mangohud](https://github.com/flightlessmango/MangoHud). Организуйте строку `fps_limit={нужное значение}` в `~/.config/MangoHud/MangoHud.conf`.

## Есть ли аналог MSI-Afterburner

Попробуйте [GreenWithEnvy](https://gitlab.com/leinardi/gwe)

## Джунгли AMD драйверов на видеокарту

Посмотрите [тут](https://www.linux.org.ru/forum/multimedia/15587007).

## Вылетает на AMD-видеокарте

Много из-за чего может возникнуть проблема.

Попробуйте удалить `AMDVLK`.

## В играх фризит при появлении "чего-то нового"

При запуске задайте переменную среды `DXVK_ASYNC=1`. Это позволит "рендерить шейдеры" не "останавливая" игру, а прямо в процессе. Но тогда некоторые "текстуры и эффекты" могут появиться не сразу.

