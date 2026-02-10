# Ubuntu-Touch-Home-Server
Transformă o tabletă veche într-un server cloud personal (NAS), accesibil de oriunde din lume.
# Acest ghid a fost testat pe:
## Hardware : 
**Dispozitiv:** Lenovo Tab M10 HD Gen2 cu Ubuntu Touch (acest sistem de operare a fost compatibil cu acest model de tabletă conform site-ului Ubuntu Ports) 

**Stocare:** SSD NVME 1TB;

**Conectivitate:**

- 1. adaptor exteren pentru SSD NVME cu mufă TYPE-C (mamă);
- 2. adaptor extern TYPE-C (tată) la USB A , HDMI și TYPE-C ( mamă ,pentru încărcare);
- 3. adaptor USB 3.0 SATA la HDD (rack) ;

## Software :

- Terminal: Aplicația nativă din Ubuntu Touch.
- Tailscale: Pentru VPN Mesh securizat (acces remote fără port forwarding).
- FileBrowser: pentru a ne vedea fișierele.
- Smartmontools: Pentru monitorizarea sănătății discului.

# Pasul 1 : Pregătirea sistemului 
Ubuntu Touch este un sistem securizat, cu partiția de sistem setată pe "Read-Only" (Doar Citire). Pentru a instala unelte de diagnoză, trebuie să permitem scrierea temporar.
**Comandă de deblocare**

```
sudo mount -o remount,rw /
```

**Structura comenzii**

- `sudo` : cere permisiuni de administrator suprem.
- `mount` : utilitarul care gestionează sistemele de fișiere.
- `-o remount,rw ` : Opțiunea (o) care spune: "Montează din nou partiția curentă, dar în mod Scriere/Citire (read/write)".
- ` / ` : Reprezintă "Rădăcina" (Root) sistemului de operare.

# Pasul 2 : Sănătatea discului.
**Instalăm pachetul necear:**

```
sudo apt update
sudo apt install smartmontools
```
- `apt` : (Advanced Package Tool) Managerul de pachete.
- `install` : Comanda de instalare.
- `smartmontools` : Pachetul software care conține uneltele de monitorizare.

 **Identificăm discul:**
 
```
lsblk
```
- `lsblk`: (List Block devices) Afișează toate discurile. Căutați discul extern după mărime (de obicei este /dev/sda sau /dev/sdb).
- **În acest ghid, vom folosi /dev/sda ca exemplu.**

**Verificăm sănătatea**
-Dacă discul scoate sunete ciudate (clicuri, țăcănituri) sau se blochează des sau nu este văzut cum trebuie pe pc, este defect fizic, indiferent ce zice comanda!
Software-ul vede doar erorile logice, nu și pe cele mecanice grave.
### Pentru o verificare completă a hardurilor este bine să folosiți o aplicație precum:
- Hard Disk Sentinel
- Victoria 5.37
- MiniTool Partition Wizard

Pentru o verificare parțială a HDD/SSD -urilor prin tabletă aveți nevoie de punctul i și iii de la **Conectivitate** și folosiți următoarele comenzi:

```
sudo smartctl -H /dev/sda` 
```
- `smartctl`: Comanda de control SMART.
- `-H`: (Health) Cere un raport rapid ("Passed" sau "Failed").
- `/dev/sda:`: Discul pe care îl interogăm.
**Avertisment important**: Chiar dacă testul returnează PASSED, asta nu garantează 100% că discul este perfect, software-ul vede doar erorile logice, nu și pe cele mecanice grave.**(pentru HDD)**

## Opțional ,dar RECOMANDAT: Test detaliat sau citire loguri.
```
sudo smartctl -t short /dev/sda   
sudo smartctl -t long /dev/sda
sudo smartctl -l selftest /dev/sda 
```
- ` -t ` : vine de la Test;
- `short` : face un test scurt de aproximativ de 2 minute;
- `long` : face un test extins ce durează mai mult de 30 de minute în funcție de HDD/SSD;
- ` -l selftest ` : -l vine de la jurnal (Log) ,iar selftest arată dacă există teste făcute până in momentul aplicării comenzii adică trebuie rulat DUPĂ ce ați așteptat finalizarea testului(short sau long).

# Pasul 3 Montarea și Permisiunile 
Pentru ca aplicațiile (precum FileBrowser) să poată vedea și modifica fișierele de pe SSD, trebuie să le montăm într-un folder deținut de utilizatorul `phablet`.
**Creăm folderul destinație (doar prima dată).**
```
mkdir -p /home/phablet/Downloads/SSD
```

- `mkdir` : (Make Directory) Comanda standard pentru a crea un folder nou.
- `-p` : (Parents) Această opțiune este foarte importantă din două motive:
1.Creează tot traseul: Dacă din greșeală folderul `Downloads` nu ar exista, această comandă îl creează automat și pe el, și pe `SSD` .
2.Dacă folderul `SSD` există deja (poate ați mai rulat comanda o dată), `-p` îi spune sistemului să nu dea eroare ("File exists"), ci să treacă mai departe liniștit.
- `/home/phablet/Downloads/SSD` : Calea completă unde va fi creat folderul.

**Montarea discului (Format ext4).**
```
sudo mount /dev/sda1 /home/phablet/Downloads/SSD
```
- `/dev/sda1` : Partiția efectivă de pe disc (numărul 1).

**Corectarea permisiunilor (FOARTE IMPORTANT).**
```
sudo chown -R phablet:phablet /home/phablet/Downloads/SSD
```
- `chown`: (Change Owner) Schimbă proprietarul fișierelor.
- `-R`: (Recursive) Aplică regula tuturor fișierelor și folderelor din interior.
- `phablet:phablet`: Setează utilizatorul phablet și grupul phablet ca proprietari.

Fără această comandă, fișierele ar aparține sistemului `Root` și nu le-ați putea deschide.

### Deconectarea în siguranță 
** Nu scoateți niciodată cablul USB fără a demonta discul software! Riscați coruperea datelor.**
```
sudo umount /home/phablet/Downloads/SSD
```
- `umount` : (Un-mount) Desface legătura, golește memoria cache și scrie ultimele date pe disc.

```
sudo eject /dev/sda
```
- `eject` : Oprește alimentarea dispozitivului USB pentru extragere sigură.

# Pasul 4: Instalare și Configurare Tailscale
Tailscale este serviciul VPN care ne permite să accesăm tableta de oriunde, chiar dacă nu suntem acasă.
```
curl -fsSL https://tailscale.com/install.sh | sh
```
- `curl` : (Client URL) O unealtă care descarcă date de pe internet în terminal.
- `fsSL` : Opțiuni care îi spun să fie silențios dacă nu sunt erori, dar să urmeze redirectările (Link-uri).
- `|` : (Pipe / Țeavă) Ia ce a descărcat curl și trimite direct la următoarea comandă.
- `sh` : (Shell) Execută scriptul descărcat pentru a instala programul.

```
sudo tailscale up
```
**Această comandă va genera un link lung în terminal. Copiați link-ul, puneți-l în browser pe telefon/PC și logați-vă cu contul Google/Microsoft. După logare, tableta e conectată!**


```
tailscale ip -4
```
Îți va arăta o adresă de genul 100.x.y.z. Acela este noul ip al tabletei tale în rețeaua VPN.

Iar pentru oprirea Tailscale folosiți comanda de mai jos:
```
sudo tailscale down
```

# Pasul 5: Instalare și Configurare FileBrowser
Aceasta este interfața grafică (site-ul web) prin care ne vedem fișierele.
Comanda de instalare ( înainte de a instala nu uitați să puneți sistemul pe Read/Write):
```
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
```
Acordarea permisiunii de rulare (Siguranță): Uneori, sistemul descarcă fișierul dar nu îl lasă să ruleze. Îl "deblocăm" manual:
```
sudo chmod +x /usr/local/bin/filebrowser
```

Funcționează exact ca la Tailscale: descarcă scriptul de instalare și îl execută.

Pornirea serverului:
```
filebrowser -r /home/phablet/Downloads/SSD -a 0.0.0.0 -p 8080
```
- `filebrowser` : Pornește aplicația.
- `-r` : (Root) Definește "Rădăcina", adică folderul pe care avem voie să îl vedem. Noi punem calea către SSD-ul montat anterior.
- `-a 0.0.0.0` : (Address) Îi spune serverului să asculte pe toate conexiunile (nu doar local). Fără asta, nu l-am putea accesa prin Tailscale.
- `-p 8080` : (Port) "Ușa" prin care intrăm.

Sau doar `sudo chmod +x filebrowser` dacă sunteți în același folder cu el.

## Pentru a intra pe server, dispozitivul de pe care accesezi (telefonul tău personal, laptopul, etc.) trebuie să fie în aceeași rețea VPN.
Pasul 1: Pregătirea telefonului/laptopului (Clientul)
- Instalează aplicația Tailscale pe telefonul tău (din Google Play / App Store) sau pe PC.
- Loghează-te în aplicație cu ACELAȘI CONT (Google/Microsoft) folosit pe tableta-server.
- Activează VPN-ul (butonul Active să fie verde).
- Acum telefonul și tableta sunt în aceeași "cameră virtuală".

Pasul 2: Accesarea fișierelor
- Deschide browserul (Chrome, Safari) pe telefon.
- Scrie adresa IP a tabletei urmată de portul 8080:
```
 http://IP-TAILSCALE:8080
```
Exemplu: http://100.85.20.5:8080
 Se va deschide interfața FileBrowser.

Pasul 3: Logare în FileBrowser
User: admin
Parolă: admin
** ATENTIE!** Se poate întâmpla ca parola să nu funcționeze , și o puteți modifica din terminal cu această comandă:
```
filebrowser -d /home/phablet/filebrowser.db users update admin --password "ParolaTaNouaAici"
```
- `filebrowser` : Apelăm programul.
- `-d /home/phablet/filebrowser.db` : (Database) Îi spunem exact unde se află "creierul" aplicației (baza de date). Fără acest parametru, comanda nu știe ce utilizator să modifice.
- `users update` : Îi dăm ordin să actualizeze un utilizator existent.
- `admin` : Numele utilizatorului pe care îl modificăm.
- `--password "..."` : Parametrul care definește noua parolă. (Puneți noua parolă între ghilimele!).

Sau în cazul în care nu mai există utilizatorul sau s-a modificat , poți creea unul cu drepturi de admin: 
```
filebrowser -d /home/phablet/filebrowser.db users add phablet "ParolaNoua" --perm.admin
```
- `users add` : Comanda de adăugare.
- `phablet` : Numele noului utilizator (poate fi orice nume vrei).
- `--perm.admin` : Îi dă drepturi depline de administrator.

Pentru a opri FileBrowser apăsați butonul Ctrl+C (acesta va opri procesul sau dacă vă faceți un script ,trebuie să folosiți comanda `pkill -f filebrowser`)

Pentru a opri Tailscale : `sudo tailscale down`
