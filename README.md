# Forensics Workshop - DDC 2022

## Intro

Denne forensics workshop er inddelt i tre mindre sessioner, der har hver deres mappe:
1. file_analysis
2. steganography
4. memory_analysis

`file_analysis` og `steganography` har to undermapper:
- `examples`: eksempelfiler brugt i løbet af sessionen
- `exercises`: mappe med øvelser og øvelsesfiler. Øvelserne er beskrevet i `README.md` og har hver en række hints, man kan få vist, hvis man sidder lidt fast. Filen `SOLUTIONS.md` indeholder løsningen på alle øvelserne samt til tider lidt ekstra info.

`memory_analysis` indeholder bare en enkelt øvelse, nemlig en case, hvor et memory dump skal analyseres. Gennemgang + spørgsmål findes i mappen i `README.md`.

## Download projektet

Hvis du har `git` installeret, kan du hente projektet ned med

    git clone https://github.com/Nissen96/forensics-workshop.git

Til memory analysis delen ligger filen `malpdf.zip`, der er uploadet via Git LFS fordi den er over 100 MB. Hvis du har Git LFS installeret, bliver den automatisk hentet ned med `clone`, ellers skal du lige hente den manuelt.

Øvelserne er skrevet i markdown, så det er smartest at åbne dem og deres solutions direkte i GitHub (eller med en markdown viewer), så de er korrekt formateret. Så undgår du også at få spoilet hints, du ikke vil se.

## Ressourcer

Her er en række relevante links, der bliver brugt eller nævnt i Workshoppen:

- Stego-toolkit: https://github.com/DominicBreuker/stego-toolkit
- CTF Field Guide: https://trailofbits.github.io/ctf/forensics/
- HackTricks: https://book.hacktricks.xyz/forensics
- Forensics labs: https://cyberdefenders.org/
- 13Cubed: https://www.youtube.com/c/13cubed

## Tools

Nedenfor følger en liste over tools, der vil være gode at installere før workshoppen, hvis du har tid. De er også alle gode at have til senere CTFer.

De fleste er mest naturlige at finde og bruge på Linux, og det vil være nemmest for jer selv, hvis I har adgang til en Linux maskine. Det kan på MacOS og Windows installeres i en VM, men kører du Windows er det også meget nemt at sætte op med WSL2. Quick guide:

1. Åbn PowerShell som administrator og kør

<!-- -->

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

2. Genstart Windows
3. Åbn Powershell som administrator og kør 

<!-- -->

    wsl --set-default-version 2

4. Åbn Microsoft Store, søg på "kali" og installér "Kali Linux"
5. Du har nu installeret Kali Linux på din Windows. Kør programmet
6. Ved første kørsel skal du opsætte en ny bruger - den beder dig selv vælge et username og password
7. Kør `sudo apt update` og `sudo apt upgrade` for at køre første update (password den spørger om er det, du lige valgte)

Alternativt kan du bruge det online toolkit "CyberChef": https://gchq.github.io/CyberChef/. Det kommer med en lang række operationer, du kan anvende på dit input. Operationerne kan chaines til en "opskrift" og gør det meget nemt at lege rundt med forskellige filer og inputs. Ikke alle opgaver kan løses med CyberChef, men en del af de første kan.

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

Volatility 2 er nemmest at hente og bruge som standalone program. Download til dit OS her: https://www.volatilityfoundation.org/releases, unzip og kør programmet via en terminal. Det virker til, der ikke er et fungerende link til version 2.6 for Mac og Linux, bare brug version 2.5 til dem.

Volatility 3 kræver Python 3 og dependencies installeres med Pip. Sørg for begge er installeret med

    sudo apt install python3 python3-pip

Hent og installer volatility3 og dens dependencies:

    git clone https://github.com/volatilityfoundation/volatility3.git
    cd volatility3
    pip3 install -r requirements.txt

Volatility bruger symbol tables for forskellige operativsystemer for at kunne analysere dem korrekt. Download følgende ZIP-filer og placer dem i mappen `volatility3/symbols/` (inde i hovedmappen, så `volatility3/volatility3/symbols` udefra):
- Windows: https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip
- Linux: https://downloads.volatilityfoundation.org/volatility3/symbols/linux.zip
- MacOS: https://downloads.volatilityfoundation.org/volatility3/symbols/mac.zip

Bemærk de filer ikke afhænger af, hvilket styresystem du selv bruger, men hvilket styresystem, du skal analysere et memory dump fra. Du kan evt. skippe MacOS fra starten, det ses noget sjældnere i en CTF challenge, og vi skal ikke bruge det i workshoppen (heller ikke Linux).
