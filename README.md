# OverTheWire: Bandit Wargame Çözümleri & Linux CLI Notları
 
Bu depo, **OverTheWire Bandit** wargame'i için tuttuğum çözüm notlarını, terminal komutlarını ve metin işleme tekniklerini içerir. Linux komut satırı temelleri, shell gezinme, metin manipülasyonu, arşiv/sıkıştırma yönetimi, ağ (networking), SSH, cron job'lar ve Git konularında kişisel bir referans rehberi olarak hazırlandı.
 
---
 
## 🛠 Kullanılan Araçlar
 
* **Dosya İşlemleri:** `cat`, `ls`, `cd`, `find`, `file`, `chmod`
* **Metin İşleme:** `grep`, `sort`, `uniq`, `strings`, `base64`, `tr`, `diff`
* **Arşiv & Sıkıştırma:** `xxd`, `gzip`, `bzip2`, `tar`
* **Ağ & SSH:** `ssh`, `scp`, `nc` (netcat), `openssl s_client`
* **Versiyon Kontrolü:** `git` (clone, log, branch, tag, push)
* **Diğer:** cron job analizi, setuid binary'ler, restricted shell escape teknikleri
---
 
## 📚 İçindekiler
 
| Seviye | Konu |
|---|---|
| 0 → 1 | SSH ile bağlanma, `cat` |
| 1 → 2 | Tire (`-`) ile başlayan dosya adları |
| 2 → 3 | Boşluk içeren dosya adları |
| 3 → 4 | Gizli dosyalar |
| 4 → 5 | `file` ile dosya türü tespiti |
| 5 → 6 | `find` ile boyut/izin filtreleme |
| 6 → 7 | `find` ile sahiplik filtreleme, hata akışını yönlendirme |
| 7 → 8 | `grep` ile kelime arama |
| 8 → 9 | `sort` + `uniq -u` ile tekil satır bulma |
| 9 → 10 | `strings` ile okunabilir veri çıkarma |
| 10 → 11 | Base64 çözme |
| 11 → 12 | ROT13 deşifreleme |
| 12 → 13 | Hexdump ters çevirme + çok katmanlı arşiv çözme |
| 13 → 14 | SSH anahtarı ile kimlik doğrulama, `scp` |
| 14 → 15 | `nc` ile port üzerinden şifre gönderme |
| 15 → 16 | `openssl s_client` ile SSL/TLS portuna bağlanma |
| 16 → 17 | Rastgele portta SSH private key yakalama |
| 17 → 18 | `diff` ile iki dosya arasındaki farkı bulma |
| 18 → 19 | Kısıtlı `.bashrc` ortamında tek komutla SSH |
| 19 → 20 | setuid binary ile başka kullanıcı olarak komut çalıştırma |
| 20 → 21 | `nc` dinleyici (listener) kurup istemci-sunucu iletişimi |
| 21 → 22 | Cron job ile dünyaya-açık geçici dosya sızıntısı |
| 22 → 23 | Cron job'da `md5sum` ile hesaplanan dosya adını takip etme |
| 23 → 24 | Cron job'un çalıştırdığı klasöre kendi script'ini bırakma |
| 24 → 25 | `nc` ile brute-force pin kodu deneme |
| 25 → 26 | Kısıtlı shell'den `more`/`vim` üzerinden kaçış |
| 26 → 27 | setuid binary ile devam |
| 27 → 28 | `git clone` ile repo indirme |
| 28 → 29 | `git log -p` ile commit geçmişinde silinmiş veriyi bulma |
| 29 → 30 | `git branch -a` ve `git checkout` ile gizli branch |
| 30 → 31 | `git tag` ve `git show` ile gizli etiket |
| 31 → 32 | `git add -f` ile `.gitignore`'u aşıp push etme |
| 32 → 33 | Büyük harf shell'den `>>` ile kaçış |
| 33 | Oyunun son seviyesi |
 
---
 
## 🚀 Seviye Çözümleri
 
### Level 0 ➔ Level 1
 
* **Konu:** Temel SSH bağlantısı kurma ve standart metin dosyalarını okuma.
* **Ne öğrenildi:** Sunucuya `ssh -p <port> kullanıcı@host` ile bağlanılır, ev dizinindeki dosyalar `ls` ile listelenir ve `cat` ile içerikleri okunur.
```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
ls
cat readme
```
 
### Level 1 ➔ Level 2
 
* **Konu:** Adı `-` (tire) olan dosyaları okuma.
* **Ne öğrenildi:** `cat -` gibi bir komut, `-` karakterini stdin'den okuma isteği olarak yorumlar. Bunu önlemek için dosya yolunu `./` ön eki ile belirtmek gerekir.
```bash
cat ./-
```
 
### Level 2 ➔ Level 3
 
* **Konu:** İçinde boşluk karakteri geçen dosya adlarını okuma.
* **Ne öğrenildi:** Boşluklu dosya adları tek tırnak (`'...'`) içine alınarak ya da `\` ile escape edilerek kullanılabilir.
```bash
cat ./'--spaces in this filename--'
```
 
### Level 3 ➔ Level 4
 
* **Konu:** Gizli dosyaları (nokta ile başlayan) görüntüleme.
* **Ne öğrenildi:** `ls` normal modda gizli dosyaları göstermez; `-a` (all) bayrağı ile birlikte kullanılmalıdır.
```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```
 
### Level 4 ➔ Level 5
 
* **Konu:** Bir klasördeki birçok dosya arasından insan tarafından okunabilir (ASCII) olanı bulma.
* **Ne öğrenildi:** `file` komutu bir dosyanın türünü (data, ASCII text, OpenPGP key vb.) tahmin eder. Wildcard (`./*`) ile tüm dosyalar taranıp `grep` ile filtrelenebilir.
```bash
file ./* | grep ASCII
cat ./-file07
```
 
### Level 5 ➔ Level 6
 
* **Konu:** Belirli boyut ve izin kriterlerine göre dosya arama.
* **Ne öğrenildi:** `find` komutu; boyut (`-size`), tür (`-type f`) ve izin (`! -executable`, yani "çalıştırılabilir OLMAYAN") gibi kriterlere göre dosya arayabilir.
```bash
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```
 
### Level 6 ➔ Level 7
 
* **Konu:** Tüm dosya sistemi genelinde, sahiplik (owner/group) bilgisine göre arama ve hata çıktısını gizleme.
* **Ne öğrenildi:** `find /` kök dizinden başlayarak arama yapar; `-group` ve `-user` filtreleri sahiplik bazlı arama sağlar. `2>/dev/null`, izin hatası gibi stderr çıktısını bastırmak için kullanılır.
```bash
find / -size 33c -group bandit6 -user bandit7 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```
 
### Level 7 ➔ Level 8
 
* **Konu:** Büyük bir metin dosyasında belirli bir kelimeyi arama.
* **Ne öğrenildi:** `grep <kelime> <dosya>` ile satır bazında eşleşme aranır.
```bash
grep millionth data.txt
```
 
### Level 8 ➔ Level 9
 
* **Konu:** Bir dosyada yalnızca bir kez geçen (tekil) satırı bulma.
* **Ne öğrenildi:** `uniq -u` yalnızca birbirine **bitişik** olan yinelenen satırları eler; bu yüzden önce `sort` ile aynı satırların yan yana gelmesi sağlanmalıdır.
```bash
sort data.txt | uniq -u
```
 
### Level 9 ➔ Level 10
 
* **Konu:** Büyük ölçüde ikili (binary) veri içeren bir dosyadan okunabilir metinleri çıkarma.
* **Ne öğrenildi:** `strings` bir dosya içindeki yazdırılabilir karakter dizilerini listeler; belirli bir işaretleyiciye (burada `=`) göre `grep` ile daraltılabilir.
```bash
strings data.txt | grep =
```
 
### Level 10 ➔ Level 11
 
* **Konu:** Base64 ile kodlanmış veriyi çözme.
* **Ne öğrenildi:** `base64 -d` (decode) ile Base64 formatındaki içerik düz metne çevrilir.
```bash
base64 -d data.txt
```
 
### Level 11 ➔ Level 12
 
* **Konu:** ROT13 (harf kaydırmalı) şifreleme çözme.
* **Ne öğrenildi:** `tr` komutu karakter setleri arasında birebir eşleme yaparak dönüştürme sağlar; ROT13 çözümü için harfler 13 pozisyon kaydırılır (`A-Za-z` → `N-ZA-Mn-za-m`).
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
 
### Level 12 ➔ Level 13
 
* **Konu:** Hexdump formatındaki veriyi ikiliye çevirme ve iç içe geçmiş, çok katmanlı sıkıştırılmış arşivleri (gzip/bzip2/tar karışık) sırayla açma.
* **Ne öğrenildi:** `xxd -r` bir hex dump'ı orijinal ikili haline geri döndürür. Ardından her adımda `file` (veya gizli sıkıştırmaları da görmek için `file -Z`) ile dosyanın gerçek türü tespit edilip, doğru uzantı verilerek (`mv`) ilgili araçla (`gzip -d`, `bzip2 -d`, `tar -xvf`) açılır. Bu süreç, iç içe kaç katman kaldığı belirsiz olduğundan **tür tespit et → uzantı ver → aç** döngüsü şeklinde tekrarlanır.
```bash
# Hex dump'ı ikiliye çevir
xxd -r data.txt out1
 
# Sıkıştırılmış içeriği de görmek için:
file -Z out1
 
# Her katmanda: türü tespit et, doğru uzantıyı ver, aç
mv out1 out1.gz  && gzip  -d out1.gz
mv out1 out1.bz2 && bzip2 -d out1.bz2
mv out1 out1.tar && tar   -xvf out1.tar
# ... (dosya "ASCII text" olarak görünene kadar tekrarlanır)
cat out_final
```
 
### Level 13 ➔ Level 14
 
* **Konu:** SSH private key ile parolasız kimlik doğrulama ve dosyaları ağ üzerinden aktarma.
* **Ne öğrenildi:** `scp` ile uzak sunucudaki bir dosya yerel makineye indirilir; ardından `ssh -i <key>` ile parola yerine bu özel anahtar kullanılarak bağlantı kurulur.
```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```
 
### Level 14 ➔ Level 15
 
* **Konu:** Yerel bir portta dinleyen ağ servisine kimlik bilgisi gönderme.
* **Ne öğrenildi:** `nc <host> <port>` ile bir TCP portuna bağlanılıp stdin üzerinden veri (burada mevcut şifre) gönderilebilir; servis doğru veriyi alınca bir sonraki seviyenin şifresini döner.
```bash
nc localhost 30000
```
 
### Level 15 ➔ Level 16
 
* **Konu:** Düz `nc` yerine SSL/TLS ile şifrelenmiş bir porta bağlanma.
* **Ne öğrenildi:** Hedef port SSL bekliyorsa düz `nc` bağlantısı çalışmaz; bunun yerine `openssl s_client -connect <host>:<port>` kullanılmalıdır.
```bash
openssl s_client -connect localhost:30001
```
 
### Level 16 ➔ Level 17
 
* **Konu:** Belirli bir port aralığını tarayıp doğru SSL portunu bulma ve karşılığında bir SSH private key almak.
* **Ne öğrenildi:** Geniş bir port aralığında hangi portların açık olduğunu ve hangi servisi çalıştırdığını görmek için `nmap` ile port taraması yapılır; `-p` port aralığını, `-sV` ise servis/versiyon tespitini belirtir. Bu şekilde SSL bekleyen doğru port bulunur. `-quiet` bayrağı ise `openssl s_client` çıktısını sadeleştirir. Doğru porta bağlanıp mevcut şifre gönderildiğinde, sunucu bir sonraki seviyeye ait SSH private key'i döndürür; bu key bir dosyaya kaydedilip izinleri düzeltilerek (`chmod 600`) doğrudan giriş için kullanılır.
```bash
# Port aralığını tarayıp hangi portun neyi çalıştırdığını tespit et
nmap -p 31000-32000 -sV localhost
 
openssl s_client -connect localhost:31790 -quiet
# ... şifreyi gönder, karşılığında gelen private key'i kaydet
```
 
### Level 17 ➔ Level 18
 
* **Konu:** İki benzer dosya arasındaki farkı tespit etme.
* **Ne öğrenildi:** `diff dosya1 dosya2`, iki dosya arasında satır bazlı farkları gösterir — burada eski ve yeni şifre listesi arasındaki tek değişen satır aranmıştır.
```bash
diff passwords.old passwords.new
```
 
### Level 18 ➔ Level 19
 
* **Konu:** `.bashrc` dosyasında oturum açar açmaz otomatik çıkış (`exit`) yapan kısıtlı bir kullanıcı ortamında komut çalıştırma.
* **Ne öğrenildi:** Etkileşimli shell açılmadan, `ssh` komutuna doğrudan çalıştırılacak komut argüman olarak verilerek `.bashrc` içindeki otomatik çıkıştan önce istenen işlem yaptırılabilir.
```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```
 
### Level 19 ➔ Level 20
 
* **Konu:** SUID (setuid) izinli bir binary ile başka bir kullanıcı yetkisiyle komut çalıştırma.
* **Ne öğrenildi:** `-rwsr-x---` izinlerindeki `s` biti, programın sahibi (burada `bandit20`) yetkisiyle çalışacağı anlamına gelir. Böyle bir binary çalıştırılarak normalde erişilemeyen dosyalar okunabilir.
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```
 
### Level 20 ➔ Level 21
 
* **Konu:** Yerel bir portta dinleyici (listener) açıp, bir istemci programın bu porta bağlanmasını sağlama.
* **Ne öğrenildi:** `nc -l -p <port>` ile arka planda bir dinleyici başlatılıp mevcut şifre bu porta yazılır; ardından hedef program (`./suconnect <port>`) bu porta bağlanıp gelen şifreyi doğrular ve karşılığında bir sonraki şifreyi iletir.
```bash
echo "<mevcut_şifre>" | nc -l -p 4444 &
./suconnect 4444
```
 
### Level 21 ➔ Level 22
 
* **Konu:** `/etc/cron.d/` altında tanımlı zamanlanmış görevleri (cron job) inceleme.
* **Ne öğrenildi:** Bir cron job, bir sonraki seviyenin şifresini geçici (`/tmp`) bir dosyaya kopyalayıp izinlerini herkese açık hale (`chmod 644`) getiriyordu; bu dosya doğrudan okunarak şifreye ulaşıldı.
```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<script_tarafından_oluşturulan_dosya>
```
 
### Level 22 ➔ Level 23
 
* **Konu:** Cron script'in çalışma zamanında dinamik olarak (`md5sum` ile) ürettiği dosya adını önceden hesaplama.
* **Ne öğrenildi:** Script, hedef dosya adını `echo "I am user $myname" | md5sum` ile üretiyordu. Aynı komut manuel çalıştırılarak dosya adı önceden tahmin edilip doğrudan okunabildi.
```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh
 
mytarget=$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)
cat /tmp/$mytarget
```
 
### Level 23 ➔ Level 24
 
* **Konu:** Bir cron job'un düzenli olarak belirli bir klasördeki (`/var/spool/<user>/foo`) script'leri, sahibi doğru kullanıcı ise otomatik çalıştırıp sildiğini fark edip kendi script'ini o klasöre bırakma.
* **Ne öğrenildi:** Cron script sadece dosya sahibi (`owner`) doğru kullanıcıya aitse çalıştırıyordu. Kendi hesabımızla (bandit23) yazdığımız, bir sonraki seviyenin şifresini okuyup dünyaya-açık bir dosyaya kaydeden script, hedef klasöre kopyalanarak cron'un otomatik çalıştırması sağlandı.
```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit24
cat /usr/bin/cronjob_bandit24.sh
 
mkdir /tmp/calisma_klasoru && chmod 777 /tmp/calisma_klasoru
cd /tmp/calisma_klasoru
nano answer.sh
# answer.sh içeriği:
#   #!/bin/bash
#   cat /etc/bandit_pass/bandit24 > /tmp/calisma_klasoru/answer
#   chmod 777 /tmp/calisma_klasoru/answer
chmod 777 answer.sh
cp ./answer.sh /var/spool/bandit24/foo
# cron çalışınca:
cat answer
```
 
### Level 24 ➔ Level 25
 
* **Konu:** 4 haneli bir pin kodunu, tüm olasılıkları deneyerek (brute-force) bir ağ servisine gönderme.
* **Ne öğrenildi:** `{0000..9999}` gibi bash range ifadesiyle tüm pin kombinasyonları tek bir döngüde üretilip `nc` üzerinden servise art arda gönderilebilir; doğru pin bulunduğunda servis bir sonraki şifreyi döner.
```bash
for pin in {0000..9999}; do
    echo "<mevcut_şifre> $pin";
done | nc localhost 30002
```
 
### Level 25 ➔ Level 26
 
* **Konu:** Girişte otomatik olarak sınırlı (restricted) bir programı çalıştırıp kapatan bir kullanıcı hesabından, terminal boyutu ve `more`/`vim` davranışından yararlanarak normal bir shell'e kaçış.
* **Ne öğrenildi:** SSH anahtarıyla giriş yapıldığında ekrana uzun bir metin `more` ile sayfa sayfa basılıyordu. Terminal penceresi küçültülerek `--More--` istemi tetiklenip `v` tuşuyla Vim editörüne geçildi; Vim komut modunda `:set shell=/bin/bash` ve `:shell` ile tam bir bash shell'i elde edildi.
```bash
scp -P 2220 bandit25@bandit.labs.overthewire.org:~/bandit26.sshkey .
ssh -i ./bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220
# "--More--" ekranında:  v
# Vim komut satırında:
:set shell=/bin/bash
:shell
cat /etc/bandit_pass/bandit26
```
 
### Level 26 ➔ Level 27
 
* **Konu:** Yine SUID izinli bir binary üzerinden başka kullanıcı yetkisiyle dosya okuma.
* **Ne öğrenildi:** Level 19'daki mantığın devamı — sahibi farklı bir kullanıcı olan çalıştırılabilir bir program, izin verdiği komutları o kullanıcı yetkisiyle işletir.
```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```
 
### Level 27 ➔ Level 28
 
* **Konu:** Bir Git deposunu SSH üzerinden klonlama.
* **Ne öğrenildi:** `git clone ssh://<user>@<host>:<port>/<repo-yolu>` ile uzak bir Git deposu SSH protokolü üzerinden indirilebilir; klasördeki dosyalar normal şekilde incelenir.
```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo
cat readme
```
 
### Level 28 ➔ Level 29
 
* **Konu:** Depodaki güncel dosyada gizlenmiş (redakte edilmiş) bir bilginin, commit geçmişinde hâlâ mevcut olması.
* **Ne öğrenildi:** `git log -p`, tüm commit'lerin diff'lerini gösterir. Güncel `README.md`'de şifre `xxxxxxxxxx` ile maskelenmiş olsa da, daha önceki bir commit'te açık haliyle eklenmişti — geçmiş asla otomatik temizlenmez.
```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
cd repo
git log
git log -p
```
 
### Level 29 ➔ Level 30
 
* **Konu:** Depoda görünmeyen, yalnızca başka bir branch'te bulunan bilgiye ulaşma.
* **Ne öğrenildi:** `git branch -a`, uzak (remote) branch'ler dahil tüm branch'leri listeler. Şifre yalnızca `dev` branch'inde bulunuyordu; `git checkout dev` ile o branch'e geçilip dosya okundu.
```bash
git branch -a
git checkout dev
cat README.md
```
 
### Level 30 ➔ Level 31
 
* **Konu:** Git tag'leri (etiketleri) üzerinden gizlenmiş veriye erişim.
* **Ne öğrenildi:** `git tag`, depodaki tüm etiketleri listeler; `git show <tag-adı>` bir etiketin işaret ettiği commit/objenin içeriğini (burada doğrudan şifreyi) gösterir.
```bash
git tag
git show secret
```
 
### Level 31 ➔ Level 32
 
* **Konu:** `.gitignore` tarafından engellenen bir dosyayı zorla ekleyip depoya push ederek sunucu tarafındaki bir git hook'unu tetikleme.
* **Ne öğrenildi:** `git add -f`, `.gitignore` kurallarını görmezden gelip belirtilen dosyayı zorla stage eder. Beklenen içerikle commit oluşturup `push` edildiğinde, sunucu tarafındaki bir pre-receive/post-receive hook bu işlemi doğrulayıp bir sonraki şifreyi döndürdü.
```bash
echo 'May I come in?' > key.txt
git add -f key.txt
git commit -m "add key.txt"
git push origin master
```
 
### Level 32 ➔ Level 33
 
* **Konu:** Tüm girdiyi büyük harfe çeviren "uppercase shell" adlı kısıtlı bir ortamdan normal bir shell'e kaçış.
* **Ne öğrenildi:** Shell, girilen her komutu büyük harfe çevirip çalıştırıyordu; bu da normal komutları kullanılamaz hale getiriyordu. Ancak `$0` (çalışan shell'in kendisini referans alan özel değişken) büyük harfe çevrilmeden çağrılabildiğinden, `>> $0` ifadesi yeni ve kısıtlanmamış bir `/bin/sh` başlatarak kısıtlamayı aştı.
```bash
>> $0
cat /etc/bandit_pass/bandit33
```
 
### Level 33 (Final)
 
* **Konu:** Oyunun (bu bölümdeki) son seviyesi.
* **Not:** Bu seviyede yeni bir görev yok; `README.txt` OverTheWire ekibinin tebrik mesajını ve diğer wargame'lere yönlendirmeyi içeriyor.
```bash
cat README.txt
```
 
---
 
## 🧠 Genel Çıkarımlar
 
* **Dosya adlandırma tuzakları** (`-`, boşluk, gizli dosya) çoğu zaman `./` öneki, tırnak işareti veya `-a`/`-la` bayraklarıyla aşılır.
* **`find`**, dosya sistemi genelinde boyut, izin, sahiplik gibi çok sayıda kritere göre arama yapabilen en güçlü araçlardan biridir; hata çıktısını `2>/dev/null` ile bastırmak pratik bir alışkanlıktır.
* **Kodlama/şifreleme katmanları** (hex, base64, ROT13, gzip/bzip2/tar) genellikle `file` komutuyla adım adım teşhis edilip ters işlemle çözülür.
* **Ağ servisleri** ile çalışırken düz TCP için `nc`, SSL/TLS için `openssl s_client` kullanılır.
* **Cron job'lar**, çalıştırdıkları script'lerin mantığı incelenerek (dosya adı tahmini, izin kontrolü, geçici dosya sızıntısı gibi) istismar edilebilir.
* **SUID binary'ler**, normalde erişilemeyen dosyalara sahibinin yetkisiyle erişim sağlayan yaygın bir yükseltme (privilege escalation) yöntemidir.
* **Kısıtlı shell'ler** (rbash, otomatik `.bashrc` çıkışı, komut büyük/küçük harf dönüştürme vb.) çoğunlukla bir yan kapı bırakır: bir editörün shell'e geçiş özelliği, `$0` gibi özel değişkenler veya doğrudan komut argümanı geçirme gibi.
* **Git**, yalnızca en güncel dosya içeriğini değil, tüm commit geçmişini (`git log -p`), branch'leri (`git branch -a`) ve tag'leri (`git tag`/`git show`) da saklar — hassas veri bir noktada commit edildiyse geçmişten silinmediği sürece hâlâ erişilebilirdir.
