# File Analysis Solutions

## Exercise 1 - hidden in plain sight

Søg efter flaget med `grep`:

	grep -o "DDC{.*}" lipsum.txt

Her søger vi med en regular expression, hvor punktum betyder "en hvilken som helst karakter" og * betyder "0 eller flere". Så den finder alle tekststrenge, der starter med "DDC{" og slutter med "}"

	DDC{how_did_you_find_me???}


## Exercise 2 - docx archive?

Åbn `empty.docx` i Word, LibreOffice, e.l. Filen er tilsyneladende tom - men der gemmer sig et flag. Teksten er sat til hvid og kan derfor ikke ses på baggrunden. Marker al tekst med `Ctrl+A` og giv det en anden farve, så flaget kommer frem:

	DDC{camouflage_flag}

DOCX-filer (og flere andre Office filer) er faktisk bare ZIP-arkiver, der indeholder en række XML-filer. Prøv at unzippe dokumentet (omdøb evt. først og sæt extension til `.zip`).

Word-arkivet indeholder den gemte fil `flag.txt` med flaget:

	DDC{vent..._så_alle_word_filer_er_bare_ZIP_i_forklædning??? Gælder det mon også PowerPoint? Hvad med Excel? :O}


## Exercise 3 - ghost data

Første flag er blevet direkte gemt som ASCII-tekst i filen. Brug `strings` til at finde ASCII-strenge og pipe outputtet til `grep` for at lede efter flaget:

	strings ghost.png | grep -o "DDC{.*}"

Det giver flaget `DDC{strings|grep_is_your_friend}`.

I filen gemmer sig også et ekstra PNG-billede, vi kan extracte med file carving teknikker. Både `foremost` og `binwalk` finder den ekstra billedefil og `foremost` extracter den også automatisk.

`binwalk -e` bruges til at extracte kendte filtyper automatisk, men virker ikke på denne fil. Når det sker, kan `binwalk --dd` bruges til mere manuel extraction:

	binwalk --dd=<type[:ext[:cmd]]>

Kommandoen extracter alle filer af typen `type`, giver dem extension `ext` og kører kommandoen `cmd` på hver. F.eks. kan vi køre

	binwalk --dd="png:png" ghost.png

hvilket extracter alle PNG-filer og giver dem extension ".png". `binwalk --dd=".*"` extracter alt `binwalk` finder (false positives er sandsynligt).

Flaget står på billedet:

	ABCTF{b1nw4lk_is_us3ful}

## Exercise 4 - manual extraction

Følg gennemgangen fra selve opgaven. Det start offset der findes med `bgrep` er `0x1388`, altså `5000` i decimal. Vi finder slut offset med

	grep -a -b -o "IEND" datablob

der findes ved offset 10689. PNG-traileren er 8 bytes lang, så vi lægger 8 til det offset for at få slut offset på billedet. Antal bytes vi vil extracte er nu

	10689 + 8 - 5000 = 5697

Kører vi kommandoen

	dd if=datablob of=output.png bs=1 skip=5000 count=5697

får vi extractet 5697 bytes fra offset 5000 og frem - hvilket lige præcis er PNG-filen. Billedet indeholder flaget

	DDC{manual_dd_extraction}

### Exercise 5 - psyduck

Åben `psyduck.png` i `stegsolve` og klik igennem farvekanalerne og bit planes.

Flaget er delt op i tre:
1. Red plane 0: `DDC{rød_som_rosen`
2. Green plane 1: `_grøn_som_græs_`
3. Blue plane 4: `blå_som_havet}`

Fuldt flag: `DDC{rød_som_rosen_grøn_som_græs_blå_som_havet}`

Læg mærke til, at sidste del var gemt helt oppe i plane 4, altså den 4. mest signifikante bit. Ser du grundigt efter, vil du på det originale billede også svagt kunne skimte noget gul tekst - det kunne man ikke, hvis det var gemt i en af de mindst signifikante bits.

### Exercise 6 - pretty cat

Vi kan bruge toolet `zsteg` til at finde skjult tekstdata i et PNG-billede. Køres `zsteg pretty_cat.png` får vi en række false positives, men også et enkelt reelt match:

	b1,rgb,lsb,xy       .. text: "Well done, you managed to use the classic LSB method. Now you know that there's nothing in the sea this fish would fear. Other fish run from bigger things. That's their instinct. But this fish doesn't run from anything. He doesn't fear. Here is your flag: "

Første del fortæller os, at dataen er gemt i bit plane 1 i alle tre RGB-kanaler, altså LSB. Vi får dog ikke hele dataen, `zsteg` giver os kun et preview. Det skyldes, at `zsteg` som default kun tjekker de første 256 bytes med hver mulig kombination, da ekstra data typisk er skjult i starten, og det ellers vil tage lang tid at køre på store filer.

Et muligt fix er at give `zsteg` flaget `-l N`, hvor `N` er den limit, man vil køre `zsteg` med. Brug 0 for at fjerne limit helt. Vi kan altså f.eks. få flaget med ved at forøge limit til 400:

	zsteg -l 400 pretty_cat.png

En anden option er at bruge `-E NAME` til at extracte data fra filen. `NAME` er den kombination af kanaler, planer, osv., man vil bruge til at lave et extract - i vores tilfælde `b1,rgb,lsb,xy`, hvor vi fandt dataen. Resultatet printes bare til skærmen, men vi kan redirecte til en fil i stedet:

	zsteg -E b1,rgb,lsb,xy pretty_cat.png > output.txt

Denne option tjekker ikke, om dataen er ASCII tekst, men extracter bare ud fra det givne `NAME`, så i vores tilfælde extracter den meget data efter teksten er slut. Vi kan se starten af filen med f.eks.:

	less output.txt

eller igen bruge `strings | grep` og se flaget: `shkCTF{Y0u_foUnD_m3_thr0ugH_LSB_6a5e99dfacf793e27a}`

### Exercise 7 - Hackerman

Dataen i billedet er skjult med `steghide`, men vi kender ikke passwordet. Vi kan prøvet at bruteforce det med `stegseek`:

	stegseek hackerman.jpg

`stegseek` skal bruge en wordlist med passwords den kan tjekke. Den leder efter standardlisten `rockyou.txt`, der følger med Kali, men som du potentielt ellers ikke har liggende. Der ligger dog en kopi af `rockyou.txt` i denne mappe i ZIP-filen `rockyou.zip`, så du kan extracte den og køre

	stegseek hackerman.jpg rockyou.txt

for at bruge den wordlist. Det finder passwordet "almost" og extracter resultatet, som er "SFRCezN2MWxfYzBycH0=". Det er base64 encoded, og vi kan decode til

	HTB{3v1l_c0rp}

Tip: Du kan finde andre gode wordlists her: https://github.com/danielmiessler/SecLists. `rockyou.txt` er den mest populære password list, men der findes mange andre til mere specifikke cases, eller med f.eks. mest brugte usernames.