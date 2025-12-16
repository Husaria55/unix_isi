# UNIX ISI Lab 3

## Zakres

- Przetwarzanie danych ciąg dalszy.
- Wyszukiwanie plików.

### Polecenia

| Grupa               | Komenda             |
|---------------------|---------------------|
| Praca z tekstem     | cut, grep, tac, rev |
| Pliki               | file, find          |
| Wykonywanie poleceń | xargs               |
| Inne                | tee                 |

### Istotna poznana składnia

**2>** - przekierowywanie standardowego wyjścia **błędu** jednego polecenia do pliku. **Uwaga: jeśli plik już istnieje, zostanie nadpisany**. Można także dodać przekierowanie na końcu pliku, używając `2>>`.

## Pytania wprowadzające

1. Jakie deskryptory ma każdy program na Linuxie? (Jak nazywają się te deskryptory, które przekierowujemy np. za pomocą `|` lub `>`?)
2. Opisz przekierowania:
   - `>`, `>>`,
   - `2>`, `2>>`,
   - `<`
   - (opcjonalne na ten moment) `2>&1`, `>&2` (to samo co `1>&2`),
   - (opcjonalne na ten moment) `&>`, `&>>`
   - (opcjonalne na ten moment) `<<`, `<<<`

## Zadania

1. Opis laboratorium dany jest w pliku `~wojnicki/lab/Lab3.md`. 
   Napisz polecenie, które wyświetli wszystkie linie zawierające ciąg znaków **jpg**, niezależnie od wielkości liter.
2. Powtórz poprzednie ćwiczenie, ale tak, aby uzyskać również kontekst 2 linii poprzedzających każde wystąpienie szukanego ciągu znaków.
3. Powtórz poprzednie, wyświetlając również numery linii, w których występuje szukany ciąg znaków.
4. Policz, ile jest linii z literą lub literami ***k*** w opisie do laboratorium.
5. Policz, ile jest linii, w których nie ma w/w litery.
6. Policz, ile jest wystąpień litery ***k*** w opisie do laboratorium.  
   **Podpowiedź**:  użyj `tr`.
7. Napisz polecenie, które wyświetla w terminalu cały opis laboratorium w jednej linii.
   - Sprawdź za pomocą `wc`, czy rzeczywiście jest to jedna linia.
8. Użyj pojedynczego polecenia do skopiowania wszystkich plików, których nazwy kończą się na `.JPG` z katalogu `zasoby` (GitLab) do katalogu `~/lab3/jpg` w Twoim katalogu domowym. 
9. Usuń z katalogu `~/lab3/jpg` pliki, które nie są obrazkami JPEG (nazwa pliku nie jest istotna).   
   **Podpowiedź**: użyj `file` oraz `grep`.
10. Używając polecenia `find`, znajdź wszystkie katalogi w katalogu `/home/` i rekursywnie wyświetl znajdujące się w nich katalogi, ale tak, żeby nie pokazywać komunikatów o błędach.   
    **Podpowiedź**: przekieruj standardowe wyjście błędu np. do `/dev/null`.
11. Wyświetl informację z poprzedniego ćwiczenia na standardowym wyjściu i jednocześnie zapisz w pliku `~/lab3/home`.
12. Wyświetl wszystkie katalogi i podkatalogi (rekurencyjnie) z katalogu `/home`, których nazwy zawierają cyfrę "0" (zero), ale nie wyświetlaj błędów.
13. Powtórz poprzednie ćwiczenie, ale ogranicz poziom zagnieżdżenia katalogów do 3.
14. Znajdź w folderze `~/lab1` wszystkie utworzone pliki tekstowe oraz przekopiuj je do folderu `~/kopia_zapasowa`.   
    **Podpowiedź**: użyj `find -exec`.
15. Znajdź w katalogu `~wojnicki/lab` wszystkie pliki, których nazwy zawierają słowo kluczowe ***lab***, ignorując wielkość liter.

## Pytania podsumowujące

Wyjaśnij składnię:
```
find ~wojnicki -type f -name "*.txt" -exec cp {} ~/kopia_zapasowa/ \; 2>&1 | tee -a plik.txt
```

