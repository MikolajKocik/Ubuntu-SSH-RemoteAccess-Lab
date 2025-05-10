# Ubuntu-SSH-RemoteAccess-Lab
Tematem labem jest połącznenie za pomocą protokołu SSH między 2-ma maszynami wirtualnymi w jednej z dystrybucji Linuxa - Ubuntu za pomocą użycia wirtualizacji typu 2-go: Host => Virtualbox.

Czym jest 'SSH'? 

Protokół sieciowy Secure Shell (SSH) wykorzystuje szyfrowanie, aby umożliwić dwóm połączonym urządzeniom — zwykle serwerowi i klientowi — bezpieczną komunikację ze sobą. Umożliwia użytkownikom bezpieczne dowodzenie i kontrolowanie odległych maszyn. Konwencjonalne metody przesyłania danych w postaci zwykłego tekstu, takie jak Telnet, FTP i logowanie, można bezpiecznie zastąpić protokołem SSH. Do jego powszechnych zastosowań należą przesyłanie plików, tunelowanie usług sieciowych i zdalna administracja.

![Wizualizacja SSH](images/Wizualizacja-ssh/ssh%20działanie.png)

## 1. Przygotowanie środowiska do pracy

Najpierw by móc rozpocząć pracę trzeba dodać maszyny wirtualne oraz wydzielić przestrzeń na dysku wraz z innymi parametrami jak: ilość RAM, CPU, obraz dysku (ISO).
Zalecam aby do tego typu ćwiczeń dysk miał +/- 25GB pojemności oraz RAM w zależności od posiadanych w komputerze. Co co CPU 1/4 zużycia powinna zajmować.

![instalacja](images/Przygotowanie%20środowiska/instalacja%20nowej%20maszyny%20i%20wybranie%20obrazu%20dysku.PNG)
![instalacja](images/Przygotowanie%20środowiska/wstępna-konfiguracja.PNG)

Kolejnym znaczącym krokiem jest dodanie kart sieciowych typu: Host-only do komunikacji lokalnie maszyn wraz siecią NAT jako odizolowane środowiska z dostępem do internetu.

![Sieć](images/Przygotowanie%20środowiska/przejście-do-ustawień.PNG)

Wybranie 1-wszej karty typu Host-only w konfiguracji przeznaczonej dla 'Experta' wraz z 2-gą kartą sieci NAT:

![Sieć](images/Przygotowanie%20środowiska/sieć%20NAT.PNG)

Jeśli już wszystko jest skonfigurowane, uruchamiamy naszą maszynę wirtualną i instalujemy Ubuntu: 

![Ubuntu](images/Przygotowanie%20środowiska/Po%20kliknięciu%20opcji,%20instaluj%20ubuntu.PNG)

Na początku proponuję dokonać update'u paczek, a dopiero potem zainstalować openssh-server:
```
  sudo apt update
```
![Update](images/Przygotowanie%20środowiska/sudo%20apt%20update.PNG)

```
  sudo apt install openssh-server
```
![ssh-server](images/Przygotowanie%20środowiska/openssh-server.PNG)

---

## 2. Konfiguracja i Użycie SSH

Po przygotowaniu maszyn wirtualnych i zainstalowaniu serwera OpenSSH na maszynie docelowej, możemy przystąpić do konfiguracji bezpiecznego dostępu zdalnego.

Pierwszym krokiem jest utworzenie par kluczy => publicznego i prywatnego.

Zasada korzystania z kluczy:
- Komp1 generuje parę kluczy
- Komp1 udostępnia klucz publiczny dla Komp2
- W Komp2 w pliku /home/user/.ssh/authorized_keys należy umieścić klucz publiczny od Komp1
- Teraz komp1 może zdalnie logować się do Komp2 (bez podawania hasła usera z Komp2)
  
Czyli najpierw generuję klucze, przesyłam do innych serwerów, w razie potrzeby dopisuję klucz publiczny do authorized_keys. 

Do wygenerowania par kluczy, używamy polecenia: 
```
  ssh-keygen -t rsa -b 4096
```
Posługujemy się tutaj algorytmem asymetrycznym RSA, który szyfruje połączenie.

![RSA](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/utworzenie%20klucza%20i%20nadpisanie%20jeśli%20już%20istnieje.PNG)

Później kluczowe jest skopiowane naszego utworzonego klucza publicznego do 2-giej maszyny wirtualnej, ponieważ to on będzie swego rodzajem odciskiem za pomocą, którego będziemy mogli się logować do 2 hosta bez podawania hasła. Do uzyskania tego efektu wpisujemy do konsoli ubuntu polecenie:

```
  sudo ssh-copy-id -i /home/mkocik/.ssh/id_rsa.pub vboxuser2@192.168.56.112
```
Gdzie vboxuser2 to nazwa hosta i po '@' jego adres IP

![Kopia_pub_rsa](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/kopia-klucza.png)

Następnie odczytujemy na 2 hoście, który będzie naszym zdalnym hostem, skopiowane klucze:

![Odczyt](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/odczyt.PNG)

Na zdjeciu jest pokazane również, że dany zdalny host może zawierać więcej kluczy również dla dostępu z innych maszyn. Z tego też powodu wygenerowałem 2 razy klucz aby zaprezentować możliwości połączeń z użyciem kryptografii nie tylko z jednego hosta.
Na końcu tylko pozostaje połączyć się z naszym zdalnym hostem oraz dokonać testu poprzez np. utworzenie folderu z daną treścią.

Polecenie:
```
  ssh 'vboxuser2@192.168.56.112'
```
![Połączenie](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/próba%20połączenia%20ze%20zdalnym%20hostem%20za%20pomocą%20klucza.PNG)
![Połączenie](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/połączenie%20udane.PNG)
![Połączenie](images/Zdalne%20połączenie%20przez%20SSH%20+%20generowanie%20par%20kluczy/utworzenie%20zdalnego%20folderu%20na%202%20maszynie.PNG)

Można zauważyć, że operujemy już na zdalnym hoście spoglądając na nazwę hosta.
