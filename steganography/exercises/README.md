# File Analysis Exercises

## Tools

- file
- strings
- hexdump / xxd
- grep
- bgrep
- binwalk / foremost
- dd
- stegsolve
- zsteg
- steghide
- stegseek

## Exercise 1 - hidden in plain sight

Kan du finde et flag i filen `lipsum.txt`? Prøv at bruge `grep` i stedet for bare at søge. `grep` returnerer som default hele den linje, der indeholder matchet - brug `grep -o` for *kun* at returnere matchet.

<details>
	<summary>Hint</summary>
	<code>grep PATTERN lipsum.txt</code>
</details>


## Exercise 2 - docx archive?

Der gemmer sig to flag i filen `empty.docx`. Prøv at undersøge, hvad DOCX-formatet i virkeligheden er!

<details>
	<summary>Hint 1</summary>
	Kan du finde et flag i selve dokumentet, når du åbner det?
</details>

<details>
	<summary>Hint 2</summary>
	Vidste du, at DOCX-filer (og andre Office filtyper) faktisk bare er ZIP-arkiver?
</details>

<details>
	<summary>Hint 3</summary>
	Prøv at unzippe word-dokumentet (evt. skift extension til .zip først) - kan du finde noget i de udpakkede filer?
</details>

## Exercise 3 - ghost data

Der gemmer sig lidt forskelligt i filen `ghost.png` - kan du finde to flag?

<details>
	<summary>Hint 1</summary>
	Måske gemmer der sig et flag direkte som tekst i den binære fil et sted? Hvordan ville du lede efter det?
</details>

<details>
	<summary>Hint 2</summary>
	Prøv at køre <code>strings</code> på filen og søg efter flaget med <code>grep</code>
</details>

<details>
	<summary>Hint 3</summary>
	Prøv nogle file carving teknikker, og se, om der gemmer sig en fil mere
</details>

<details>
	<summary>Hint 4</summary>
	<code>binwalk</code> finder en ekstra PNG-fil. <code>binwalk -e</code> virker dog ikke i dette tilfælde. Når det sker, kan man bruge <code>binwalk --dd=".*"</code> for at tvinge <code>binwalk</code> til at extracte alt den finder. Du kan køre <code>file</code> på hver extracted fil for at se, hvad de indeholder, og herefter give dem korrekt extension og åbne.
</details>

## Exercise 4 - manual extraction

Der gemmer sig en PNG-fil i filen `datablob`. Den kan nemt extractes med `binwalk` eller `foremost`, men prøv i stedet en manuel extraction med `dd`. Syntaks:

	dd if=datablob of=output.png bs=1 skip=??? count=???

Prøv evt. selv at finde `skip` og `count`, men ellers er her en mulig fremgangsmåde:

PNG-filens offset (`skip`) kan findes ved at lede efter dens magic number `89 50 4e 47`. Prøv at bruge `bgrep` til det:

	bgrep 89504e47 datablob

Offset outputtes i hex, så du skal lige konvertere til decimal, før du bruger det i `skip`.

Antal bytes (`count`) kan udregnes ved at finde PNG-filens slutning og trække start offset fra. Igen kan `bgrep` bruges til at lede efter PNG-filens trailer ("IEND"), men når du bare leder efter ASCII tekst, kan du bare bruge `grep`:

	grep -a -b -o IEND datablob

De tre flag gør følgende:
* -a: få vist match fra en binary fil som tekst
* -b: print offset på match
* -o: match *kun* på søgningen, ikke hele linjen, der indeholder den (vigtigt for at få korrekt offset)

Du har nu et start offset og slut offset på filen og kan udregne `count`. Husk at lægge trailerens størrelse til for at få den med også:

	count = end_offset - start_offset + 8

Kører du `dd` kommandoen med de værdier, bør den extracte PNG-billedet til `output.png`.

### Exercise 5 - psyduck

Brug `stegsolve` til at gennemse hver bit plane for RGB-kanalerne i billedet `psyduck.png`. Hvad kan du se? I hvilke planer og kanaler ser du det?

### Exercise 6 - pretty cat

Samme teknik er brugt til at gemme data i billedet `pretty_cat.png`, men denne gang er der ikke gemt et billede, men i stedet ASCII tekst. Hvilket tool kan du bruge til at finde teksten? I hvilke kanaler og bits var det gemt?

Toolet finder ASCII-teksten automatisk, men tror ikke flaget er en del af teksten og viser det ikke. Prøv at tjekke toolets help menu og se, om du kan extracte al dataen.

<details>
	<summary>Hint 1</summary>
	Prøv at køre <code>zsteg</code> på billedet
</details>

<details>
	<summary>Hint 2</summary>
	<code>zsteg -E</code> kan bruges til at extracte et fuldt payload fra en given kombination af planer og kanaler
</details>

### Exercise 7 - Hackerman

Er du en ægte hackerman? Se om der skjuler sig noget i `hackerman.jpg` for at bevise det! Jeg tror dog ikke du kommer ret langt uden mit password!

<details>
	<summary>Hint 1</summary>
	Dataen er gemt med <code>steghide</code>. Hvad kan du gøre uden et password?
</details>

<details>
	<summary>Hint 2</summary>
	Prøv at bruteforce passwordet med <code>stegseek</code>
</details>

<details>
	<summary>Hint 3</summary>
	Flaget er base64 encoded
</details>