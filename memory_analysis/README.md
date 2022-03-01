# Malicious PDF

## Case

Company X har kontaktet dig og bedt dig foretage en forensics analyse i forbindelse med en nylig hændelse. En af deres medarbejdere modtog en e-mail fra en anden medarbejder med et link til en PDF-fil. Ved åbning af filen lagde medarbejderen ikke mærke til noget særligt, men de har for nylig haft mistænkelig aktivitet på deres bankkonto.

Den nuværende teori er, at brugeren har modtaget en e-mail med en URL til et forfalsket PDF-dokument. Ved åbning af dokumentet i Acrobat Reader blev et ondsindet JavaScript program kørt, der overtog ofrets system.

Company X har taget et memory dump af medarbejderens maskine og har spurgt dig om at analysere den virtuelle hukommelse og give dem svar på deres spørgsmål.

## Intro

Unzip `malpdf.zip`. Prøv at analysere Windows dumpet `malpdf.vmem` med `volatility 2` og/eller `volatility 3` og svar på de følgende spørgsmål. Jeg antager her, at dit volatility 2 program er navngivet `vol2` og volatility 3 `vol3` - udskift ellers med din egen path.

Jeg vil anbefale I laver en `evidence` mappe og gemmer resultatet fra alle scanninger der, så de kun skal køres én gang.

## Spørgsmål

#### Profil

Volatility kræver en *profil* af dumpet, for at kunne forstå og analysere det. Det klarer volatility 3 selv, men i volatility 2 skal profilen først findes manuelt.

Til det kan man bruge `imageinfo` og `kdbgscan`. `imageinfo` giver forslag til profiler ud fra dumpet og kører hurtigere en `kdbgscan`, men `kdbgscan` er beregnet til at være mere pålideligt og forsøger at finde den rigtige profil.

Prøv at køre `vol2 -f malpdf.vmem imageinfo`. Hvilke profiler foreslår den?

<details>
	<summary>Svar</summary>
	Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86
</details>

Kør nu `vol2 -f malpdf.vmem kdbgscan`. Hvad finder den?

<details>
	<summary>Svar</summary>
	kdbgscan finder i dette tilfælde de samme to profiler. Her er det vigtigt at tjekke under PsActiveProcessHead og PsLoadedModuleList, om den har fundet nogle relaterede profiler eller om der står (0 processes) og (0 modules) - så er profilen nok en false positive
</details>

<details>
	<summary>Konklusion</summary>
	Begge profiler bør være fine, og du skal herefter køre alle plugins med <code>vol2 -f malpdf.vmem --profile WinXPSP2x86 PLUGIN_NAVN</code>
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

Du kan tjekke alle commands brugeren har kørt med

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

Hvilken process har PID 644, og hvornår blev den starter?

<details>
	<summary>Svar</summary>
	Det er winlogon.exe med CreateTime 2010-02-26 03:34:04.000000
</details>

Vi så tidligere `AcroRd32.exe` havde process ID 1752 - men hvad er dens parent PID (PPID)? Og hvilken process er det?

<details>
	<summary>Svar</summary>
	PPID er 888 og er firefox.exe
</details>

#### Memory Dump

Der er altså tilsyneladende hentet et PDF-dokument i Firefox, som er blevet åbnet i Adobe Acrobat Reader. Det ville være interessant at se, hvor dokumentet var hentet fra.

Vi kan printe al memory fra en bestemt process (i det her tilfælde `firefox.exe` med PID 888). Start med at lave en `dump` folder:

	mkdir dump

Du kan køre `memdump` for at dumpe for en specifik process:
	
	vol2 -f malpdf.vmem --profile WinXPSP2x86 memdump -p 888 --dump-dir dump

Vi kan hoppe ind i mappen `dump` og køre vores klassiske `strings | grep` kommando for at lede efter en URL med en PDF:

	strings 888.dmp | grep -i "http://.*pdf"

Hvilken URL finder du?

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

Muligvis er én af de PDF-filer malicious og indeholder malware. Vi kan forsøge at slå deres hash op på VirusTotal. Hashet kan laves med

	md5sum *

Prøv at copy paste de forskellige hashes ind i https://www.virustotal.com/gui/home/search. Finder du noget på nogle af dem?

<details>
	<summary>Svar</summary>
	Sidste PDF med hash f32aa81676c7391528afe08e437765cc er flagged som malicious
</details>

Den ondsindede PDF detectes af flere AntiViruser som et PDF exploit, men flere nævner også JS, altså JavaScript. Prøv at køre en `strings` på PDF'en og `grep` efter JavaScript. Giver det noget?

<details>
	<summary>Svar</summary>
	Ja, linjen <code>&lt;&lt;&sol;S&sol;JavaScript&sol;JS 1054 0 R&gt;&gt;</code> bliver fundet, så PDF-filen kører sandsynligvis noget ondsindet JavaScript kode.
</details>

Det JavaScript kan extractes med toolet "pdf-parser" fra "pdftools", men det springer vi lige over her - slå det gerne op, hvis du vil lære mere!

#### Netværk

Vi springer lige over at analysere JavaScript koden, men der kan man se, at PDF-en laver en network connection, så lad os analysere dem også.

Vi kan normalt liste netværks connetions med `windows.netstat` og `windows.netscan` i Volatility 3, men dette dump er fra en XP maskine, og der kan Volatility 3 ikke bruges. `netscan` fra Volatility2 kan heller ikke, men det kan pluginnet `connection`:

	vol2 -f malpdf.vmem --profile WinXPSP2x86 connections

Hvor mange connections finder vi?

<details>
	<summary>Svar</summary>
	11
</details>

Hvad er maskinens lokale IP-adresse?

<details>
	<summary>Svar</summary>
	192.168.0.176
</details>

Processen `AcroRd32.exe` med PID 1752 åbnede en netværksconnection, da dokumentet blev åbnet. Til hvilken IP?

<details>
	<summary>Svar</summary>
	Se kolonnen Pid i connections output. Her er remote address 212.150.164.203
</details>

#### IE History

Lad os se nærmere på den connection, PDF'en åbnede. Her kommer volatility3 lidt til kort, men volatility2 har plugginet `iehistory`:

	vol2 -f malpdf.vmem --profile WinXPSP2x86 iehistory

Hvilken URL blev accessed af `AcroRd32.exe`?

<details>
	<summary>Svar</summary>
	http://search-network-plus.com/load.php?a=a&st=Internet%20Explorer%206.0&e=2
</details>

Hvad er navnet på den fil, der blev downloadet?

<details>
	<summary>Svar</summary>
	file.exe
</details>

Her vil man igen kunne analysere videre på malwaren, men jeg har lige valgt at stoppe selve casen her - kig gerne videre!

#### Password Hashes

Urelateret til selve casen, men meget relevant normalt!

Når man gemmer et password er det sikrest først at hashe det. En hash-funktion er en envejs-funktion, der laver en hvilken som helst tekst om til et "hash" af fast længde. Idéen er, man ikke kan gå tilbage fra hashet til den oprindelige tekst, så det er sikkert at gemme hashet i stedet for passwordet.

Når en bruger så logger ind, hasher man bare deres input og sammenligner med det gemte hash. Er de ens, kan man regne med det var det samme password.

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

Hashes kan ikke revertes, men kan crackes ved at hashe en masse passwords og sammenligne resultatet. Det kan gøres med programmet "John The Ripper", der følger med Kali (kør `john` fra terminalen), men man kan også gøre det online, bl.a. på https://crackstation.net/

Prøv at indtaste administrator passwordet der. Kan du cracke det? Hvad er passwordet?

<details>
	<summary>Svar</summary>
	Administratoren har bare valgt passwordet "password"
</details>
