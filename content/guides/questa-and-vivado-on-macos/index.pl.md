+++
title = 'Setup Questa and Vivado on MacOS'
date = 2026-08-19T20:56:41+02:00
draft = true
tags = ["questasim", "vivado", "macos", "apple", "amd", "siemens", "altera", "orbstack", "enterprise-linux"]
+++

1. Przygotowanie środowiska pod narzędzia EDA na macOS
Należy zainstalować OrbStack'a, aktualnie jest on najlepszym sposobem na korzystanie z maszyn wirtuanych na Linuxie.
Osobiście polecam wybranie dystrybucji z rodziny RHEL, gdyż zapewniają one najwyższą kompatybilność z (leciwymi) narzędziami do pracy pod ASIC/FPGA. Mój wybór padł na Rocky Linux.
Koniecznie należy wybrać architekturę x86_64, jako że każdy program w tej dziedzinie technicznej jest z myślą o tej architekturze pisany. OrbStack wykorzystuje sprzętową Rosettę do emulacji, więc skok wydajności będzie mocny w porównaniu z kontenerami Dockerowymi.

(tu wstaw zdjęcie z OrbStacka z tworzenia VM)

Zależności jakie są potrzebne:

```sh
# Rocky VM
$ sudo dnf install -y epel-release
$ sudo dnf install -y openssh-server xorg-x11-xauth xclock libX11 libXrender libXtst libXi make gcc tar gzip
```
(```openssh-server``` może wydawać się zaskakujący, ale wyjaśnienie pojawi się niżej)

2. Zainstalowanie i wstępna konfiguracja XQuartz'a

XQuartz jest najlepszym sposobem na uruchamianie okienek wykorzystując protokół X11 z np. zdalnych połączeń.

Najprostszy krok w tym poradniku, pobieramy XQuartz z np. Homebrew:

```sh
# macOS
$ brew install --cask xquartz
```

W jego opcjach należy włączyć poniższe:

(tu screenshot z ustawień XQuartz związanymi z połączeniem SSH)

3. Pobranie instalatorów QuestaSim i Vivado Design Suite ze stron Altery i AMD
Na hobbystyczny użytek korzystam z Questa FPGA Starter Edition (stąd w poradniku będzie widoczny instalator do tejże wersji) oraz Vivado w nienajnowszej wersji - 2025.2 ([tutaj powód](https://www.reddit.com/r/FPGA/comments/1thstyc/vivado_20261_basic_limited_debugging_xsim/?show=original)) tl;dr AMD postanowiło, że bezpłatna wersja będzie bardziej okrojona niż dotychczas - pierwotnie to nawet planowali nie wypuszczać jej na Linux'a.

Nie trzeba tu za bardzo tłumaczyć, więc opowiem anegdotkę. Załatwienie licencji, odkąd Altera objęła system SSLC całkowicie, jest łatwiejsze niż jeszcze rok temu. Pamiętam jak musiałem czekać miesiąc na to, aż Intel pozwoli mi założyć konto - na swoje nieszczęście poprosiłem o to przed Świętami Bożego Narodzenia. Potem się okazało, że konieczne jest utworzenie 2FA - na szczęście tu wystarczyło jedynie 15 minut konsultacji mailowej z no-reply.

*WAŻNE*: Na stronie Altera SSLC należy przy rejestracji podać adres MAC WIDOCZNY w VM-ce OrbStack'a:

```sh
# Rocky VM
$ ip link
...
2: eth0@if8: [...]
    link/ether xx:xx:xx:xx:xx:xx <----- tu powinien być twój adres MAC do podania na stronie Altery
```

Reszta wygląda tak jak przy pozyskiwaniu pliku z licencją .dat ze strony Altery. [https://www.youtube.com/watch?v=s9sJEv9YApY](https://www.youtube.com/watch?v=s9sJEv9YApY)

Wrzucanie plików na maszyny OrbStacka jest banalnie proste. OrbStack załącza dysk sieciowy z systemami plików dla każdej maszyny, do którego dostęp mamy chociażby z poziomu Findera (co nie jest wcale takie oczywiste). 

(tutaj wrzuć zdjęcie z NFS OrbStacka)

Dostęp do systemu plików każdej z utworzonych maszyn jest możliwy nawet gdy są wyłączone. Dopiero wyłączenie OrbStacka pozbawia nas możliwości wygodnego przeglądania.

4. Konfiguracja serwera SSH
OrbStack zapewnia wbudowany serwer SSH do łączenia się z VM-ką na poziomie macOS. Problem polega na tym, że nie wspiera on ```X11Forwarding```-u. Stąd konieczne jest uruchomienie ```openssh-server``` pobranego wcześniej:

```sh
$ systemctl start sshd
```

Warto w OrbStacku wyłączyć przekazywanie sieci lokalnych VM-ów poza naszego Maka:

(tu wstaw screenshot ze stosowną opcją w OrbStacku)

Koniecznie należy ustawić hasło dla naszego użytkownika za pomocą:
```sh
# Rocky VM
$ sudo passwd
```

Po tych krokach powinniśmy móc łączyć się z VM-ką w następujący sposób:
```sh
# macOS
$ ssh -i ~/.orbstack/ssh/id_ed25519 -Y <user>@<machine_name>.orb.local
```
```user``` to nazwa użytkownika jakiego utworzyliśmy na VM-ce, a ```machine_name``` to nazwa VM-ki podana w OrbStacku.

5. Instalowanie QuestaSim
Instalator QuestaSim w wersji graficznej używa instrukcji procesora, które akurat przez Rosettę nie są tłumaczone, stąd wita nas taki błąd:
```Illegal instruction```
(serio, niczego innego nie dostajemy)

Możemy zainstalować bez GUI (chyba najlepsza opcja, choć ściana tekstu dla licencji, którą trzeba Enter-em przeklikać, może zniechęcić) wpisując:
```sh
# Rocky VM
$ ./<questa_installer> --mode text
```

albo za wszelką cenę uruchomić instalator na Qt. W tym celu należy wyłączć Rosettę na czas instalacji Questy:
```sh
# macOS
$ orb config set rosetta false
$ orb stop
$ orb start
```

Zastosowałem u siebie tę drugą opcję i wiem, że ona na pewno działa. Po instalacji można przywrócić Rosettę z powrotem.

QuestaSim należy uruchamiać z podanym parametrem ```SALT_LICENSE_SERVER```, jeżeli mamy plik to podajemy bezwzględną ścieżkę do niego.

Przykładowy command line uruchamiający Questę:
```sh
# Rocky VM
$ SALT_LICENSE_SERVER=/home/tendan/LR-178485_License.dat ~/altera/25.1std/questa_fse/bin/vsim -gui
```

6. Instalacja Vivado

Tutaj nie ma większej filozofii, uruchamiamy instalator Vivado (ja wybieram zawsze Vitis), nie instalowałem Xilinx USB Cable Drivers (TBA czy działa przez OrbStacka).
*UWAGA!* Instalator na splash screenie może gwałtownie migać, a sam ma w większości białe kolory. Po przejściu do faktycznej instalki wszystko wraca do normy.

7. Naprawa Vivado

Zbyt pięknie by było, gdyby Vivado działało bez zarzutu od razu po instalacji, prawda? Pierwsze co zwróci waszą uwagę to najpewniej interfejs, który laguje niemiłosernie. 

W tej sytuacji należy pod Vivado stworzyć sesję SSH z następującymi parametrami:

```sh
$ ssh [[..]] -C <user>@<vm-name>.orb.local
```
```[[..]]``` oznacza po prostu resztę parametrów, które są zwykle wykorzystywane (w tym np. ```-Y```).

W pliku konfiguracyjnym powłoki VM-ki (najpewniej .bashrc) należy dodać:
```sh
export _JAVA_OPTIONS="-Dsun.java2d.xrender=false"
```
albo podawać za każdym razem jak uruchamiane jest Vivado.

Kolejnym problemem z jakim się zmierzycie to próba uruchomienia syntezy czy co gorsza implementacji. Obie te rzeczy kończą się crashem Vivado, choć tu ciekawostka - dla drobnych projektów (jak np. demo multipleksera) jest spora szansa, że synteza zakończy się pomyślnie jeszcze przed wykrzaczeniem się programu. Na wynik implementacji w takich okolicznościach bym nie liczył.

Z góry chciałbym podziękować autorowi tego [projektu](https://github.com/filmil/vivado-docker) na GitHub-ie, za przygotowanie udev_stub.c, którego należy skompilować i podrzucić do katalogu, który będzie widziany przez Vivado:
```sh
# Rocky VM
gcc udev_stub.c -shared -fPIC -o udev_stub.so
sudo mv udev_stub.so /opt/udev_stub.so
```

Niestety Vivado jest na tyle wybredne, że nie przyjmie od tak dowolnej biblioteki. Koniecznie trzeba mu podawać nazwy, które on dobrze kojarzy. Stąd też podmieniamy jego referencję dla libudev bez ruszania bibliotek systemowych: 
```sh
# Rocky VM
$ mkdir -p ~/fakelib
$ cp /opt/udev_stub.so ~/fakelib/libudev.so.1
$ ln -sf libudev.so.1 ~/fakelib/libudev.so
export LD_LIBRARY_PATH=~/fakelib:$LD_LIBRARY_PATH
```

8. Wykorzystanie Automatora do ikonek w Launchpadzie pod Questę i Vivado

Na sam koniec zostawiłem bonus w postaci uruchamiania obu programów z poziomu macOS-a. Jedynym warunkiem jest oczywiście uruchomiona OrbStackowa wirtualna maszyna.

W Automatorze należy utworzyć nową Aplikację. Obie apki zwyczajnie uruchamiają skrypt powłoki.

Dla Questy:
```zsh
# Questa.app
open -a XQuartz

if [ -z "$DISPLAY" ]; then
    export DISPLAY=$(ls -t /private/tmp/com.apple.launchd/*/org.xquartz:0 2>/dev/null | head -n 1)
fi

/usr/bin/ssh -i ~/.orbstack/ssh/id_ed25519 -Y <user>@<nazwa_vm>.orb.local "SALT_LICENSE_SERVER=<path_to_license_file.dat> <path_to_vsim> -gui"
```

Dla Vivado:
```zsh
# Vivado.app
open -a XQuartz

if [ -z "$DISPLAY" ]; then
    export DISPLAY=$(ls -t /private/tmp/com.apple.launchd/*/org.xquartz:0 2>/dev/null | head -n 1)
fi

/usr/bin/ssh -i ~/.orbstack/ssh/id_ed25519 -Y -C <user>@<nazwa_vm>.orb.local "export LD_LIBRARY_PATH=~/fakelib:$LD_LIBRARY_PATH ; export _JAVA_OPTIONS=\"-Dsun.java2d.xrender=false\" ; source <path_to_settings64.sh> && <path_to_vivado>"
```

Dla dopełnienia całości można dodać ikony pod utworzone w ten sposób apki. Wystarczy wejść w "Informacje" konkretnej i klikając na jej ikonę (domyślnie robot z Automator-a) i wklejając docelową, skopiowaną wcześniej ze schowka.
