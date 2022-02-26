# Forensics Workshop - DDC 2022

## Intro

Denne forensics workshop er inddelt i fire mindre sessioner, der har hver deres mappe:
1. file_analysis
2. steganography
3. disk_analysis
4. memory_analysis

Hver session har to undermapper:
- `examples`: eksempelfiler brugt i løbet af sessionen
- `exercises`: mappe med øvelser og øvelsesfiler. Øverlserne er beskrevet i `README.md` og har hver en række hints, man kan få vist, hvis man sidder lidt fast. Filen `SOLUTIONS.md` indeholder løsningen på alle øvelserne samt til tider lidt ekstra info.

## Download projektet

Hvis du har `git` installeret, kan du hente projektet ned med

    git clone https://github.com/Nissen96/forensics-workshop.git

Ellers kan du downloade filerne manuelt.

Øvelserne er skrevet i markdown, så det er smartest at åbne dem og deres solutions her på siden (eller med en markdown viewer), så de er korrekt formateret. Så undgår du også at få spoilet hints, du ikke vil se.

## Tools

Vi skal bruge en del forskellige tools til øvelserne. De fleste er mest naturlige at finde og bruge på Linux, og det vil være nemmest for jer selv, hvis I har adgang til en Linux maskine - både nu og generelt til CTFer. Gerne bare i en VM eller via WSL.

Jeg har forsøgt at finde MacOS og Windows udgaver/alternativer til alle tools, så alle bør kunne være med uanset.

Nedenfor kommer en liste over tools, der vil være gode at installere før workshoppen, hvis du har tid. De er også alle gode at have til senere CTFer.

Alternativt kan du bruge det online toolkit "CyberChef": https://gchq.github.io/CyberChef/. Det kommer med en lang række operationer, du kan anvende på dit input. Operationerne kan chaines til en "opskrift" og gør det meget nemt at lege rundt med forskellige filer og inputs. Ikke alle opgaver kan løses med CyberChef, men en del af de første kan.

### Linux

### File Analysis

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

### Steganography


### Disk Analysis


### Memory Analysis


### Windows

### File Analysis

**file** (filetype detection)

Download og udpak https://github.com/nscaife/file-windows/releases/download/20170108/file-windows-20170108.zip

Programmet `file.exe` kan nu køres via PowerShell el. CMD, f.eks.

    C:\<DOWNLOAD PATH>\file-windows-20170108\file.exe file.png

(omdøb evt. bare til `file` og tilføj filplaceringen til din PATH)

**strings** (find ASCII strings in file)

Download og unzip https://download.sysinternals.com/files/Strings.zip

Programmet `strings.exe` kan nu køres via PowerShell el. CMD, f.eks.

    C:\<DOWNLOAD PATH>\Strings\strings.exe file.png

(omdøb evt. bare til `strings` og tilføj filplaceringen til din PATH)

**HxD** (hex editor)

Download og unzip https://mh-nexus.de/downloads/HxDSetup.zip og kør installeren

**exiftool** (extract metadata)


Download og kør https://oliverbetz.de/cms/files/Artikel/ExifTool-for-Windows/ExifTool_install_12.40_64.exe

**pngcheck** (check PNG filer)

Download og unzip http://prdownloads.sourceforge.net/png-mng/pngcheck-3.0.3-win32.zip?download

Programmet `pngcheck.win64.exe` kan nu køres via PowerShell el. CMD, f.eks.

    C:\<DOWNLOAD PATH>\pngcheck-3.0.3-win32\pngcheck.win64.exe file.png

(omdøb evt. bare til `pngcheck` og tilføj filplaceringen til din PATH)

### Steganography


### Disk Analysis


### Memory Analysis


### MacOS

### File Analysis

**Hex Fiend** (hex editor)

Download og installér https://github.com/HexFiend/HexFiend/releases/download/v2.14.1/Hex_Fiend_2.14.1.dmg

**exiftool** (extract metadata)

Download og installér https://exiftool.org/ExifTool-12.40.dmg

**pngcheck** (check PNG filer)

    brew install pngcheck

### Steganography


### Disk Analysis


### Memory Analysis
