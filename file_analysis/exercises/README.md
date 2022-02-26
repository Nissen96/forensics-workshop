# File Analysis Exercises

## Tools

- file
- strings
- hexdump / xxd
- hexedit / ghex / HxD / Hex Fiend
- base64
- exiftool
- pngcheck

## Exercise 1 - encodings

Decode følgende:

	001101010011000000100000001101000011000100100000001101010011010000100000001101010011100000100000001101010011000000100000001101010011000000100000001100110011100100100000001101100011000100100000001110010011100000100000001101010011000100100000001101000011001100100000001101100011001000100000001110010011100000100000001100010011000100110000001000000011000100110001001101100010000000110101001100000010000000110110001110000010000000110001001100000011100000100000001110000011010100100000001101010011001100100000001101000011100100100000001101000011001000100000001101100011010100100000001101110011000100100000001101010011001000100000001101000011001100100000001101100011001100100000001101000011000100100000001101000011011100100000001100110011010100100000001101010011000000100000001101000011000100100000001100010011000100110111001000000011000100110000001100000010000000110101001101100010000000110100001110010010000000110100001100100010000000110110001101010010000000110110001101010010000000110101001100110010000000110100001100110010000000110110001100110010000000110011001100110010000000111001001100010010000000110111001110010010000000110101001100000010000000110110001110000010000000110111001100100010000000110110001100010010000000110101001100010010000000110101001100000010000000111001001100110010000000110001001100010011010100100000001100010011000100110000001000000011010100110100001000000011010000110011001000000011011000110011001000000011001100110011001000000011000100110000001100000010000000111000001100100010000000110101001100000010000000110110001110000010000000111001001100000010000000110111001100110010000000110101001100010010000000110100001110000010000000110001001100000011000000100000001100110011100000100000001101010011011000100000001101010011001000100000001101000011001100100000001101100011001000100000001100010011000100110111001000000011010100110000001000000011001100110111001000000011010100110000001000000011011000111000001000000011100000110001001000000011011000110111001000000011010100110001001000000011010100110000001000000011100100110011001000000011000100110001001101100010000000110011001101110010000000110110001100010010000000110100001100110010000000110110001100100010000000110001001100010011011100100000001101000011010000100000001100110011010100100000001101010011000000100000001101100011100000100000001110010011100100100000001101110011100100100000001101010011001100100000001101010011000000100000001110010011001100100000001100010011000100110110001000000011001100110100001000000011011000110000001000000011010000110011001000000011011000110011001000000011001100110011001000000011100100110100001000000011100000110000001000000011010100110000001000000011010000110001001000000011000100110000001110000010000000111001001101000010000000110101001101010010000000110100001110010010000000110110001110010010000000111001001100100010000000111000001100110010000000110101001100100010000000110100001100110010000000110110001100100010000000110001001100010011011100100000001101010011000000100000001100110011011100100000001101010011000000100000001101100011100000100000001100010011000000111000001000000011100000110101001000000011010100110101001000000011010100110001001000000011001100110110001000000011010100111000001000000011001100110100001000000011010100110000001000000011010000110011001000000011011000110010001000000011100100110001001000000011011100111001

<details>
	<summary>Hint 1</summary>
	Decode fra binary
</details>

<details>
	<summary>Hint 2</summary>
	Decode resultat fra decimal
</details>

<details>
	<summary>Hint 3</summary>
	Decode resultat fra base85
</details>

<details>
	<summary>Hint 4</summary>
	Decode resultat fra hex
</details>

<details>
	<summary>Hint 5</summary>
	Decode resultat fra base64
</details>

## Exercise 2 - file extension

Prøv at åbne filen `funny_video.mp4`. Hvad sker der? Hvorfor?

<details>
	<summary>Hint</summary>
	Stol ikke på extension, brug <code>file</code>
</details>

## Exercise 3 - magic numbers

Filer har en signatur - et magic number - der afgør filtypen. `file` bruger det magic number til at identificere filtypen.

En liste over filsignaturer kan findes her: https://www.garykessler.net/library/file_sigs.html

1. Hvad er signaturen for PDF-filer i hex og i ASCII?

<details>
	<summary>Hint</summary>
	Søg på "PDF" i listen over filesignaturer
</details>

---

Nogle filer har udover en magic header også en trailer, der afslutter filen.

2. Hvad er traileren for PNG-filer?

<details>
	<summary>Hint</summary>
	Søg på "PNG" i listen over filesignaturer
</details>

---

3. Filsignaturen for filen `what_filetype` er blevet ændret en smule, så magic bytes ikke længere matcher den faktiske filtype. Prøv at køre `file what_filetype`, og se hvad `file` outputter. Kan du finde den rigtige signatur alligevel og rette den, så filen kan åbnes?

<details>
	<summary>Hint 1</summary>
	Lav et hexdump eller åben filen i en hex editor for at se hvad de første bytes er. Prøv at søge på nogle af dem i listen over filsignaturer og se om du kan finde et "næsten" match.
</details>

<details>
	<summary>Hint 2</summary>
	Filens signatur er efter ændringen <code>38 7A BC AF 27 11</code> og kun første og sidste byte er ændret.
</details>

<details>
	<summary>Hint 3</summary>
	Prøv at bruge en hex editor til at fikse de to ændrede bytes. Hvis du bruger <code>hexedit</code> kan du gemme og lukke filen med <code>Ctrl+X</code>.
</details>

## Exercise 4 - metadata

Når man skal analysere en fil, er det vigtigt ikke kun at kigge på filens indhold, men også metadata - det kan give meget værdifuld information om filen.

Tag et kig på billedet `so_meta.jpg`, og se om du kan finde noget interessant i filens metadata.

<details>
	<summary>Hint 1</summary>
	Brug <code>exiftool</code> til at se filens metadata
</details>

<details>
	<summary>Hint 2</summary>
	Tjek feltet "XP Comment"
</details>

### Exercise 5 - corrupted file

Filen `corrupted.png` virker til at være beskadiget og kan ikke åbnes korrekt. `file` kan heller ikke identificere filtypen. Prøv at bruge PNG specification (http://www.libpng.org/pub/png/spec/1.2/PNG-Contents.html) til at vurdere, hvad der er galt og ret det med en hex editor. Toolet `pngcheck` kan hjælpe til at identificere fejlene.

<details>
	<summary>Hint 1</summary>
	Start med at køre <code>xxd corrupted.png</code> eller åben filen i en hex editor. Er filens magic number rigtigt? Sammenlign med https://www.garykessler.net/library/file_sigs.html (søg på PNG)
</details>

<details>
	<summary>Hint 2</summary>
	Prøv at køre <code>pngcheck</code> på filen, når dens magic byte er blevet fikset - hvad forventer <code>pngcheck</code> at finde? Og hvad finder du i stedet? Fiks det!
</details>


<details>
	<summary>Hint 3</summary>
	Prøv at køre <code>pngcheck</code> på filen nu - får du en anden fejl? Husk at finde hjælp i PNG specification!
</details>

<details>
	<summary>Hint 4</summary>
	<code>pngcheck</code> brokkede sig over IHDR længden. IHDR er en af de forskellige chunk types i PNG filer. <a href="http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html#PNG-file-signature">PNG specs</a> fortæller dig, at hver chunk består af en 4-byte length, så en 4-byte chunk type, chunk data, og til sidst en 4-byte CRC. Det er altså de 4 bytes lige før "IHDR" den er gal med.
</details>

<details>
	<summary>Hint 5</summary>
	IHDR length står lige nu til <code>00 00 0B AD</code>, men hvad skal den faktisk være? Igen bruger vi PNG specification og finder siden om <a href="http://www.libpng.org/pub/png/spec/1.2/PNG-Chunks.html#C.IHDR">IHDR Image Header</a> under "Chunk Specifications".

	Hvad består IHDR chunken af? Hvor mange bytes i alt? Hvad skal IHDR length så erstattes med?
</details>

<details>
	<summary>Hint 6</summary>
	Nu er magic bytes, IHDR chunk type og IHDR chunk length fikset. Prøv at køre <code>pngcheck</code> igen. Hvad får du at vide? Hvor skal du rette fejlen? Prøv enten at søge på de forkerte bytes i din hex editor eller brug PNG specification til at se, hvor de er.
</details>