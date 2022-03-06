# File Analysis Solutions

## Exercise 1 - encodings

Strengen er binær data og vi decoder først fra binary til ASCII:

	50 41 54 58 50 50 39 61 98 51 43 62 98 110 116 50 68 108 85 53 49 42 65 71 52 43 63 41 47 35 50 41 117 100 56 49 42 65 65 53 43 63 33 91 79 50 68 72 61 51 50 93 115 110 54 43 63 33 100 82 50 68 90 73 51 48 100 38 56 52 43 62 117 50 37 50 68 81 67 51 50 93 116 37 61 43 62 117 44 35 50 68 99 79 53 50 93 116 34 60 43 63 33 94 80 50 41 108 94 55 49 69 92 83 52 43 62 117 50 37 50 68 108 85 55 51 36 58 34 50 43 62 91 79

Dette er igen en numerisk encoding - ligner mest af alt almindelig decimal. Decode fra decimal:

	2)6:22'=b3+>bnt2DlU51*AG4+?)/#2)ud81*AA5+?![O2DH=32]sn6+?!dR2DZI30d&84+>u2%2DQC32]t%=+>u,#2DcO52]t"<+?!^P2)l^71E\S4+>u2%2DlU73$:"2+>[O

Dette kan være sværere at genkende, men er Base85, vi igen kan decode til ASCII:

	52 45 52 44 65 32 56 75 59 32 39 6b 61 57 35 6e 63 31 39 68 62 47 78 66 64 47 68 6c 58 33 64 68 65 58 30 3d

Denne streng er hex encoded (indeholder tal og også bogstaver fra a-f). Decode fra hex:

	RERDe2VuY29kaW5nc19hbGxfdGhlX3dheX0=

Læg mærke til = i slutningen af strengen - dette er base64 og vi decoder en sidste gang:

	DDC{encodings_all_the_way}

Fuld CyberChef recipe: https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)From_Decimal('Space',false)From_Base85('!-u')From_Hex('Auto')From_Base64('A-Za-z0-9%252B/%253D',true)

## Exercise 2 - file extension

Filen har extension `.mp4` og vil derfor automatisk blive forsøgt åbnet i et lyd/video-program. Men filen er slet ikke en MP4-fil. Programmet vil derfor enten give en fejlbesked, eller endnu værre bare ikke kunne afspille filen, men ikke give en fejl.

`file funny_video.mp4` viser, at filen faktisk er et PNG-billede. Tjek evt. også indhold med en hex editor. Her kan du se ASCII strengen "PNG" i starten.

Åbn filen direkte fra et billedeprogram eller omdøb den til `funny_video.png` og åbn som normalt, så åbnes den korrekt. Her ses flaget `DDC{stol_aldrig_på_extensions!}`

Tip: Mange binære filer er store, og hvis du kører `xxd` eller `strings` får du hele outputtet på én gang. Du kan pipe til programmet `less`, e.g.

	strings fil.png | less
	xxd fil.png | less

for kun at se én side af gangen. Brug pil op/ned for at gå en linje op/ned ad gangen. Brug SPACE for at gå en side frem. HOME/END for at gå til start/slut. Tryk q for at afslutte.

## Exercise 3 - magic numbers

1. Brug linket med listen over filsignaturer og søg på PDF.
PDF-filer har magic number `25 50 44 46`, som i ASCII er `%PDF`. Dvs. PDF-filer starter med de fire bytes.

2. Søg på PNG. Traileren er i hex `49 45 4E 44 AE 42 60 82` - sådan slutter PNG-filer. De første fire bytes er i ASCII `IEND`.

3. Køres `file what_filetype` får man bare svaret `data`, fordi `file` ikke genkender signaturen. Filens signatur er `38 7A BC AF 27 11` og den første og sidste byte er ændret. Hvis man søger på nogle par af bytes - f.eks. `38 7A`, `7A BC`, `BC AF` - finder man hurtigt et match på dem, der ikke indeholder en corrupted byte, og man finder frem til filtypen `7Z`, en 7-Zip komprimeret fil. Den har signaturen `37 7A BC AF 27 1C`. Åbn en hex editor og ændr første byte fra `38 -> 37` og sidste fra `11 -> 1C`, så signaturen er rigtig. Køres `file` igen, får man nu den rigtige filtype identificeret. Extract indholder `flag.txt` med `7z e what_filetype` for at få flaget `DDC{don't_assume_file_knows_the_answer!}`.

## Exercise 4 - metadata

Se filens metadata med `exiftool so_meta.jpg` og find feltet "XP Comment", hvor flaget er: `DDC{metadata_can_cary_valuable_info}`

Hvis du ved, du kigger efter et flag, kan det være en fordel at bruge `grep` til at lede efter flag formatet. Du kan pipe output fra en hvilken som helst kommando til `grep` for at søge i outputtet, f.eks.

	exiftool so_meta.jpg | grep "DDC{.*}"

Tip: `strings` bruges tit til at lede efter tekst i en binær fil, men du kan ikke regne med, at al metadata er encoded som almindelig ASCII / UTF-8 tekst og printes med `strings`. F.eks. finder `strings so_meta.jpg` ikke flaget, fordi "XP Comment" er encoded som UTF-16. Det kan findes med `strings` ved at ændre encoding:

	strings -e b so_meta.jpg

men det er en god idé at kigge metadata igennem eksplicit med `exiftool` og ikke kun regne med `strings`.

## Exercise 5 - corrupted file

Filen er en PNG-fil, der er blevet corrupted fire forskellige steder.

Køres `file corrupted.png` får vi at vide, at filen bare er `data`, så der virker til at være noget galt med filens signatur.

Vi åbner filen i en hex editor og ser på filens magic number. https://www.garykessler.net/library/file_sigs.html viser det skal være `89 50 4E 47 0D 0A 1A 0A`, tredje byte er `4F` i stedet, så der står "POG" i stedet for "PNG". Det fikser vi først.

`file` kan stadig ikke genkende filen, men vi kan bruge `pngcheck`, der fortæller os at "first chunk must be IHDR". PNG-filer består af en række forskellige chunks, bl.a. en header chunk, der indikeres af IHDR og data chunks, der indikeres med IDAT. Ser vi på filens bytes igen, ser vi der næsten i starten står "OHDR". Det retter vi til "IHDR", så chunk typen passer.

Vi prøver `pngcheck` igen, der giver os fejlen "invalid IHDR length". Tjekker vi [PNG specification](http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html#PNG-file-signature) under chunk layout, kan vi se, at hver chunk består af fire dele:
1. Chunk length (4-byte værdi)
2. Chunk type (4-byte værdi)
3. Chunk data (selve chunkens indhold)
4. CRC (4-byte værdi, der indikerer om chunken er blevet ændret)

Det kunne altså tyde på, at chunk length er blevet sat forkert. Det er værdien lige før chunk typen "IHDR", altså de fire bytes `00 00 0B AD` (`BAD` for at hinte, at de blevet corrupted). Men hvad skal længden være? Det kan vi se ved at kigge på IHDR chunk specifikationen i dokumentet igen: http://www.libpng.org/pub/png/spec/1.2/PNG-Chunks.html#C.IHDR. Her ses, at IHDR chunken indeholder 7 forskellige felter, der i alt fylder 13 bytes. Vi skal altså ændre chunk length til 13 - som i hex er `0D`. Vi sætter length til `00 00 00 0D`.

Kører vi `file` nu, får vi output

    PNG image data, 1400 x 1400, 8-bit/color RGBA, non-interlaced

Vi kan prøve at åbne filen, men der er stadig noget galt. `pngcheck` fortæller os nu:

    CRC error in chunk IHDR (computed 1a0bb537, expected 1a0bb573)

Der er altså en fejl i chunkens CRC. Den burde være `1A 0B B5 37`, men er `1A 0B B5 73`. Vi kan enten finde de bytes manuelt i vores hex editor eller søge på dem. Alternativt kan vi tjekke PNG specs igen, og se at CRCen er gemt lige efter chunkens data, som vi ved er 13 bytes lang, så vi kan tælle 13 bytes frem fra efter "IHDR", hvor vi finder `1A 0B B5 73`. Vi retter sidste byte til `37` og gemmer filen.

Nu kan filen åbnes og vi får flaget

    DDC{fixing_corrupted_PNGs!}