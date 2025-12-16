# UNIX ISI Lab 8: Użytkownicy i system plików

### Polecenia

| Grupa                      | Komenda             |
|----------------------------|---------------------|
| Prawa dostępu              | chmod, umask, chown |
| Informacje o użytkownikach | users, id, w, who   |
| Potoki                     | mkfifo              |
| Informacje o plikach       | stat, readlink      |
|                            |                     |

## Przydatne linki

1. <https://stackabuse.com/guide-to-understanding-chmod/>
2. <https://www.dobreprogramy.pl/linux-czym-jest-uprawnienie-suid,6628631747647617a> i <https://linuxexpert.pl/posts/2132/prawa-dostepu-do-plikow/>
3. <https://phoenixnap.com/kb/what-is-umask>

## Pytania wprowadzające

1. Jakie powinny być uprawnienia dla zasady?
2. Jak sprawdzić uprawnienia plików?
3. Jak interpretować uprawnienia?
4. Jak ustawiać uprawnienia tylko dla siebie, dla grupy, dla wszystkich? Jak działają numery?
5. Co dają uprawnienia `x` i `w` dla katalogów?
6. Jak sprawdzić właściciela pliku?
7. Jak zmienić właściciela?
8. Jak sprawdzić, oraz jak zmienić grupę pliku?

## Zadania

1. Uruchom polecenie (upewnij się, że wiesz co się stanie): `mkdir -p a/{b,c,d}`
   Dlaczego: `ls -ld a | cut -f 2 -d' '`
   zwraca na standardowym wyjściu 5? Tzn. dlaczego katalog 'a' ma pięć nazw?

2. Uruchom polecenie (upewnij się, że wiesz co się stanie):

   ```bash
   mkdir -p a/{b,c,d} ; ln -sf `pwd`/a a/b/link
   ```

   a następnie:

   ```bash
   cd a/b/link/b/link/b
   ```

   W jakim katalogu jesteś? Sprawdź działanie polecenia `pwd` oraz `pwd -P`.

3. Utwórz link (hard) do pliku `~/.bashrc` o nazwie `konfiguracja_bash`.
   Jaką wartość zwróci polecenie `ls -l ~/.bashrc` w 2-giej kolumnie? Co ona oznacza?

4. Sprawdź czy możesz utworzyć link (hard) do katalogu. Jaki jest status takiej operacji?

5. Utwórz w domowym katalogu link symboliczny do katalogu `/tmp` i nazwij go `smietnik`.
   - W jakim katalogu będziesz jeżeli przejdziesz do katalogu `smietnik`?
   - Jak dowiedzieć się na trzy sposoby dokąd prowadzi link o nazwie `smietnik`?

6. Udostępnienie wybranych plików dla innych użytkowników. Pracuj w parze.
   - Skonfiguruj swój domowy katalog tak, aby nikt oprócz Ciebie (i administratora ;)) nie mógł odczytać jego zawartości.
   - Utwórz w swoim domowym katalogu katalog 'pub-r', tak aby był dostępny do odczytu dla wszystkich (ale Twój domowy nie!).
   - Sprawdź, prosząc drugą osobę z pary, czy w/w prawa dostępu działają.

7. Udostępniania plików w grupach użytkowników. Pracuj w parze.
   - Sprawdź do jakiej wspólnej grupy użytkowników należycie z osobą z pary. (Jakiego użyjesz polecenia?).
   - Utwórz katalog `s` w swoim domowym katalogu.
   - Zmień grupowego właściciela na wspólną grupę.
   - Zmień prawa dostępu tak aby katalog `s` był dla Ciebie dostępny do odczytu, zapisu i dostępu, ale tylko do odczytu i dostępu dla grupy i nie był dostępny w ogóle dla pozostałych. Ćwiczenie wykonaj na 2 sposoby, korzystając ze składni symbolicznej i numerycznej polecenia `chmod`.
   - Utwórz plik `1` w katalogu `s`.
   - Sprawdź, prosząc drugą osobę z pary, czy może:
     - odczytać zawartość katalogu,
     - przejść do tego katalogu?
   - Jakie trzeba nadać prawa dostępu do pliku `1`, żeby inny użytkownik z wspólnej grupy mógł zobaczyć jego zawartość?
   - Ustaw bit `sgid` na katalogu `s` i powtórz powyższe ćwiczenia, tworząc nowy plik o nazwie `2`.
   - Co zmieniło się w prawach dostępu do pliku `2` w porównaniu z `1`?

8. Czy trzeba być właścicielem pliku, żeby móc zmienić jego nazwę? Podpowiedź: co z właścicielem katalogu? Sprawdź jakie operacje umożliwia ustawienie prawa `w` do katalogu, w którym znajduje się plik, a jakie ustawienie `w` do tego pliku.

9. Lista obecności.
   Stwórz posortowaną listę aktualnie zalogowanych użytkowników (tylko loginy). Upewnij się, że każda nazwa użytkownika występuje w niej tylko raz. Zapisz ją do pliku o nazwie `~/lab1/obecnosc/aktualny_czas_epoch` ( Unix timestamp epoch). Podpowiedź: sprawdź polecenie `cut`

10. Email dla ubogich.

- Utwórz nazwany potok (named pipe) o nazwie `~/in` - będzie służyć jako sposób przesyłania do Ciebie wiadomości.
- Upewnij się, że `in` ma prawo do zapisu dla wszystkich.
- Napisz skrypt, który w nieskończonej pętli odczytuje dane z `~/in` i dopisuje do pliku `~/inbox`.
- Jeżeli ktoś chce Ci wysłać wiadomość powinien ją zapisać do pliku `in` w Twoim katalogu domowym np. `echo 'Cześć tu Czesio.' > ~/in`.
- Wiadomości będą składowane w pliku `~/inbox`, możesz je tam odczytać, a jak nadasz odpowiednie prawa dostępu to nikt inny nie będzie ich widział.
