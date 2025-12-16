# UNIX ISI Lab 9: Użytkownicy, system plików, zadania cykliczne

### Polecenia

| Grupa                       | Komenda                       |
|-----------------------------|-------------------------------|
| Kompresja                   | gzip, gunzip                  |
| Archiwa plików              | tar                           |
| Prawa dostępu               | umask                         |
| Ograniczenia                | ulimit                        |
| Terminale w terminalu       | screen                        |
| Zadania odroczone/cykliczne | at, atq, atrm, crontab, watch |
|                             |                               |

i... polecenia z poprzednich laboratoriów.

## Przydatne linki
1. [Understanding crontab](https://linuxhandbook.com/crontab/)

## Pytania wprowadzające
1. Co nam daje kompresja w czasach dużej pamięci?
2. Co to za rodzaj archiwum "tar"?
3. Czemu wprowadzać limity na procesy?
4. Co nam daje narzędzie `screen`? Kiedy go używać?
5. Co tak właściwie daje nam cron? Kiedy jego użycie jest potrzebne?

## Zadania

1. Sprawdź jak działa kompresja plików.
   - Utwórz nowy katalog. 
   - Skopiuj do niego plik `/etc/passwd`. 
   - Utwórz plik o nazwie `dwssap`, który zawiera wszystkie linie z pliku `passwd`, ale w odwrotnej kolejności. (Podpowiedź: `tac`).
   - Napisz polecenie, które skompresuje wszystkie pliki w bieżącym katalogu i zapisze je ze zmienionymi nazwami kończącymi się na `.gz`.
   - Porównaj wielkość skompresowanego plik `passwd` z oryginałem.
   - Sprawdź, czy w skompresowanych plikach są linie zawierające Twoją nazwę użytkownika. (podpowiedź: `zcat`).

2. Sprawdź jak działa tworzenie archiwów `tar`.
   - Umieść katalog z poprzedniego ćwiczenia, wraz z jego zawartością, w skompresowanym archiwum `tar`.
   - Sprawdź listę plików w archiwum.
   - Rozpakuj archiwum do nowego katalogu.

3. Sprawdź ile możesz uruchomić procesów? (podpowiedź: `ulimit`).
   - Sprawdź ile aktualnie procesów używasz (włączając w to wątki, zwróć uwagę na kolumnę LWP w rezultacie: `ps x -L`).
   - Ustaw limit "miękki" 2 proces powyżej tej wartości.
   - Sprawdź, co się stanie gdy przekroczysz ten limit:
     - Uruchom  `ls | wc -l`, a potem `ls | cat | wc -l`, a potem `ls | cat | cat | wc-l` itd. aż do uzyskania stosownego komunikatu.

4. Skonfiguruj bash, tak aby nowo tworzone pliki nie mogły być dostępne (w ogóle) dla
    nikogo poza Tobą. Zaprogramuj bash, aby w/w ustawienie było włączone przy
    ponownym zalogowaniu. (Podpowiedź: `umask`).

5. Używając `screen`, uruchom w 3 oknach: `vi`, `man screen`, `watch date`.
   - Przełączając się między oknami: napisz coś w `vi`, przeczytaj man, zobacz jak zmieniają się rezultaty `watch`.
     Uważaj na skróty klawiszowe (C-a jest prefiksem dla poleceń screen)!
   - Odłącz `screen`, tak aby okna (sesja) nie uległy zamknięciu.
   - Wyloguj się z serwera.
   - Zaloguj się ponownie.
   - Sprawdź czy masz aktywne sesje `screen`, podłącz się do aktywnej sesji.

6. Skonfiguruj `screen` w taki sposób, aby w ostatniej linii zawsze wyświetlał informacje o otwartych oknach.

7. Użyj dwóch terminali, albo `screen`.
    - W pierwszym uruchom polecenie, które co 1 sekundę będzie wyświetlać informację o tym, ile jest twoich procesów powstałych z uruchomienia programu bash.
    - W drugim terminalu zaloguj się do systemu (lub utwórz nowe okno w screen).   
    Czy liczba wyświetlana w pierwszym terminalu zwiększyła się?

8. Korzystając z mechanizmu `at` zaprogramuj polecenie, które wyświetli napis 'Już czas...' w jednym z terminali, których używasz do połączenia z serwerem. Odpowiedni plik terminala możesz znaleźć korzystając z polecenia:
```
ps x | grep bash | tr -s ' ' | cut -d ' ' -f 3
```
lub:
```
who | grep $USER | tr -s ' ' | cut -d' ' -f2
```

9. Zakładając, że masz katalog work (w swoim katalogu domowym), w którym przechowywane są bardzo ważne dane, zaprogramuj zlecenie okresowe (`cron`), które codziennie o godzinie takiej jaka będzie za pięć minut (spójrz na zegarek) wykona kopię zapasową tego katalogu (skopiuje całą zawartość katalogu work do innego katalogu np. work.kopia). Sprawdź po 5 minutach czy pliki skopiowały się poprawnie.

10. Usuń wpis `cron` dodany w poprzednim poleceniu.

11. Skonfiguruj `bash`, tak aby nowo tworzone pliki nie mogły być dostępne (w ogóle) dla nikogo poza Tobą. Zaprogramuj bash, aby w/w ustawienie było włączone przy ponownym zalogowaniu.

12. Napisz skrypt, który zawiadamia użytkownika, że inny użytkownik (lub użytkownicy zalogowali się do systemu). Nazwij go 'useralert' (user alert) i spraw aby skrypt był poleceniem wykonywalnym (odpowiednie `chmod`). Zawiadomienie powinno być wyświetlone na:
    - terminalu na którym było uruchomione polecenie,
    - zmodyfikuj kod tak aby było wyświetlone na wszystkich terminalach na których pracujesz.
    Przykładowe użycie:
    ```
    useralert wojnicki &
    ```
    Skrypt poinformuje użytkownika, który go uruchomił, kiedy użytkownik o nazwie 'wojnicki' zaloguje się do systemu.

Pseudokod:
```
endless loop{
    check for the user
    if the user has logged in {
        notification
        wait until the user logs out
    }
}
```
