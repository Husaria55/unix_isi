# UNIX ISI Lab 7: BASH

## Zakres

- Wymiana kluczy SSH.
- Wiele procesów w BASH.
- Sygnały w BASH.

## Polecenia

| Grupa                 | Komenda    |
| --------------------- | ---------- |
| Procesy, sygnały      | trap, wait |
| Obliczenia            | bc         |
| Nazwy plików          | basename   |
| Konwersja obrazków    | convert    |
| Informacje o systemie | uptime     |


## Przydatne linki

1. <https://www.ssh.com/academy/ssh/keygen>
2. (dodatkowe, FFmpeg) <https://www.ffmpeg.org/ffmpeg.html>


## Istotna poznana składnia
`polecenie &` - uruchomienie polecenia w tle (tym sposobem można wiele poleceń wywołać współbieżnie)


## Zadania

1. Wygeneruj parę kluczy publiczny-prywatny dla komputera, z którego łączysz się z serwerem. 
   - **Uwaga:** sugeruję nie zmieniać domyślnej ścieżki i nazwy dla kluczy, oraz nie ustawiać hasła na klucz.
   - Uwaga: jeśli już używasz klucza SSH to upewnij się, że sobie nie nadpiszesz plików tracąc bezpowrotnie klucze i może nawet jedyny dostęp do pewnych urządzeń

2. Wymień klucze, tak abyś mógł logować się na serwer bez podawania hasła dostępowego.
   Zrób to korzystając z polecenia `ssh` (nie `scp`).

3. Zaprogramuj skrypt BASH, o nazwie `k` (jak kalkulator), który obliczy wyrażenie arytmetyczne będące jego argumentem, z uwzględnieniem wartości rzeczywistych np. `kalkulator 2.2*9` powinno zwrócić na standardowym wyjściu `19.8`. Podpowiedź: użyj `bc`.

4. Napisz skrypt, który:
    - co 3 sekundy będzie dodawać do pliku ~/myps.log pojedynczą linię zawierającą informacje o aktualnej dacie i czasie, ilości uruchomionych procesów oraz obciążeniu systemu, w formacie:   
    YYYY-MM-DD HH:MM:SS,n,l1,l5,l15   
    gdzie n to ilość procesów, l1, l5, l15 to obciążenie systemu (load) odpowiednio w ciągu ostatnich 1, 5, 15 minut,
    - zaprogramuj aby w/w nazwa pliku (~/myps.log) była przechowywana w zmiennej, aby w razie potrzeby można było ją łatwo zmienić,
    - spraw, aby skrypt był odporny na SIGTERM oraz C-c (ctrl c),
    - spraw, aby skrypt po otrzymaniu SIGUSR1 kasował aktualną zawartość pliku ~/myps.log i wyświetlał na wyjściu błędów: "File ~/myps.log cleaned".

5. Pobierz z internetu trzy dowolne obrazki JPG.
   Umieść je na serwerze (np. za pomocą polecenia `scp`).
   Napisz skypt, który współbieżnie konwertuje w/w obrazki do formatu PNG w katalogu wskazanym jako jego argument.
     - Napisz funkcje, która będzie dokonywać konwersji i uruchom ją współbieżnie na plikach ze wskazanego katalogu.
     - Do konwersji użyj polecenia `convert`.
     - Sprawdź czy konwertowany plik jest obrazkiem JPG.
     - Jeżeli jakiś plik nie jest obrazkiem JPG wyświetl stosowny komunikat na standardowym wyjściu błędów.
     - Nazwa pliku po konwersji ma składać się z oryginalnej nazwy pliku i kończyć się na `.png`.
     - Jeżeli nazwa docelowa istnieje nie dokonuj konwersji, wyświetl stosowny komunikat na standardowym wyjściu błędów.
     - Skrypt ma zakończyć się, wraz z zakończeniem wszystkich procesów konwersji dla każdego z plików.
       Podpowiedź: użyj `wait`.
     - Jeżeli skrypt urucomiony zostanie z opcją `-v` (verbose), to wyświetl nazwę każdego konwertowanego obrazka na standardowym wyjściu oraz policz ile sekund trwała konwersja wszystkich plików, z dokładnością do 1/10 sekundy, i wyświetl ten czas na standardowym wyjściu.
     - Zaprogramuj skrypt tak, żeby opcja `-v` mogła być użyta jako pierwszy lub drugi parametr. 
       Podpowiedź: użyj `shift`.

6. Prześledź wykonanie programu z poprzedniego ćwiczenia korzystając z `set -v` oraz `set -x`.

7. Napisz skrypt, który:
   - Tworzy dwa pliki tymczasowe w systemowym katalogu tymczasowym za pomocą polecenia `mktemp`.
     - Pierwszy plik służy jako dziennik zdarzeń (zapisz jego ścieżkę pod zmienną o nazwie `TEMP_LOG`).
     - Drugi plik przechowuje listę wszystkich nowych plików (zapisz jego ścieżkę pod zmienną o nazwie `TEMP_FILES_LIST`).
   - Argumentem skryptu jest ścieżka do katalogu, który ma być monitorowany (np. ~/Documents).
   - Skrypt co 5 sekund sprawdza, czy w podanym katalogu pojawiły się nowe pliki lub katalogi:
     - Jeśli pojawiły się nowe pliki, zapisuje ich nazwy do pliku `TEMP_FILES_LIST` oraz rejestruje informację w `TEMP_LOG` w formacie:
       "YYYY-MM-DD HH:MM:SS: Dodano plik: <filename>"
   - Obsługuje sygnał SIGUSR1, po którego otrzymaniu wyświetla zawartość pliku TEMP_FILES_LIST.
   - Obsługuje sygnał SIGUSR2, po którego otrzymaniu wyświetla zawartość pliku TEMP_LOG.
   - sygnał SIGTERM oraz C-c (ctrl c) powinien usunąć oba pliki oraz zatrzymać działanie skryptu.


## Pytania podsumowujące

1. Jak sprawdzić z poziomu skryptu jakie nowe pliki powstały w katalogu od ostatniego sprawdzenia?
2. Czym się w praktyce różnią `set -v` i `set -x`, co zmieniają?
