# UNIX ISI Lab 10: wyrażenia regularne, sed, awk

## Polecenia

| Grupa                | Komenda        |
| -------------------- | -------------- |
| Przetwarzanie danych | grep, awk, sed |

### Pytania wprowadzające

1. Jak przy pomocy polecenia `grep` znaleźć dowolną literę?
2. Jak przy pomocy polecenia `sed` zamienić jeden tekst na inny w każdej linii?
3. Czy wyrażenia regularne polecenia `grep` mają zastosowanie do jednej linii czy wielu?
4. Co daje opcja '-E' w poleceniu `grep`?

### Użyteczne linki

- <https://en.wikipedia.org/wiki/Regular_expression>
- <https://regex101.com/>
- <https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/>
- <https://www.geeksforgeeks.org/awk-command-unixlinux-examples/>

## Zadania

**Uwaga:** w poniższych ćwiczeniach nie należy modyfikować zawartości plików wejściowych.

1. Tylko za pomocą polecenia `grep` i wyrażeń regularnych znajdź w pliku `/etc/passwd` wszystkie linie zawierające konta studentów (posiadające login w formacie XXXX, gdzie XXXX to czteroliterowy login). 
   Zadbaj o to, aby wypisane zostały tylko linie mające podany wzorzec w pierwszej kolumnie zawierającej login.
2. Tylko za pomocą polecenia `grep` i wyrażeń regularnych znajdź w pliku `/etc/passwd` wszystkie linie, na których końcu znajduje się litera **n**.
3. Tylko za pomocą poleceń `cut`, `grep` i wyrażeń regularnych znajdź w pliku `/etc/passwd` wszystkie linie zawierające użytkowników posiadających dwucyfrowe lub trzycyfrowe ID.
4. Za pomocą polecenia `sed` wyświetl zawartość pliku `/etc/passwd`.
5. Za pomocą polecenia `sed` zamień separator - dwukropek - w pliku `/etc/passwd` na spację.
6. Za pomocą polecenia `sed` wyświetl tylko loginy użytkowników zapisanych w pliku `/etc/passwd`.
7. Za pomocą polecenia `sed` wyświetl linie od 2 do 5 z pliku `/etc/passwd`.
8. Za pomocą polecenia `sed` wyświetl linie z pliku `/etc/passwd` opisujące osoby mające login zaczynający się na 'w' lub 'z'.
9. Przekształć polecenie z zadania 10, Laboratorium nr 2, tak aby dane wyjściowe były w formacie jak poniżej (zamieniona kolejność kolumn):   
    ***data,ile   
    20201010,2   
    20201011,3***
10. Dany jest plik dziennika z informacjami o żądaniach połączeń sieciowych w formacie:
```
Log start [01/Jan/2024:09:00:00 +0000]
192.168.1.10 - - [01/Jan/2024:10:00:00 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.10 - - [01/Jan/2024:11:00:00 +0000] ERROR
10.0.0.5 - - [01/Jan/2024:10:01:00 +0000] "POST /login HTTP/1.1" 404 567
192.168.1.10 - - [01/Jan/2024:10:02:00 +0000] "GET /style.css HTTP/1.1" 200 876
172.16.0.1 - - [01/Jan/2024:10:03:00 +0000] "GET /image.jpg HTTP/1.1" 304 0
192.168.1.10 - - [01/Jan/2024:10:04:00 +0000] "GET /index.html HTTP/1.1" 200 1500
Log end [01/Jan/2024:10:09:00 +0000]
```
    Napisz skrypt Awk, który:
    - przetwarza tylko wiersze, w których kod stanu HTTP (przedostatnia liczba w linii) to 200 lub 404,
    - śledzi adresy IP: dla każdego unikalnego adresu IP (cztery liczby oddzielone kropkami na początku każdej linii) obliczy:
      - całkowitą liczbę żądań wysłanych przez ten adres IP (pojedyncza liniia to jedno żądanie),
      - całkowitą liczbę bajtów wysłanych przez serwer w odpowiedzi na żądania z tego adresu IP (ostatnia liczba w linii),
    - podsumuje wyniki: po przetworzeniu wszystkich wierszy wydrukuje podsumowanie dla każdego unikalnego adresu IP w następującym formacie:
```
Adres IP: <Adres_IP>, Całkowita liczba żądań: <Liczba>, Całkowita liczba wysłanych bajtów: <Bajty>
```

    - pokaże numer wiersza pierwszego wystąpienia: dla każdego unikalnego adresu IP pokaże  numer wiersza (NR) pierwszego napotkania go w pliku dziennika; wydrukuj te informacje poniżej podsumowania adresu IP w formacie:
```
Pierwszy raz zauważony w wierszu: <Numer wiersza>
```

    Przykładowy wynik (na podstawie powyższego przykładowego dziennika):
```
Adres IP: 192.168.1.10, Całkowita liczba żądań: 3, Całkowita liczba wysłanych bajtów: 3600
Pierwszy raz zauważony w wierszu: 1
Adres IP: 10.0.0.5, Całkowita liczba żądań: 1, Całkowita liczba wysłanych bajtów: 567
Pierwszy raz zauważony w wierszu: 2

```

**Podpowiedź 0**: Użyj tablic asocjacyjnych.

**Podpowiedź 1**: Możesz użyć parametru NR (numer bieżącej linii) i NF (liczba pól w przetwarzanym rekordzie).

**Podpowiedź 2**: W `awk` działają pętle, np.:

```
for(x=1;x<NF;x++) {
}
```

**Podpowiedź 3**: W `awk` istnieje zestaw wbudowanych funkcji. Niektóre z nich mogą być przydatne, np. `match` oraz `substr`.

Więcej: <https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html>   
Porównanie wyrażeń regularnych: <http://www.greenend.org.uk/rjk/tech/regexp.html>

11. (*) Dany jest [plik](./zasoby/sample_awk_sed_grep_data.txt), który jest wyjściem z procesu rozponawania mowy.
    Napisz skrypt, który zwróci na standardowym wyjściu tekst będący efektem w/w rozpoznawania.
    Rozpoznany tekst znajduję się w liniach, które zaczynają się słowem kluczowym: `sentence1`
