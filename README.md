# Forensics Workshop - DDC 2023

## Intro

Denne workshop er inddelt i tre hovedområder, der hver har tilknyttet et oplæg samt en exercise session.
Vi vil bruge CTFd til øvelserne, så de mapper, der ligger herinde er ikke relevante i år, kun denne README.

## VM opsætning

Jeg har forberedt en Kali VM med alle de nødvendige tools og filer på, som nemt kan importeres i VirtualBox.

Vær opmærksom på, at filen er næsten 15 GB, så den kan tage et godt stykke tid at downloade og sætte op.

Steps:

1. Download VirtualBox fra https://www.virtualbox.org/ og installér
2. Hent OVA-filen til workshoppen herfra: https://nextcloud.ntp-event.dk:8443/s/cGMyWDTBwCg5bLJ/download/Forensics%20Workshop%20-%20DDC%202023%20%28kali%29.ova
3. Åben VirtualBox, klik "Tools" og herefter "Import"
4. I inputfeltet skal stien til OVA-filen stå, så klik på mappeikonet og vælg filen
5. Under indstillinger kan du evt. vælge en anden Machine Base Folder
6. Sørg for at "Import hard drives as VDI" er markeret
7. Vælg Finish og vent på import

Du kan nu køre den virtuelle maskine og tilgå de tools og filer, vi skal bruge til workshoppen.

Username og password er begge `kali`.

## Tools

Du er velkommen til at bruge din egen VM / setup, men det antages du har adgang til Linux.

Nedenfor er listen over de tools, vi kommer til at bruge, samt installationsinstruktioner til hver. De er alle gode at have til fremtidige CTFs.

Alternativt (eller som supplement) kan du bruge det online toolkit "CyberChef": https://gchq.github.io/CyberChef/. Det kommer med en lang række operationer, du kan anvende på dit input. Operationerne kan chaines til en "opskrift" og gør det meget nemt at lege rundt med forskellige filer og inputs. Ikke alle opgaver kan løses med CyberChef, men en del af de første kan.

### Linux

#### File Analysis

**xxd** (hexdump)

    sudo apt install xxd

**hexedit** (CLI hex editor)

    sudo apt install hexedit

**ghex** (GUI hex editor)

    sudo apt install ghex

**exiftool** (extract metadata)

    sudo apt install exiftool

**pngcheck** (check PNG filer)

    sudo apt install pngcheck

#### Steganography

**bgrep** (binary grep)

    sudo apt install curl
    mkdir -p $HOME/.local/bin
    curl -L 'https://github.com/tmbinc/bgrep/raw/master/bgrep.c' | sudo gcc -O2 -x c -o /usr/local/bin/bgrep -

**binwalk** (file carving)

    sudo apt install binwalk

**foremost** (file carving)

    sudo apt install foremost

**stegsolve** (GUI image stego)

    wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve
    chmod +x stegsolve

Kræver Java! Køres med

    java -jar stegsolve

Flyt evt. til en specifik mappe og opret et alias

    mkdir ~/tools
    mv stegsolve /tools
    alias stegsolve='java -jar ~/tools/stegsolve'

Nu kan programmet bare køres med kommandoen `stegsolve` i din terminal. Tilføj evt. aliaset til din `.bashrc` for at persiste det til næste session.

**steghide** (stego tool)

    sudo apt install steghide

**stegseek** (crack steghide)

    wget https://github.com/RickdeJager/stegseek/releases/download/v0.6/stegseek_0.6-1.deb
    sudo apt install ./stegseek_0.6-1.deb
    rm stegseek_0.6-1.deb

`stegseek` skal bruge en wordlist med passwords den kan tjekke. Den leder som default efter wordlisten `rockyou.txt`, der følger med Kali og er den mest brugte password list. Har du ikke Kali (eller vil du bruge en anden wordlist) kan du finde en række muligheder her: https://github.com/danielmiessler/SecLists.

**zsteg** (PNG/BMP textual stego)

    sudo apt install ruby
    sudo gem install zsteg

#### Memory Analysis

Til memory analysis skal vi bruge programmet Volatility. Det findes i version 2 og 3. Version 3 er lidt nemmere at bruge, men version 2 har stadig nogle plugins, der ikke findes til version 3 endnu. Det er en god idé at installere begge, så du har alle funktioner klar.

Volatility 2 er nemmest at hente og bruge som standalone program. Unzip og kør programmet via en terminal:

- Linux: http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_lin64_standalone.zip
- Windows: http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_win64_standalone.zip
- Mac: http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_mac64_standalone.zip

Volatility 3 kræver Python 3, og dependencies installeres med Pip. Sørg for begge er installeret med

    sudo apt install python3 python3-pip

Hent og installer volatility3 og dens dependencies:

    git clone https://github.com/volatilityfoundation/volatility3.git
    cd volatility3
    pip3 install -r requirements.txt

Volatility bruger symbol tables for forskellige operativsystemer for at kunne analysere dem korrekt. Download følgende ZIP-filer og placer dem i mappen `volatility3/symbols/` (inde i hovedmappen, så `volatility3/volatility3/symbols` udefra):
- Windows: https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip
- Linux: https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip
- MacOS: https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip

Bemærk de filer ikke afhænger af, hvilket styresystem du selv bruger, men hvilket styresystem, du skal analysere et memory dump fra. Du kan skippe MacOS fra starten, det ses noget sjældnere i en CTF challenge, og vi skal ikke bruge det i workshoppen (heller ikke Linux, men det ses regelmæssigt).


### Windows

Bruger du Windows, er det hurtigt at sætte en Linux op i WSL:

1. Åbn PowerShell som administrator og kør

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

2. Genstart Windows
3. Åbn Powershell som administrator og kør 

    wsl --set-default-version 2

4. Åbn Microsoft Store, søg på "kali" og installér "Kali Linux"
5. Du har nu installeret Kali Linux på din Windows. Kør programmet
6. Ved første kørsel skal du opsætte en ny bruger - den beder dig selv vælge et username og password
7. Kør `sudo apt update` og `sudo apt upgrade` for at køre første update (password den spørger om er det, du lige valgte)

## Repo

OBS: Vi kommer ikke til at følge denne struktur i år, vi bruger i stedet CTFd!

Denne forensics workshop er inddelt i tre mindre sessioner, der har hver deres mappe:

1. file_analysis
2. steganography
4. memory_analysis

`file_analysis` og `steganography` har to undermapper:
- `examples`: eksempelfiler brugt i løbet af sessionen
- `exercises`: mappe med øvelser og øvelsesfiler. Øvelserne er beskrevet i `README.md` og har hver en række hints, man kan få vist, hvis man sidder lidt fast. Filen `SOLUTIONS.md` indeholder løsningen på alle øvelserne samt til tider lidt ekstra info.

`memory_analysis` indeholder bare en enkelt øvelse, nemlig en case, hvor et memory dump skal analyseres. Gennemgang + spørgsmål findes i mappen i `README.md`.

Øvelserne er skrevet i markdown, så det er smartest at åbne dem og deres solutions direkte i GitHub (eller med en markdown viewer), så de er korrekt formateret. Så undgår du også at få spoilet hints, du ikke vil se.

OBS: Vi kommer ikke til at følge denne struktur i år, vi bruger i stedet CTFd!

## Ressourcer

Her er en række relevante links, der bliver brugt eller nævnt i Workshoppen:


- CTF Field Guide: https://trailofbits.github.io/ctf/forensics/
- HackTricks: https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology
- Forensics labs: https://cyberdefenders.org/blueteam-ctf-challenges/
- Flere forensics labs: https://blueteamlabs.online/
- 13Cubed: https://www.youtube.com/c/13cubed
- Stego-toolkit: https://github.com/DominicBreuker/stego-toolkit
- Aperi'solve: https://aperisolve.com/
