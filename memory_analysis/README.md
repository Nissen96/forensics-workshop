# Malicious PDF

## Case

Company X har kontaktet dig og bedt dig foretage en forensics analyse i forbindelse med en nylig hændelse. En af deres medarbejdere modtog en e-mail fra en anden medarbejder med et link til en PDF-fil. Ved åbning af filen lagde medarbejderen ikke mærke til noget særligt, men de har for nylig haft mistænkelig aktivitet på deres bankkonto.

Den nuværende teori er, at brugeren har modtaget en e-mail med en URL til et forfalsket PDF-dokument. Ved åbning af dokumentet i Acrobat Reader blev et ondsindet JavaScript program kørt, der overtog ofrets system.

Company X har taget et memory dump af medarbejderens maskine og har spurgt dig om at analysere den virtuelle hukommelse og besvare deres spørgsmål.

## Intro

Unzip `malpdf.zip`. Prøv at analysere Windows dumpet `malpdf.vmem` med `volatility 2` og/eller `volatility 3` og svar på de følgende spørgsmål. Det antages her, at dit volatility 2 program er navngivet `vol2` og volatility 3 `vol3` - udskift ellers med din egen path (hedder som default typisk bare `vol.py`).

Tip: Lav en `evidence` mappe og gem resultatet fra alle scanninger der, så de kun skal køres én gang.

## Spørgsmål

#### Profil

Volatility 2 kræver en *profil* af dumpet, for at kunne forstå og analysere det (ikke nødvendigt med Volatility 3, der skal du bare have downloadet symbol tables).

Til det kan man bruge `imageinfo` og `kdbgscan`. `imageinfo` giver forslag til profiler ud fra dumpet og kører hurtigere en `kdbgscan`, men `kdbgscan` er beregnet til at være mere pålideligt og forsøger at finde den rigtige profil med sikkerhed.

Prøv at køre `vol2 -f malpdf.vmem imageinfo`. Hvilke profiler foreslår den?

<details>
	<summary>Svar</summary>
	Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86
</details>

Kør nu `vol2 -f malpdf.vmem kdbgscan`. Hvad finder den?

<details>
	<summary>Svar</summary>
	I dette tilfælde de samme to profiler. Tip: Tjek PsActiveProcessHead og PsLoadedModuleList for hver potentiel profil. Hvis der står (0 processes) og (0 modules) er det sandsynligvis en false positive.
</details>

<details>
	<summary>Konklusion</summary>
	Begge profiler bør være fine, lad os bare bruge WinXPSP2x86. Alle plugins skal så køres med <code>vol2 -f malpdf.vmem --profile WinXPSP2x86 PLUGIN_NAVN</code>
</details>

#### Basic info

Med volatility 2 kunne `imageinfo` bruges til at få lidt basis info om dumpet. Med volatility 3 kan du køre

	vol3 -f malpdf.vmem windows.info

Hvornår blev dumpet taget?

<details>
	<summary>Svar</summary>
	Se under "Image date and time" i vol2 eller "SystemTime" i vol3: 2010-02-27 20:12:38
</details>

#### Command Line

Du kan tjekke alle kommandoer brugeren (eller andre applikationer) har kørt med

	vol2 -f malpdf.vmem --profile WinXPSP2x86 cmdline
	vol3 -f malpdf.vmem windows.cmdline

Hvilken kommando blev kørt som den næstsidste, hvilket program er det og hvad var processens ID?

<details>
	<summary>Svar</summary>
	AcroRd32.exe med PID 1752. En Google søgning fortæller, at programmet er Adobe Acrobat Reader, så det har muligvis åbnet den PDF, medarbejderen downloadede
</details>

Hvilken kommando resulterede i processen med PID 888?

<details>
	<summary>Svar</summary>
	firefox.exe
</details>

#### Processer

Lad os se lidt nærmere på systemets kørende processer. Her har volatility primært tre forskellige plugins: `pslist` til at liste processer, `pstree` til at vise dem i en træstruktur og `psscan`, der også viser skjulte processer.

Prøv at køre alle tre og gem resultaterne:

	vol3 -f malpdf.vmem windows.pslist > pslist.txt
	vol3 -f malpdf.vmem windows.pstree > pstree.txt
	vol3 -f malpdf.vmem windows.psscan > psscan.txt

Hvor mange processer finder de?
<details>
	<summary>Svar</summary>
	27 processer
</details>

Hvilken process har PID 644, og hvornår blev den startet?

<details>
	<summary>Svar</summary>
	Det er winlogon.exe med CreateTime 2010-02-26 03:34:04.000000
</details>

Vi så tidligere `AcroRd32.exe` havde process ID 1752 - men hvad er dens parent PID (PPID)? Og hvilken process er det?

<details>
	<summary>Svar</summary>
	PPID er 888 og er firefox.exe, som vi også så under kørte commands.
</details>

#### Memory Dump

Vi ved nu, at der er åbnet en Firefox process, der har åbnet Adobe Acrobat Reader. Det stemmer fint overens med vores viden om, at medarbejderen klikkede på et link. Det er så blevet åbnet i Firefox, en PDF er blevet downloadet, og den PDF er gennem Firefox åbnet i Adobe Acrobat Reader. Det ville være interessant at se, hvilken side dokumentet blev hentet fra.

Vi kan printe al memory fra en bestemt process (i det her tilfælde `firefox.exe` med PID 888). Start med at lave en `dump` folder:

	mkdir dump

Du kan køre `memdump` for at dumpe hukommelsen for en specifik process:
	
	vol2 -f malpdf.vmem --profile WinXPSP2x86 memdump -p 888 --dump-dir dump

Vi kan hoppe ind i mappen `dump` og køre vores klassiske `strings | grep` kommando for at lede efter en URL med en PDF:

	strings 888.dmp | grep -i "http://.*pdf"

(`-i` bruges til case insensitive search). Hvilken URL finder du?

<details>
	<summary>Svar</summary>
	http://search-network-plus.com/cache/PDF.php?st=Internet%20Explorer%206.0
</details>

Herudover vil det være relevant at dumpe memory for PDF-readeren, så vi potentielt kan få adgang til PDF-filen:

	vol2 -f malpdf.vmem --profile WinXPSP2x86 memdump -p 1752 --dump-dir dump

Vi kan bruge `foremost` til at udtrække PDF-filer fra dumpet:

	foremost -t pdf 1752.dmp

Hvor mange PDF-filer finder den?

<details>
	<summary>Svar</summary>
	7
</details>

Prøv at tjekke størrelsen på dem - husk `foremost` extracter på baggrund af file signatures osv., så nogle extractions er ikke nødvendigivs brugbare filer. Hvilke af dem vil du vurdere kunne have relevans?

<details>
	<summary>Svar</summary>
	Kun de sidste to har en størrelse på mere end nogle få hundrede bytes.
</details>

Muligvis er én af de PDF-filer malicious og indeholder malware. En teknik til at tjekke det er at lave et MD5-hash af filen, og slå det op på siden VirusTotal. Hashene kan laves med

	md5sum *

(`*` for at køre kommandoen på alle filer i mappen).

Prøv at copy paste hashet fra hver PDF-fil ind i https://www.virustotal.com/gui/home/search. Finder du noget på nogle af dem?

<details>
	<summary>Svar</summary>
	Sidste PDF med hash f32aa81676c7391528afe08e437765cc er flagged som malicious
</details>

Den ondsindede PDF detectes af flere AntiViruser som et PDF exploit, men flere nævner også JS, altså JavaScript. Prøv at køre en `strings` på PDF'en og `grep` efter JavaScript. Giver det noget?

<details>
	<summary>Svar</summary>
	Ja, linjen <code>&lt;&lt;&sol;S&sol;JavaScript&sol;JS 1054 0 R&gt;&gt;</code> bliver fundet, så PDF-filen kører sandsynligvis noget ondsindet JavaScript kode.
</details>

Næste step ville være at forsøge at extracte og analysere JavaScript malwaren fra PDF-filen, men det er lidt mere avanceret. Hvis du vil prøve det, kan du bare læse videre, ellers kan du springe ned til næste afsnit om netværk.

#### Malware Analyse - Avanceret, kan skippes

For at extracte JavaScript koden fra PDF'en, kan du bruge toolet `peepdf` - PDF Analysis Tool: https://eternal-todo.com/tools/peepdf-pdf-analysis-tool. Klik på `Download it!`, hent og extract ZIP-filen og kør programmet med `python2 peepdf_0.3/peepdf.py` (kræver Python 2). 

Du kan nu køre interactive mode på PDF'en med malware med

	python2 ./peepdf_0.3/peepdf.py -i 00601560.pdf

Dette vil med det samme give dig noget info om PDF'en, inklusive hvilke *objects* og *streams* den indeholder (find mere om PDF-formatet online, det er ret komplekst). Du kan bl.a. se følgende:

	Objects with JS code (1): [1054]

Object 1054 indeholder altså JavaScript kode, og vi kan extracte det med

	object 1054 > malware.js

Du kan afslutte interactive mode med Ctrl+D. Malwaren ligger nu i `malware.js`, men er obfuscated - altså der er gjort meget for at skjule, hvad koden gør, bl.a. ved at give alle variable og funktioner random navne, fjerne al whitespace, encode værdier på mærkelige måder, som så bliver decoded af nogle af funktionerne, etc.

Analyse og reverse engineering af obfuscated malware er en klassisk type forensics opgave og godt at øve sig på. Jeg vil ikke gennemgå det her, men nogle tips er:

- Åbn koden i en code editor, der hjælper dig med syntax highlighting
- Fix indentation og whitespace først - skab overblik
- Find funktioner og variable, du hurtigt kan se hvad gør og omdøb dem med det samme
- Det vil i sig selv typisk resultere i en bedre forståelse for de resterende funktioner og variable, som nu kan omdøbes osv.
- Hvis du ikke er nervøs for, hvad koden gør, kan du evt. bare køre den - uden nødvendigvis at forstå den - men vær altid forsigtig!

I dette tilfælde bliver det meste af koden brugt på at decode de to første variable med binær data. Resultatet viser sig at være mere JavaScript kode, der indeholder tre forskellige stykker shell code til hhv. Adobe Acrobat version 7, 8 og 9. Shell coden downloader en fil fra en bestemt URL, hvilket vi vil fortsætte med at analysere herunder.


#### Netværk

JavaScript malwaren forsøger at oprette en connection til http://search-network-plus.com/load.php?a=a&st=Internet%20Explorer%206.0&e=2.

Vi kan bruge volatility til at analysere oprettede network connections og tjekke, om dette skete i praksis

Vi kan normalt liste network connetions med `windows.netstat` og `windows.netscan` i Volatility 3, men dette dump er fra en XP maskine, og der kan Volatility 3 ikke bruges. `netscan` fra Volatility2 kan heller ikke, men det kan pluginnet `connections`:

	vol2 -f malpdf.vmem --profile WinXPSP2x86 connections

Hvor mange connections finder du?

<details>
	<summary>Svar</summary>
	11
</details>

Hvad er maskinens lokale IP-adresse?

<details>
	<summary>Svar</summary>
	192.168.0.176
</details>

Processen `AcroRd32.exe` med PID 1752 åbnede en connection, da dokumentet blev åbnet. Til hvilken IP?

<details>
	<summary>Svar</summary>
	212.150.164.203 (se kolonnen Pid i connections output og find remote address for den med Pid 1752)
</details>

#### IE History

Vi ved nu, at PDF'en oprettede en network connection. Dette blev gjort via Internet Explorer, og her har volatility 2 plugginet `iehistory` (ikke lavet til vol 3 endnu):

	vol2 -f malpdf.vmem --profile WinXPSP2x86 iehistory

Hvilken URL blev accessed af `AcroRd32.exe`?

<details>
	<summary>Svar</summary>
	http://search-network-plus.com/load.php?a=a&st=Internet%20Explorer%206.0&e=2
</details>

Vi ser det matcher den URL vi fandt i JavaScript malwaren.

Hvad er navnet på den fil, der blev downloadet?

<details>
	<summary>Svar</summary>
	file.exe
</details>

Vi er stødt på denne URL et par gange, og det kunne tyde på, at http://search-network-plus.com bliver brugt til at hente forskelligt ondsindet kode ned. Vi kan evt. tjekke, om der laves flere connections til den side med en standard `strings`:

	strings malpdf.vmem | grep "^http://search-network-plus.com/" | sort | uniq

`sort | uniq` sørger for kun at vise hvert resultat én gang. Finder du andre connections til `load.php` siden fra før? Hvordan adskiller de sig fra den første?

<details>
	<summary>Svar</summary>
	Der er flere andre connections til siden `load.php`, og de er forskellige i query parameteren <code>e</code>, nemlig <code>e=1</code>, <code>e=2</code> og <code>e=3</code>.
	Et godt bud er, at parameteren <code>e</code> bruges til at vælge, hvilket exploit der skal hentes ned.
</details>

#### Malware Detection + Analysis

Videre analyse kan vise, at malwaren, der blev hentet ned, inficerede processen `winlogon.exe`. Vi så helt i starten den har PID 644. Volatility har pluginnet `malfind`, der kan hjælpe os med automatisk at finde kendt malware og dumpe det. Vi kan køre `malfind` for PID 644 og dumpe resultatet i vores `dump` mappe:

	vol3 -f malpdf.vmem -o dump windows.malfind --pid 644 --dump

Hopper vi ind i `dump` mappen, ser vi, der er lavet en række dumps, navngivet ud fra deres start og slut offset. Prøv at køre `md5sum` på filen med lavest offset (hvilket potentielt er starten på malwaren). Hvad er dens hash?

<details>
	<summary>Svar</summary>
	a9cf2c4d1c1ee7ff7146f0f2d7dfdae7
</details>

Prøv at slå hashet op på VirusTotal. Hvad kalder de fleste AntiVirus-programmer denne malware?

<details>
	<summary>Svar</summary>
	Zbot
</details>

Prøv at slå navnet op på Google. Hvad kan du finde om malwaren? Hvordan passer det med casen?

<details>
	<summary>Svar</summary>
	"Zbot" eller "Zeus Trojan" er designet til at stjæle information, især i forbindelse med online banktransaktioner og logins til banksystemer. Den åbner først en connection til en remote server og downloader en krypteret konfigurationsfil, der indeholder den adresse, den senere skal uploade al information til. Herefter bruger den primært key logging og "Man-in-the-Browser" (MitB) angreb til at extracte finansiel information fra brugeren.
</details>

Vi kan sidst også lige dumpe hukommelsen for denne process:

	vol2 -f malpdf.vmem --profile WinXPSP2x86 memdump -p 644 --dump-dir dump

og se om vi kan finde noget der peger i retning af bankproblemer:

	strings dump/644.dmp | grep "http.*bank.*"

Hvilken URL finder du?

<details>
	<summary>Svar</summary>
	https://onlineeast#.bankofamerica.com/cgi-bin/ias/*/GotoWelcome
</details>

#### Konklusion

Vi har nu konkluderet følgende:

- Brugeren har klikket på følgende link i en e-mail: http://search-network-plus.com/cache/PDF.php?st=Internet%20Explorer%206.0
- Dette har hentet en PDF ned, som indeholdt ondsindet JavaScript kode
- Koden kører noget shellcode i Adobe Acrobat Reader, der opretter en connection til http://search-network-plus.com/load.php?a=a&st=Internet%20Explorer%206.0&e=2 og henter filen `file.exe`
- Denne fil inficerede processen `winlogon.exe` med malwaren `Zbot` (eller `Zeus`).
- Malwaren er designet til at opsnappe finansiel information og sende det tilbage til attackeren

Det er et eksempel på, hvordan man nogenlunde systematisk kan analysere et angreb. Der er stadig mere at undersøge, bl.a. den konfigurationsfil malwaren først henter og den laver også en ændring i Windows registry for at persiste ved genstart af computeren - men det er lidt out of scope her.

#### Password Hashes - Privilege Escalation

Normalt er det en rigtig god idé også at lave et extract af password hashes og forsøge at cracke dem. Det er ikke direkte relevant for casen her, men er tit rigtig vigtigt, så lad os prøve det alligevel på samme dump.

Når man gemmer et password er det sikrest først at hashe det. En hash-funktion er en envejs-funktion, der laver en hvilken som helst tekst om til et "hash" af fast længde. Idéen er, man ikke kan gå tilbage fra hashet til den oprindelige tekst, så det er sikkert at gemme hashet i stedet for passwordet.

Når en bruger logger ind, hasher man bare deres input og sammenligner med det gemte hash. Er de ens, kan man regne med det var det samme password.

Et hash kan ikke revertes, men det kan potentielt crackes, og hvis det lykkes en attacker, kan de få adgang til en bruger med flere privilegier (privilege escalation). Lige i denne case blev det ikke brugt, men lad os alligevel se, hvad vi selv kan finde frem til.

Vi kan udtrække password hashes fra dumpet med

	vol3 -f malpdf.vmem windows.hashdump

Hvilke brugere finder du?

<details>
	<summary>Svar</summary>
	Administrator, Guest, HelpAssistant og SUPPORT_388945a0
</details>

Hvad er `nthash` for Administrator brugeren?

<details>
	<summary>Svar</summary>
	8846f7eaee8fb117ad06bdd830b7586c
</details>

Dette hash kan potentielt crackes. Det gøres ved at hashe en masse passwords og sammenligne resultatet med vores hash. Det kan gøres med programmet "John The Ripper", der følger med Kali (kør `john` fra terminalen), men man kan også gøre det online, bl.a. på https://crackstation.net/

Prøv at indtaste administrator passwordet der. Kan du cracke det? Hvad er passwordet?

<details>
	<summary>Svar</summary>
	Administratoren har bare valgt passwordet "password"
</details>

En attacker med adgang til denne maskine ville altså meget nemt kunne lave privilege escalation og få admin rights.
