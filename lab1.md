# UNIX ISI Lab 1

## Zakres

- Podstawowe interakcje z systemem.
- Uruchamianie poleceń.
- Łączenie poleceń.
- System plików.

### Polecenia

| Grupa                      | Komenda                               |
|----------------------------|---------------------------------------|
| Obsługa haseł              | passwd                                |
| Pomoc                      | man, apropos, whatis                  |
| Obsługa plików i katalogów | pwd, cd, ls, mkdir, rmdir, rm, cp, mv |
| Praca z tekstem            | more, less, wc, echo                  |
| Informacje o czasie        | date                                  |

### Istotna poznana składnia

**|** (pipe) - przesłanie standardowego wyjścia komendy na standardowe wejście innej komendy

**CTRL + D** - wysłanie znaku końca tekstu, wyjście z shella, wyjście z narzędzi oczekujących na tekst

**CTRL + C** - wysłanie sygnału SIGINT do programu (czyli najczęściej w praktyce ubicie go)


## Zadania


1. Wejdź do powłoki unixowej. Możesz to zrobić na własnym komputerze, chociaż zalecane jest wejście na [przygotowany serwer](lab.kis.agh.edu.pl) (wymagana sieć AGH). Zaloguj się korzystając z `ssh` (Linux/MacOS) lub programu [Putty](https://putty.org/) (Windows).
   1. **Uwaga:** Konta zakładamy na [stronie Panel KIS](panel.kis.agh.edu.pl:3000)

2. Używając polecenia `apropos`, znajdź polecenie służące do zmiany hasła, korzystając ze słowa kluczowego `authentication`. Podpowiedź: polecenie zwróci wiele wyników, więc stronicuj je za pomocą `less` lub `more`, a następnie spróbuj wyszukać odpowiednie polecenie, używając słowa `update`.

3. Przeglądnij materiały z wykładu dostępne w katalogu `/home/wojnicki/wyk/` (katalog znajduje się na serwerze lab.kis.agh.edu.pl) (notatki znajdują sie w folderze: `wyk`, sesja jest w: `scripts`).

4. Odczytaj sesje poleceń z wykładu. Sesja znajduje się w pliku: `/home/wojnicki/wyk/script/w1`. Jakim poleceniem to zrobisz? Czy wszystkie znaki wyświetlane są poprawnie?

5. Używając poleceń `whatis` oraz `man` odnajdź w dokumentacji co oznacza trzecia kolumna w pliku `/etc/passwd`. Podpowiedź: `whatis passwd`, otwórz odpwowiednią sekcję manuala. 

6. Co robi opcja `-a` polecenia `man`?

7. Jaka jest ścieżka dostępu do Twojego katalogu domowego? Jakim poleceniem sprawdzić pełną ścieżkę do katalogu, w którym jesteś?

8. Znajdź w dokumentacji polecenia `bash` co oznacza tylda (czyli `~`).
   - Podpowiedź: znajdź rozdział `Tilde Expansion`. 
   1. Co oznacza `~wojnicki`?

9. Jaka jest dzisiejsza data?

10. Która jest godzina (tylko godzina, bez daty, sprawdź w manualu odpowiednie opcje polecenia do wyświetlania daty i czasu)?

11. Za pomocą polecenia `wc` sprawdź ile jest katalogów domowych użytkowników na serwerze. Podpowiedź: katalogi użytkowników znajdują się w katalogu `/home`.

12. Za pomocą polecenia `mkdir` utwórz w swoim katalogu domowym, katalog o nazwie `lab1`, a w nim katalog `dziennik`. Polecenie musi działać niezależnie od tego, w jakim się jest katalogu oraz "rekursywnie" - tj. stworzyć oba katalogi naraz, jeden w drugim. Podpowiedź: przeczytaj dokumentację do polecenia `mkdir`.

13. W swoim domowym katalogu utwórz katalog o nazwie `tmp`.

14. Skopiuj plik `/etc/passwd` do `~/tmp`.

15. Skopiuj plik `/etc/passwd` do `~/tmp` pod nazwą `passwd.system.orig`.

16. Zmień nazwę pliku `passwd.system.orig` na `passwd.orig`.

17. Skopiuj plik `/etc/passwd-` do `~/tmp` pod nazwą `passwd`. Co się stało z poprzednią zawartością pliku o tej nazwie. Podpowiedź: żeby to sprawdzić porównaj wielkość ob plików np.: `ls -l /etc/passwd*`.

18. Wykonaj poprzednie ćwiczenie ale z opcją `-i`. Jaka jest różnica. Sprawdź w dokumentacji co robi ta opcja.

19. Skopiuj rekursywnie (wraz z plikami i podkatalogami) całą zawartość katalogu `~/lab1` do `~/tmp`.

20. Utwórz katalog `~/tmp/to_remove`. Przenieś wszystkie pliki z katalogu `~/tmp` do `~/tmp/to_remove`.

21. Wyświetl wszystkie pliki w swoim domowym katalogu, niezależnie od tego w jakim roboczym katalogu się znajdujesz.

22. Wyświetl wszystkie pliki we wszystkich katalogach w swoim domowym katalogu. Podpowiedź: zobacz dokumentację polecenia `ls`.

23. Zmień nazwę katalogu `~/tmp` na `~/trash`

24. Usuń katalog `~/trash` wraz z wszystkimi plikami i podkatalogami.

25. Spróbuj za pomocą polecenia `rm` usunąć katalog `/tmp`. Co wyświetliło się na terminalu i dlaczego? 

26. Zakończ pracę z powłoką. Uwaga: możesz to zrobić przynajmniej na 3 sposoby: 1 skrót klawiszowy, 2 polecenia.


## Pytania podsumowujące
1. Czym różni się `apropos` od `man`?
2. Jakie narzędzie najlepiej użyć do przeglądania plików tekstowych, aby zachować odpowiednie formatowanie?
3. Dlaczego plik `/etc/passwd` jest tak istotny w systemie Linux?
4. Co może się stać z istniejącymi plikami, gdy skopiujemy inny plik o tej samej nazwie?
5. Co tak właściwie robi `|` (pipe)?
