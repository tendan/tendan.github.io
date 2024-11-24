+++
title = 'GTA IV Installation on Linux Guide'
date = 2024-08-03T23:40:03+02:00
difficulty = 'Łatwy'
+++
Poradnik instalacji przeznaczony dla wersji 1.0.8 (Steam + RGL)

0. Zainstaluj GTA IV poprzez Steama
Za pierwszym razem zainstaluje również Rockstar Games Launcher, co zabierze dodatkowe 1 GB wolnego miejsca.
Zabawną rzeczą jest używanie przez RGL czcionek Windowsowych, przez co jeżeli ich nie macie, wyświetli Times New Roman (jak to było w moim przypadku).

1. Zainstaluj DXVK (https://github.com/doitsujin/dxvk/releases)
Najważniejsza rzecz do zainstalowania, żeby GTA IV działało poprawnie.
Jak wspomnieli jego autorzy, służy on do tłumaczenia API Direct3D na Vulkana, dzięki czemu uruchamianie na Linuxie nie powinno być już problemem.
Jeżeli używałeś tego, żeby zwiększyć wydajność na Windowsie (jak ja), będziesz mocno zaskoczony tym jak to dobrze działa na Linuxie (tak jak być powinno).
Odpakuj d3d9.dll i dxgi.dll z folderu x64 (lub x32 jeżeli jesteś na 32-bitowcu) do folderu "GTAIV" w folderze z grą ([...]/steamapps/common/Grand Theft Auto IV).

2. Zainstaluj ThirteenAG's FusionFix (https://github.com/ThirteenAG/GTAIV.EFLC.FusionFix)
Odpakuj zawartość archiwum do folderu "GTAIV" wymienionego wyżej.

3. Wyłącz wstępne buforowanie shaderów na Steamie
Wymagane, żeby DXVK w ogóle funkcjonowało. Ja głupi się zastanawiałem dlaczego nie odczuwam różnicy po instalacji, mając to włączone. Dodatkowo będziecie niepotrzebnie czekali za każdym odpaleniem gry na prerender shaderów.

4. Dodaj zmienną środowiskową WINEDLLOVERRIDES lub (jeżeli to nie działą) uruchom winecfg na prefixie z GTA IV i ustaaw natywne DLL-e: d3d9, dxgi i dinput8.
Konieczne dla Wine/Proton - ustawienie tego powstrzymuje je przed nadpisywaniem w/w bibliotek.
n - natywna (używa biblioteki dołączonej z Windowsem, bądź aplikacją (w naszym przypadku jest to GTA IV))
b - załączona (używa biblioteki dołączonej przez Wine'a lub Protona)

Jeżeli chcesz używać WINEDLLOVERRIDES:
W "Opcjach uruchamiania" wpisz: ```WINEDLLOVERRIDES="dinput8,dxgi,d3d9=n,b" %command```
Jeżeli chcesz (jak ja) nadpisać konfigurację Wine/Proton z użyciem winecfg:
```$ WINEPREFIX="<path_to_pfx_gta_iv_use>" winecfg```
W sekcji "Libraries" dodaj następujące biblioteki:
  dinput8 (native, bundled)
  dxgi (native, bundled)
  d3d9 (native, bundled)

5. Użyteczne flagi/opcje uruchamiania:
  -norestriction
  -percentvidmem 100
  -availablevidmem 4096
  -nomemrestrict
  -norestrictions
  -noprecache
  -novblank

Możesz je dodać do commandline.txt w folderze "GTAIV" lub wpisać je do "Opcji uruchamiania" we właściwościach GTA IV na Steamie.
