# Wstęp #

Pierwszym krokiem jest pobranie źródeł projektu z [repozytorium SVN](http://code.google.com/p/tesseract-polish/source/checkout).

Następnie należy zapoznać się z treścią pliku [README](http://code.google.com/p/tesseract-polish/source/browse/trunk/README) znajdującego się w pobranych źródłach.

# Co jest do zrobienia #

Kwestia plików słownikowych jest w zasadzie załatwiona, więc pozostało jedynie opracowywać nowe, wartościowe pliki `.box` towarzyszące plikom `.tif` z treningowymi obrazami.

Obecnie jestem na takim etapie, że jeszcze nie zdążyłem opracować plików `.box` dla  przykładów, które sam przygotowałem, w rozdzielczości 600 DPI.

W chwili pisania tej strony dysponowałem tylko plikami w rozdzielczości 300 DPI, każdy w trzech wariantach dla różnych czcionek (**courier**, **helvetica**, **times**), opracowanymi przeze mnie i [osoby współpracujące](http://code.google.com/p/tesseract-polish/source/browse/trunk/ATTRIBUTIONS) - patrz katalog [boxtiff.pol](http://code.google.com/p/tesseract-polish/source/browse/trunk/src/training_images/boxtiff.pol/).

# Co robić #

Tymczasem uruchamiając `make` w katalogu `training_images`, można uruchomić wygenerowanie (w oparciu o plik [sample\_pl.txt](http://code.google.com/p/tesseract-polish/source/browse/trunk/src/training_images/sample_pl.txt) wszystkich czterech przygotowanych przeze mnie serii tekstów treningowych.

Wymagane jest uprzednio posiadanie zainstalowanych narzędzi:
  * fold (na praktycznie każdym Linuxie jest)
  * cedilla (**uwaga**: na Ubuntu Hardy trzeba zainstalować dodatkowo `texlive` i `texlive-fonts-recommended` a następnie podlinkować go pod lokalizacją starego Tex-a texmf, bo Cedilla ma zakodowane jeszcze stare ścieżki: `ln -s /usr/share/texmf-texlive /usr/share/texmf-tetex`. [Ubuntu bug 148941](https://bugs.launchpad.net/ubuntu/+source/cedilla/+bug/148941), [Debian bug 427522](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=427522).
  * csplit
  * gs (Ghostscript)
  * tesseract z zainstalowanymi najnowszymi plikami dla języka Polskiego `tessdata/pol.*`, dostępnymi [tutaj](http://code.google.com/p/tesseract-polish/source/browse/trunk/tessdata/)

Wynikiem działania `make` będą (między innymi) pliki Postscript z dokumentami treningowymi.

Należy je wydrukować na papierze, a następnie wskanować w 600 dpi. Przy tym warto zwrócić uwagę, czy dokumenty te nie były już przez kogoś przetworzone dla danych czcionek w danej rozdzielczości DPI (obecnie wszystkie dokumenty ).

Wynikowe skany należy zapisać w katalogu `boxtiff.pol` w formacie TIFF z odcieniami szarości, skompresowanym LZW (mamy ograniczone miejsce w SVN!). Należy zadbać o to, żeby dysponować skanami wszystkich trzech wariantów dokumentu (dla czcionek **courier**, **helvetica** i **times**).

Plikom należy nadać nazwy niosące odpowiednie informacje - w tym rozdzielczość skanowania, model urządzenia drukującego, opcjonalnie model urządzenia skanującego. Przyjęty wzór nazwy to:

```
NAZWA_DOKUMENTU-NAZWA_CZCIONKI-MODEL_DRUKARKI-MODEL_SKANERA-ROZDZIELCZOSC_DPI.ROZSZERZENIE
```

Przykładowo:

```
sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.tif
```

W katalogu `boxtiff.pol` można zobaczyć również pliki o krótszych nazwach - są to pliki, które nie były drukowane i skanowane, lecz bezpośrednio skonwertowane z Postscript do TIFF na komputerze. Były używane do wstępnego treningu i nie ma już potrzeby tworzyć nowych w tej technologii.

Następnie, posiłkując się dokumentacją na temat trenowania Tesseract-a (http://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract - rozdział "Make box files"), przygotowujemy pliki box dla tych obrazów TIF (zwracamy uwagę, aby dodać `-l pol` do wywołania tesseract-a):

```
tesseract NAZWA_DOKUMENTU.tif NAZWA_DOKUMENTU.box -l pol batch.nochop makebox
mv NAZWA_DOKUMENTU.box.txt  NAZWA_DOKUMENTU.box
```

Edytujemy plik `NAZWA_DOKUMENTU.box` swoim ulubionym edytorem wspierającym kodowanie UTF-8.

Należy to robić **uważnie**, **pieczołowicie** i z **pełnym zrozumieniem celu i zasad działania tego procesu**. Jeśli dokument [TrainingTesseract](http://code.google.com/p/tesseract-ocr/wiki/TrainingTesseract) jest dla ciebie niezrozumiały, nie zajmuj się tym! Od jakości plików `.box` ściśle zależy przyszła skuteczność rozpoznawania tekstów.

Kiedy zakończymy prace nad plikiem `.box` dla jednego TIFF-a dotyczącego jednej czcionki, zabieramy się za `.box` dla analogicznego TIFF-a dotyczącego kolejnej czcionki.

# Sprawdzenie #

Po zakończeniu prac nad wszystkimi trzema plikami .box dla TIFF-ów dla wszystkich trzech czcionek, dokonujemy dodatkowego sprawdzenia.

Ponieważ TIFF-y te przedstawiają ten sam tekst wydrukowany różną czczionką, to ciąg tekstowy zawarty w każdym pliku `.box` powinien być identyczny z pozostałymi (znaki mogą być inaczej pogrupowane w drójki lub trójki, ale po ich uszeregowaniu powinny utworzyć ten sam ciąg).

Do sprawdzenia tego służą skrypty `check_box.pl` oraz `docdiff_checkfiles.sh`.

Pierwszy przyjmuje treść pliku `.box` na standardowym wejściu i emituje ciąg znaków (bez odstępów) na standardowym wyjściu.

Jeśli wszystkie pliki `.box` z tej samej partii (n.p. `part3`) są poprawne, to `check_box.pl` z każdego z nich wyprodukuje ten sam tekst.

Skrypt `docdiff_checkfiles.sh` wymaga zainstalowania narzędzia `docdiff`, pozwalającego na porównywanie plików znak po znaku.

Jako argumenty podajemy dwa pliki (z założenia o podobnej lub identycznej zawartości) i ujrzymy ich zawartość z wyrodrębnionymi wszelkimi różnicami.

Tak więc sprawdzenie plików `.box` przebiega następująco (na przykładzie plików `.box` dla partii `part1`):

```
./check_box.pl < sample_pl-part1-courier-scan300dpi.box > sample_pl-part1-courier-scan300dpi.check
./check_box.pl < sample_pl-part1-helvetica-scan300dpi.box > sample_pl-part1-helvetica-scan300dpi.check
./docdiff_checkfiles.sh sample_pl-part1-courier-scan300dpi.check sample_pl-part1-helvetica-scan300dpi.check
```

Jeśli są jakieś różnice pomiędzy plikiem `.box` dla skanu czcionki courier i helvetica, to zostaną one wyróżnione kolorami (`docdiff` trzeba uruchamiać na kolorowym terminalu ANSI!)

Różnice takie są wynikiem omyłki w pracy nad plikiem `.box` i wymagają jej poprawienia.

Ponieważ mamy trzy różne warianty dla trzech czcionek, to można je połączyć w trzy różne pary. Zatem dokonujemy w sumie trzech sprawdzeń (każda czcionka z każdą inną).

Dodatkowo sprawdzić możemy zgodność sum MD5 - powinny być identyczne dla plików `*.check` dla różnych czcionek lecz tego samego tekstu treningowego. Przykład sprawdzenia:

```
$ md5sum sample_pl-part3-*.check
7b05a0ef2a291157f545620dbf7114d9  sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.check
7b05a0ef2a291157f545620dbf7114d9  sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.check
7b05a0ef2a291157f545620dbf7114d9  sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.check
```

Jeśli mamy pewność co do poprawności plików `.box`, to dokonujemy próbnego treningu Tesseract (wytworzy on pliki danych w bieżącym katalogu):

```
$ ./train_tesseract.sh
checking sample_pl-part0-courier.box with output to sample_pl-part0-courier.check
checking sample_pl-part0-helvetica.box with output to sample_pl-part0-helvetica.check
checking sample_pl-part0-times.box with output to sample_pl-part0-times.check
checking sample_pl-part1-courier-digital_cam.box with output to sample_pl-part1-courier-digital_cam.check
checking sample_pl-part1-helvetica-digital_cam.box with output to sample_pl-part1-helvetica-digital_cam.check
checking sample_pl-part1-times-digital_cam.box with output to sample_pl-part1-times-digital_cam.check
checking sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.box with output to sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.check
checking sample_pl-part2-helvetica-scanner_HP_DeskJet_F4180_300dpi.box with output to sample_pl-part2-helvetica-scanner_HP_DeskJet_F4180_300dpi.check
checking sample_pl-part2-times-scanner_HP_DeskJet_F4180_300dpi.box with output to sample_pl-part2-times-scanner_HP_DeskJet_F4180_300dpi.check
checking sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.box with output to sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.check
checking sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.box with output to sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.check
checking sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.box with output to sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.check
Reading sample_pl-part0-courier.tr ...
Reading sample_pl-part0-helvetica.tr ...
Reading sample_pl-part0-times.tr ...
Reading sample_pl-part1-courier-digital_cam.tr ...
Reading sample_pl-part1-helvetica-digital_cam.tr ...
Reading sample_pl-part1-times-digital_cam.tr ...
Reading sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part2-helvetica-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part2-times-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.tr ...
Reading sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.tr ...
Reading sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.tr ...

Writing Merged Microfeat ...Done!
Reading sample_pl-part0-courier.tr ...
Reading sample_pl-part0-helvetica.tr ...
Reading sample_pl-part0-times.tr ...
Reading sample_pl-part1-courier-digital_cam.tr ...
Reading sample_pl-part1-helvetica-digital_cam.tr ...
Reading sample_pl-part1-times-digital_cam.tr ...
Reading sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part2-helvetica-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part2-times-scanner_HP_DeskJet_F4180_300dpi.tr ...
Reading sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.tr ...
Reading sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.tr ...
Reading sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.tr ...
Clustering ...

FreeTrainingSamples...
Writing normproto ...
Extracting unicharset from sample_pl-part0-courier.box
Extracting unicharset from sample_pl-part0-helvetica.box
Extracting unicharset from sample_pl-part0-times.box
Extracting unicharset from sample_pl-part1-courier-digital_cam.box
Extracting unicharset from sample_pl-part1-helvetica-digital_cam.box
Extracting unicharset from sample_pl-part1-times-digital_cam.box
Extracting unicharset from sample_pl-part2-courier-scanner_HP_DeskJet_F4180_300dpi.box
Extracting unicharset from sample_pl-part2-helvetica-scanner_HP_DeskJet_F4180_300dpi.box
Extracting unicharset from sample_pl-part2-times-scanner_HP_DeskJet_F4180_300dpi.box
Extracting unicharset from sample_pl-part3-courier-scanner_Brother_DCP-135C_300dpi.box
Extracting unicharset from sample_pl-part3-helvetica-scanner_Brother_DCP-135C_300dpi.box
Extracting unicharset from sample_pl-part3-times-scanner_Brother_DCP-135C_300dpi.box
Wrote unicharset file ./unicharset.

Don't forget to correct pol.unicharset if your iswalpha/iswdigit functions malfunction (no pun intended)!
```

Proces ten wytworzy pliki `.log` odpowiadające plikom `.box`. Sprawdzamy je w poszukiwaniu niepożądanych komunikatów świadczących o błędach (n.p. zachodzących na siebie box-ach, które są ignorowane podczas treningu).

Po poprawieniu wszystkich pomyłek i niezgodności w plikach `.box`, przesyłamy używane pliki `.tif` oraz `.box` do opiekuna projektu, czyli mnie - **[aleksander dot adamowski at olo.org.pl](http://olo.org.pl)**.

# W czym jeszcze można pomóc #

## Słowniki ##

Może się jeszcze przydać pomoc w opracowaniu odpowiednich słowników DAWG języka Polskiego.

Mają one w formie źródłowej postać pliku tekstowego w UTF-8 zawierającego po jednym słowie na wiersz.

Potrzebne są dwa takie pliki słownika: słów najczęściej używanych (maksymalnie 1500 słów) oraz wszelkich słów pozostałych we wszelkich odmianach (ale nie więcej, niż milion, słownik z http://www.sjp.pl zawiera prawie 3,5 miliona, zatem jest za duży).

Dla mnie najlepszą formą byłaby baza słów (we wszystkich odmianach) wzbogacona o informacje o ich częstotliwości występowania w języku. Mogłaby mieć na przykład postać pliku tekstowego z wierszami w formacie "SŁOWO TABULATOR LICZBA\_WYSTĄPIEŃ" lub pliku bazy Berkeley DB zawierającego mapę SŁOWO -> LICZBA\_WYSTĄPIEŃ.

Dane o częstotliwości lub liczbie wystąpień danej formy słowa powinny być wyliczone w oparciu o dane z jakiegoś korpusu języka Polskiego. Próbowałem tego rodzaju statystyki wydobyć z korpusu 250 mln-segmentowego ze strony http://www.korpus.pl/, ale ma on postać binarną i mi się to nie udało.

Właściwie, gdyby ktoś mi udostępnił wystarczająco duży i reprezentatywny zbiór tekstów w języku Polskim, mógłbym samodzielnie wygenerować taką bazę.

Obecny słownik DAWG najczęściej używanych wyrazów utworzyłem na bazie korpusu Słownika frekwencyjnego polszczyzny współczesnej (http://www.korpus.pl/index.php?page=download - plik `frek.bin.tar.bz2`), a słownik wyrazów pozostałych na bazie starego słownika do ispell-a (http://ispell-pl.sourceforge.net/).

## Ocena efektywności obecnej wersji ##

Aby zorientować się, czy prace idą we właściwym kierunku oraz gdzie występują najczęściej błędy, należy przeprowadzać okresowe testy skuteczności rozpoznawania znaków.

Przyda się pomoc osoby, która jest chętna takie raporty opracowywać.

Opracowanie pierwszego raportu powinno polegać na przygotowaniu reprezentatywnych skanów jakichś rzeczywistych tekstów oraz przepuszczeniu ich przez Tesseract wyposażony w nasze pliki `pol.*`.

Raport powinien zawierać statystyki na temat ilości błędów oraz wskazywać najczęściej powtarzające się błędy - co pozwoli skupić się na nich w kolejnych sesjach treningowych.

W kolejnych raportach należy skorzystać z tych samych skanów aby wyniki można było porównywać ze sobą.