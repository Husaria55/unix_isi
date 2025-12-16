# UNIX ISI Lab 4

## Zakres

- VI(M)

### Polecenia

| Grupa           | Komenda |
| --------------- | ------- |
| Praca z tekstem | `vim`   |

### [VIM Cheat Sheet](https://vim.rtorr.com/)

Lista najczęściej używanych komend z skrótów klawiszowych dostępnych w edytorze VIM - szerszy zakres komend i bardziej zwięzły format niż w przypadku `vimtutor`. Przydatne dla osób znających zastosowanie danych komend, ale nie pamiętających jeszcze ich składni.

## Pytania wprowadzające:
Czym jest VIM, a czym VI?

## Zadania

1. Wykonaj wszystkie zadania dostępne w tutorialu VIM z rodziałów od 1 do 6 włącznie. Tutorial VIM-a jest dostępny przy użyciu komendy ​`vimtutor​`. Rozdział 7, w szczególności konfigurację z użyciem `.vimrc` pozostawiamy dla chętnych.
2. Utwórz plik o nazwie "muz" z tytułami 4 ulubionych utworów muzycznych, użyj `vim` lub `cat`.
3. Za pomocą `vim` otwórz muz, dodaj w 1-szej linii opis: "ulubione utwory". Zapisz plik pod tą samą nazwą.
4. Za pomocą `vim` otwórz muz, tak aby kursor znalazł się w 3 linii, następnie:
    - Włącz numerację linii.
    - Wpisz w linii poniżej (użyj pojedynczego polecenia do przejścia do nowej linii i włączenia trybu wstawiania) jeszcze jeden tytuł.
    - Przed każdą linią z tytułem dodaj linię z nazwą zespołu wykonującego dany utwór. Zrób tak, aby przynajmniej dwa utwory były wykonywane przez ten sam zespół.
    - Wetnij linie z utworami jeden tabulator w prawo.
    - Zapisz zawartość w pliku muz1.
    - Jak się dowiedzieć, nie opuszczając edytora, ile linii ma plik?
5. Otwórz muz1, a następnie:
    - Przesuń kursor przed wyraz "utwory" w 1-szej linii.
    - Usuń wszystkie litery do końca linii.
    - Zmień litery w słowie po lewej stronie kursora na przeciwne (małe na duże, duże na małe; znajdź polecenie, które to robi).
    - Przejdź do ostatniej linii, usuń ją w całości.
    - Zapisz zawartość w pliku muz2.
6. Otwórz plik muz1 i zmień tabulatory na przecinki. Połącz linie, tak aby pojedyncza linia reprezentowała zespół oraz wszystkie utwory. Zapisz zawartość w pliku muz3.
7. Otwórz muz3 i zmień jego zawartość, używając mechanizmu kopiowania, tak aby pojedyncza linia reprezentowała utwór tj.: nazwa wykonawcy, nazwa utworu. Zapisz jako muz4.
8. Sprawdź co mówi polecenie `file` o pliku muz4.
9. Otwórz plik muz4 i posortuj rosnąco linie począwszy od linii nr 2.
10. Otwórz plik muz i wyszukaj napis "utwory" przechodząc do niego, zastąp wyraz "utwory" przez "kawałki", wyjdź z edytora nie zapisując pliku. Postaraj się to zrobić jak najszybciej, możesz sobie mierzyć czas w ten sposób:

    ```bash
    s=`date +'%s'`; vim +/utwory muz; expr `date +'%s'` - $s
    ```

## Dodatkowe

### [VIMRC](https://www.freecodecamp.org/news/vimrc-configuration-guide-customize-your-vim-editor/) - Dla zaawansowanych

Edytor VIM może być konfigurowany z użyciem pliku `.vimrc`, umożliwia on domyślną aktywację opcji edytora, ustawianie własnych skrótów klawiszowych, włączenie kolorowania składni, korzystanie z szeregu open-sourcowych pluginów, implementację własnych skryptów, etc. Wszystko to czyni edytor VIM potężnym narzędziem, wykorzystywanym przez wielu programistów jako główny edytor tekstowy.

### [VIM Genius](http://www.vimgenius.com/) - dla chcących poćwiczyć 

## Pytania podsumowujące VIM
1. Jak wyjść z `vim`a?
2. Jak dopisać coś w istniejącej linijce?
3. Jak zrobić kopiuj & wklej
4. Jak zamienić więcej niż jedno wystąpienie tekstu?
5. Jak cofnąć i ponowić?
6. Jak przechodzić po wynikach wyszukania?
