# UNIX ISI Lab 2

## Zakres

- Podstawowe przetwarzanie danych.

### Polecenia

| Grupa           | Komenda                        |
|-----------------|--------------------------------|
| Praca z tekstem | cat, cut, tr, head, tail, sort |
| Inne            | cp, echo, date, which, ls      |

### Istotna poznana składnia

**>** - przekierowywanie standardowego wyjścia jednego polecania do standardowego wejścia innego polecania. **Uwaga: w razie istnienia pliku nadpisze jego zawartość!**

**>>** - przekierowywanie standardowego wyjścia jednego polecania do standardowego wejścia innego polecania. **Uwaga: w razie istnienia pliku dopisze do niego na końcu**

`$(komenda)` - substytucją poleceń (command substitution). Umożliwia ona uruchomienie komendy/komend wewnątrz innej komendy, a wynik jej działania (wyjście komendy) zostaje podstawiony w miejsce `$(komenda)`

**\* (wildcard)** - symbol wieloznaczny, który reprezentuje dowolny ciąg znaków (w tym także pusty ciąg). Jest często używany w operacjach na plikach, aby odwołać się do więcej niż jednego pliku lub do wzorców nazw plików

**(polecenie1 && polecenie2 | polecenie3 ...)** - grupowanie poleceń. Polecenia ujęte w nawiasach są traktowane jako jedna całość.

## Pytania wprowadzające
1. Jak sprawdzamy czym jest dana komenda?
   1. \* A jeśli to nie jest plik?
2. Jakie są dostępne przekierowania w bashu? Opisz każde z nich.
3. Czy po opcji jednoliterowej wstawiamy biały znak (np. spacje) przed podaniem argumentu? Czyli np. `komenda -d':'` czy `komenda -d ':'`?

## Zadania

1. Jaka jest ścieżka dostępu do polecenia `date` (gdzie w systemie plików znajduje się plik programu `date`)? Używając złożonego polecenia wyświetl wszystkie informacje o tym pliku. **Podpowiedź:** połącz odpowiednio `ls` i `which`.

2. Wyświetl 2 i 3 linię z pliku `/etc/passwd` za pomocą poleceń `head` oraz `tail`. **Podpowiedź:** obetnij od góry, a potem od dołu.

3. Przy pomocy polecenia `seq` zapisz trzykrotnie liczby od 1 do 10 (włącznie) w pliku `~/lab2/numbers.txt`, każda w nowej linii. Polecenie można wykonać trzykrotnie, za każdym razem dopisując sekwencję liczb do pliku.

4. Przy pomocy polecenia `uniq` wyświetl tylko unikalne liczby z pliku `~/lab2/numbers.txt` (usuń powtórzenia).

5. Napisz polecenie, które znajdzie największą liczbę w pliku `~/lab2/numbers.txt`. **Podpowiedź:** użyj `sort` i `head` albo `tail`.

6. Napisz polecenie (sekwencje poleceń), które umieści wpis w dzienniku. Wpis w dzienniku składa się z godziny oraz tekstu i powinien być umieszczony w katalogu `~/lab2/dziennik/`, w pliku o nazwie RRRRMMDD, gdzie RRRR to aktualny rok, MM to miesiąc, DD dzień. Polecenie musi działać niezależnie od dnia, w którym będzie wykonywane i zawsze tworzyć plik z aktualną datą. Więcej wpisów w ten sam dzień powoduje dodanie kolejnych linii do w/w pliku.   
Na przykład polecenie uruchomione w dniu 04.10.2020 o godzinie 10:34 powinno utworzyć plik w katalogu `~/lab2/dziennik` o nazwie:    
    `20201004`   
którego zawartość to:   
    `10:34 To jest mój wpis`   
Kolejne wpisy tego samego dnia powinny być w kolejnych liniach w tym pliku.
    - **Podpowiedź:** można wynik jednej komendy wykorzystać w nazwie pliku w następujący sposób: `komenda1 > $(komenda2).txt`

7. Umieść w dzisiejszym dzienniku kilka wpisów, a następnie skopiuj dzisiejsze wpisy do pliku reprezentującego "jutro" (tomorrow). Powinno to działać niezależnie od dnia, w którym wykonywane jest polecenie.

8. Wyświetl zawartość dziennika dla wszystkich wpisów z bieżącego miesiąca. **Podpowiedź:** wykorzystaj polecenie `date` w celu określenia aktualnego miesiąca.

9. Do tej pory używaliśmy polecenia `cat` do wyświetlania zawartości plików. Za jego pomocą można również zapisywać tekst do pliku.
   Wykonaj polecenie:   
   ***cat > ~/lab2/cat_test.txt***   
   Wpisz za pomocą klawiatury kilka linijek tekstu i wciśnij CTRL+D (dla ciekawskich: jest to znak EOT - End of Transmission).   
   Sprawdź czy w pliku  ***~/lab2/cat_test.txt*** znajdują się wpisane przez Ciebie dane (należy uprzednio utworzyć katalog lab2 w odpowiedniej lokalizacji).

10. Sprawdź czy w ***~/lab2/dziennik*** posiadasz co najmniej 2 pliki dziennika (patrz: zadanie 6.) dla dowolnych dat. Jeśli nie, utwórz 2 przykładowe dzienniki (np. 20241010 oraz 20241011) wpisując do nich dowolne dane. Następnie napisz polecenie, które wyświetli raport z wpisów we wszystkich dziennikach z katalogu ***~/lab2/dziennik***. Raport powinien dostarczać informacji ile jest wpisów (linii w pliku) każdego dnia w formacie CSV (każda kolumna danych oddzielona przecinkiem).
    Pojedyncza linia powinna wyglądać tak:   
    ***n,RRRRMMDD***   
    gdzie RRRR to rok, MM miesiąc, DD dzień, n - ilość wpisów/linii.   
    Przykład:   
    ***2,20241010
    3,20241011***

    **Podpowiedź:** zbuduj potok z użyciem: `wc`, `tr`, `cut`.
    
11. Rozbuduj polecenie raportujące, tak aby dodać nagłówek CSV "ile,data" identyfikujący kolumny w 1-szej linii i zapisz do pliku ***~/lab2/raport1***.   
    Przykład:   
    ***ile,data   
    2,20241010
    3,20241011***
    
    
12. Napisz polecenie, które pokaże zawartość pliku ***~/lab2/raport1***, ale bez nagłówka (1-szej linii).

13. Napisz polecenie, które pokaże tylko ilości wpisów z pliku ***~/lab2/raport1*** (tylko wartości w pierwszej kolumnie). Użyj w tym celu polecenia `cut`.

14. Napisz polecenie, które znajdzie datę dla największej ilości wpisów w pliku ***~/lab2/raport1***. 



## Pytania podsumowujące
1. Jakie liczby wyświetla `seq 1 10` (czy wyświetla też 1 i 10)?
2. Jak usuwamy powtórzenia z pliku?


### Przydatne linki:
- https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file
