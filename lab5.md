# UNIX ISI Lab 5: procesy

### Polecenia

| Grupa                     | Komenda                                                            |
|---------------------------|--------------------------------------------------------------------|
| Procesy, sygnały i pamięć | fg, bg, jobs, ps, top, htop, pidof, kill, killall, free -m, pstree |
| Priorytety                | nice, renice                                                       |
| Inne                      | uptime                                                             |

## Pytania wprowadzające:
Co tak właściwie robi `kill`?


## Zadania


1. Otwórz za pomocą `vim` plik muz4, następnie:
    - Przesuń kursor przed wyraz "utwory" w 1-szej linii.
    - Wstrzymaj aktualne zadanie (proces) za pomocą CTRL+Z.
    - Otwórz ponownie za pomocą `vim` plik muz4. Co się stało? Dlaczego?   
    Przeczytaj uważnie komunikat.
    - Sprawdź listę zadań poleceniem `jobs`. Powinien być widoczny jeden element, będący procesem wstrzymanym w pierwszym podpunkcie.
    - Przywróć proces zastopowany w pierwszym podpunkcie za pomocą polecenia `fg`.
    - Jesteś teraz w pierwotnie otwartym procesie (edytorze `vim`). Możesz już go zamknąć.
    - Sprawdź listę zadań poleceniem `jobs`. Lista powinna być pusta ze względu na zamknięcie edytora, który wcześniej był zepchnięty w tło.
2. System operacyjny nie przydziela zadaniom zastopowanym czasu procesora, a tym samym nie wykonują one swojej pracy. Istnieją jednak okoliczności, w których potrzebujemy, aby procesy zepchnięte w tło wciąż pracowały. Przeprowadź eksperyment:
    - **UWAGA: Wykonując poniższe kroki na wspólnym serwerze pamiętaj o innych, którym możesz uniemożliwić pracę - odpalaj poniższy program krótko.**
    - Pobierz do swojego katalogu domowego źródło programu "calc.c" za pomocą polecenia:
      `wget https://gitlab.kis.agh.edu.pl/miroman/unix_isi/-/raw/main/Lab/zasoby/calc.c`
    - Skompiluj program poleceniem:
      `gcc -o prog calc.c`
    - Uruchom program poleceniem `./prog`
    - Program nie robi nic pożytecznego – wykonuje w nieskończoność obliczenia matematyczne, ale nie wyświetla ich wyników na ekran, wobec tego może się wydawać, że nic się nie dzieje.
    - Zaloguj się do serwera w drugim oknie terminala (lub od nowa w `putty`) i sprawdź użycie zasobów systemu przez swoje procesy poleceniem:
    ```
    top -b -n 1 -u `whoami`
    ```
    Znajdź na liście program `prog` i zauważ, że ma bardzo duże zużycie procesora w kolumnie **%CPU**.
    - Wróć do pierwszego terminala, w którym działa prog i wciśnij CTRL+Z (przesuwając go w tło i jednocześnie stopując).
    - Wróć do drugiego terminala i ponownie wykonaj polecenie `top` z tym samym zestawem argumentów. Zauważ, że zużycie procesora spadło do zera. Jest to ewidentny dowód, że program faktycznie jest zastopowany i nie korzysta z zasobów systemowych.
    - Wróć do pierwszego okna terminala i wykonaj polecenie `bg`. Dzięki niemu proces programu `prog` został wznowiony (pomimo, że jest nadal w tle). Możesz ponownie wykonać top, aby zobaczyć, że program znowu zużywa zasoby systemowe. Mimo tego program nie okupuje pierwszego terminala i masz dostęp do wydawania     poleceń.
    - Przywróć program do pierwszego planu za pomocą polecenia `fg` i zamknij go przez CTRL+C.
3. Możesz poćwiczyć kontrolowanie zadań uruchamiając wiele instancji różnych programów, wkładając je w tło, a następnie wznawiając działanie w tle poszczególnych z nich. 
    - Poeksperymentuj i sprawdź, które zadania będą używane przez `bg` i `fg` jeśli nie przekażesz do nich żadnych argumentów. Jak kontrolować poszczególne z zadań? 
    - Znajdź w internecie lub w manie co oznaczają plusy i minusy przy niektórych zadaniach w poleceniu `jobs`.
4. Znajdź proces, który zużywa najwięcej pamięci.  
   Jaka jest różnica pomiędzy RSS (Resident Set Size) a VSZ (Virtual Memory Size)?
5. Jaki proces zużywa najwięcej CPU?
6. Ile jest wszystkich procesów w systemie?
7. Ile jest moich procesów w systemie? Z jakich programów powstały?
8. Ile jest procesów bez terminala (kolumna TTY w ps)?
9. Program `zasoby/datecnt` co sekundę generuje na standardowym wyjściu aktualny czas i datę.
   - Program można pobrać komendą: `wget https://gitlab.kis.agh.edu.pl/miroman/unix_isi/-/raw/main/Lab/zasoby/datecnt`, jednakże aby móc go wykonać należy jeszcze nadać uprawnienia do wykonywania: `chmod +x datecnt`
   - Uruchom w/w program w tle przekierowując standardowe wyjście do pliku.
   - Sprawdź, czy nowe dane pojawiają się w pliku.
   - Przywróć zadanie do pierwszego planu (`fg`).
   - Wstrzymaj jego działanie, czy dane pojawiają się w pliku?
   - Włącz działanie zadania w tle (`bg`), czy dane pojawiają się w pliku?
   - Zaloguj się w drugim terminalu, za pomocą polecenia `kill` wypróbuj działanie sygnałów SIGSTOP i SIGCONT na w/w zadaniu (odnajdź PID tego zadania).
   - Powtórz ćwiczenie bez przekierowania wyjścia do pliku.
10. Uruchom komendę w tle i poczekaj na jej zakończenie: `(sleep 10 && echo "Ble") &`
  - Ponów powyższy krok, ale zamknij terminal, z którego uruchomiłeś komendę, sprawdź w procesach czy komenda jest nadal uruchomiona czy nie?

## Pytania podsumowujące:
1. Co robi CTRL + Z w terminalu? Co robi CTRL + C w terminalu?
2. Co się stanie jak zamkniemy terminal, w którym jest uruchomiona komenda?
