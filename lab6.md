# UNIX ISI Lab 6: BASH

## Zakres

- Podstawy BASH.

## Polecenia

| Grupa            | Komenda                       |
| ---------------- | ----------------------------- |
| Inne             | alias, basename, file         |

## Przydatne linki

1. <https://bash-prompt-generator.org/>
2. <https://phoenixnap.com/kb/bash-redirect-stderr-to-stdout>
3. <https://www.cyberciti.biz/faq/bash-while-loop/>
4. <https://www.cyberciti.biz/faq/unix-howto-read-line-by-line-from-file/>
5. <https://itngineer.com/tworzenie-wlasnych-polecen-w-shellu-linux/>
6. <https://www.cyberciti.biz/faq/bash-get-basename-of-filename-or-directory-name/>
7. <https://linuxhint.com/bash_operator_examples/#o13>
8. <https://www.cyberciti.biz/faq/bash-for-loop/>
9. <https://linuxize.com/post/bash-if-else-statement/>
10. <https://linuxhint.com/30_bash_script_examples/>

## Istotna poznana składnia
`$0` - nazwa polecenia, które jest wykonywane wraz ze ścieżką do polecenia
`$1`, `$2`, ... - kolejne argumenty uruchomienia polecenia
`$#` - liczba argumentów uruchomienia polecenia
`$*` - wszystkie podane argumenty (poza nazwą polecenia) w formie jednej zmiennej


## Pytania wprowadzające
1. Jaka jest różnica między pojedynczym cudzysłowiem ('), a podwójnym (") w bashu?
2. Jak się tworzy alias permanentnie? Jak się sprawdza jakie się ma aliasy?
3. Co to są zmienne środowiskowe? Jak się je tworzy?

## Zadania

1. Dodaj alias `lst`, który będzie pokazywał ostatnio zmodyfikowane pliki w bieżącym katalogu, zrób zmianę permanentną.
2. Zmień swój znak zachęty `bash`, tak aby był 2-linijkowy, w 1-szej linii pokazywał:   
aktualny katalog oraz liczbę zadań w tle, w drugiej linii znak $.   
Zaprogramuj w/w zmianę tak, aby było to domyślne ustawienie po zalogowaniu.
3. Skopiuj wszystkie pliki z katalogu [zasoby](./zasoby) z repozytorium do katalogu `~/lab6`.  
Można je skopiować z katalogu `/home/wojnicki/lab/zasoby`.    
Napisz polecenie, które zmieni nazwy plików w katalogu ~/lab6 z kończących się na .JPG na kończące się na .jpg.   
Podpowiedź: skorzystaj z pętli for oraz `basename`.
4. Zapisz polecenie z poprzedniego zadania w pliku ~/bin/JPG2jpg, tak aby można je
było uruchomić jako polecenie `JPG2jpg`   
Zmodyfikuj je tak, aby:
    - jeżeli polecenie nie ma argumentu, konwersja nazw plików realizowana była w bieżącym katalogu,
    - jeżeli polecenie ma 1 argument, i jest to katalog, wykonaj konwersję w tym katalogu,
    - jeżeli polecenie ma opcje -h, wyświetl informacje o tym co robi i jaka jest składnia.   
    Sprawdź, czy ~/bin jest w zmiennej środowiskowej PATH, jeżeli nie, to odpowiednio zmodyfikuj ~/.bashrc.
5. Napisz polecenie, które usunie z katalogu ~/lab6 pliki, które nie są obrazkami JPG.   
Podpowiedź: użyj pętli `for` oraz `file`.
6. Zaprogramuj przypominacz, narzędzie, które przypomni o określonej dacie określoną
treść.   
W pliku ~/reminder umieść daty i treści, o których przypominacz ma przypomnieć, 1 linijka 1 przypomnienie, w formacie:   
YYYY-MM-DD tekst do przypomnienia
    - Napisz skrypt, który wyświetli przypomnienia dla dzisiejszej daty.
    - Zaprogramuj uruchomienie w/w skryptu po zalogowaniu się na serwerze, tak aby przypomnienia były wyświetlone w terminalu.
7. \* Utwórz polecenie `stronnicuj`, które wykonuje podział plików w katalogu na mniejsze grupy (strony).
    1. Polecenie powinno przyjmować następujące argumenty:
        1. **Katalog źródłowy**: katalog, który ma być podzielony na mniejsze grupy plików. Jeśli nie podano argumentu, przyjmij domyślnie katalog bieżący.
        2. **Maksymalna liczba plików w katalogu (N)**: liczba plików w każdej grupie/katalogu. Domyślnie ustaw na 10.
        3. **Zachowanie struktury katalogów**: wartość `true` lub `false`, która określa, czy należy zachować strukturę podkatalogów z katalogu źródłowego. Domyślna wartość to `false`.
        4. **Katalog docelowy**: opcjonalny argument określający katalog, w którym mają zostać zapisane podzielone pliki. Jeśli nie podano katalogu docelowego, przyjmij domyślnie katalog o nazwie:
        `{nazwa_katalogu_źródłowego}_stronnicowany`.
    2. Opis działania:
        - Polecenie powinno dzielić pliki z katalogu źródłowego na mniejsze grupy, zapisując je w katalogach docelowych oznaczonych kolejnymi numerami (np. `1`, `2`, `3` itd.).
        - Jeśli wartość trzeciego argumentu to `true`, struktura podkatalogów powinna zostać zachowana. Oznacza to, że pliki będą kopiowane do odpowiednich podkatalogów w katalogu docelowym:
        - Przykład: `src/core/header.h` -> `src_stronnicowany/1/core/header.h`
        - Jeśli wartość trzeciego argumentu to `false`, struktura podkatalogów powinna zostać spłaszczona. Oznacza to, że pliki będą kopiowane do katalogów docelowych bez zachowywania oryginalnych podkatalogów:
        - Przykład: `src/core/header.h` -> `src_stronnicowany/1/header.h`

## Pytania wprowadzające
1. Jakie kroki trzeba zrobić aby utworzyć własne polecenie w systemie Unix uruchamiane po prostu `JPG2jpg` (bez podania ścieżki i programu, który to wykonuje)?
