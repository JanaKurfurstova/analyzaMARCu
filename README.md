# Kontrola dat pomocí OpenRefine

## Spouštění OpenRefine

Je třeba mít nainstalovanou Javu: [https://java.com/en/download/manual.jsp](https://java.com/en/download/manual.jsp)

Downolad: [https://openrefine.org/download.html](https://openrefine.org/download.html)

### Alokace RAM - čím víc, tím líp
* Windows: Před prvním spuštěním `openrefine.exe` je třeba navýšit v `openrefine.l4j.ini` max memory memory heap size. Pro začátek -Xmx4096M (pro větší knihovny raději 6144, 8192, 10240). 
* Linux: `.refine -m 10240M`

OpenRefine pak jede zde: [http://127.0.0.1:3333](http://127.0.0.1:3333)

**NEJLÉPE POUŠTĚT VE FIREFOXU.**

## Stahování exportů

### Windows

Použít PSCP.

**"KNIHOVNA" vždy přepsat zkratkou knihovny (mkkh, mkmt apod.)**

Soubory nepřejmenováváme - musí se tak jmenovat kvůli skriptům.

Jaká je cesta ke klíči?
* Putty > Connection > SSH > Auth > Private key for authentication (\*.ppk)
Napřed někdo musí přidat na back veřejný klíč. Je třeba mu ho poslat:
* Puttygen > Load > \*.ppk > passphrase > zkopírovat public key (ssh-rsa…)

### Linux

Normálně stáhnout z backu přes SCP.

## Otevření exportů

Ve Windowsech je třeba při otevírání specifikovat kódování UTF-8.

Je třeba mít otevřené všechny 4 soubory.

## Aplikace skriptů 

**Undo/Redo > Apply**

Do otevřené okna vložit JSON.

### 1. KNIHOVNA_nezdeduplikovane txt

Vložíme obsah [OpenRefineApplyFirstOnNezdedup.json](OpenRefineApplyFirstOnNezdedup.json)

### 2. KNIHOVNA txt

Vložíme obsah [OpenRefineApplySecondOnAll.json](OpenRefineApplySecondOnAll.json)

Tohle trvá v závislosti na množství dat a alokované paměti.

Až to doběhne, můžeme zavřít ostatní tři otevřené soubory.

## Samotná kontrola dat

Jdeme podle Googlového formuláře s využitím kontrolních sloupců.

### Exporty pro knihovny
* **Export**
* **Custom tabular exporter**
* **De-select All**
* odfajfkovat **Output empty row**
* zaklikat prvních 5 sloupců
* Download

### Export pro kontrolu duplicit
* faseta na neprázdné **interdup** a invert TRUE **no996** 
* **Export**
* **Custom tabular exporter**
* **De-select All**
* odfajfkovat **Output empty row**
* odfajfkovat **Output column headers**
* zaklikat sloupce:
    * OAI
    * autor
    * titul
    * rok
    * ISN
    * interdup
    * format
    * 250a
    * isbn
    * cnb
    * title
    * 245aMultiTest

Na export se pak použije tabulka **interdup**.

Tabulka má list s vysvětlivkami.

Vhodné je si tabulku zkopírovat pro každou knihovnu a původní verzi nechat být.

Tabulka je předdělaná pro 10000 řádků. Když by jich bylo víc, vzorce se bohužel automaticky nerozkopírují až na konec tabulky. To je pak třeba udělat ručně, ale nestává se to moc často.

Nejlepší je podívat se předem, kolik řádků má export. Když ještě před jeho nakopírováním do tabulky odmažeme přebytečné řádky, netrvají pak výpočty tak dlouho.

### Kontrolní sloupce, jak jdou za sebou:

* **OAI**
* **autor** - pro výstupy
* **titul** - pro výstupy
* **rok** - pro výstupy, klíč
* **ISN** - pro výstupy
* **interdup** - číslo sloučeného záznamu v době exportu
* **nezdedup** - nezdeduplikované záznamy
* **040a** - sigla původce záznamu
* **bl_language** - jazykový klíč
* **no996** - nečlánkové záznamy bez jednotek jsou TRUE
* **996s** - pro kontrolu zkratek dostupnosti
* **format** - klíč pro typ dokumentu
* **LDRdoctype** - LDR|06 + LDR|07
* **245h** - druh dokumentu z 245$h
* **300doctype** - písmena z 300$a
* **titulNosic** - nezdeduplikované knihy s jednotkou, které mají v názvu CD, MC, LP nebo DVD
* **neNoty** - hudebniny s jednotkou zapsané jako textové dokumenty, viz [more_info.md](more_info.md)
* **neMapy** - kartografické dokumenty s jednotkou zapsané jako textové dokumenty, viz [more_info.md](more_info.md)
* **genre** - zobecněný žánr, viz [more_info.md](more_info.md) 
* **oldORgrey** - nezdeduplikované knihy s jednotkou do r. 1890 nebo novější, které mohou být regionální či šedá literatura, viz [more_info.md](more_info.md)
* **displaced245c** - nezdeduplikované české, slovenské a bezjazykové záznamy s jednotkou, kde je podezření na výskyt údajů patřících do 245$c v 245$abnp, , viz [more_info.md](more_info.md)
* **authkeyCheck** - u záznamů s hlavním personálním autorem (s jednotkami nebo analytické) rozlišuje: A0D0 - nezdeduplikované bez autority, A0D1 - zdeduplikované bez autority, A1D0 - nezdeduplikované s autoritou, A1D1 - zdeduplikované s autoritou
* **yearMismatch** - nezdeduplikované neanalytické záznamy s jednotkami a nesedícími roky v 008 a 26X
* **noYear** - nezdeduplilované záznamy (s jednotkou nebo analytické), které nemají ani přibližné datum vydání
* **invalidISN** - záznamy s jedntkami a nějakým číselným údajem zadaný jako ISBN nebo ISSN, ale neutvořil se klíč
* **noISN** - nezdeduplikované knihy od 1989, u které nemají ISBN a nebyl u nich rozpoznán jiný problém, viz [more_info.md](more_info.md)
* **cnbCheck** - označení zdeduplikovaných a nezdeduplikovaných AV záznamů s EANem a jednotkami
* **eanCheck** - označení zdeduplikovaných a nezdeduplikovaných záznamů s čČNB a jednotkami
* **pnCheck** - označení rozpoznatelných nakladatelských čísel (OK) v nezdeduplikovaných záznamech audiodokumentů s jednotkami, u nerozpoznatelných vypíše první indikátor
* **773check** - seznam podpolí 773 u nezdeduplikovaných článků
* **vicedil** - 245 obsahuje $p nebo $n.
* **zCyklu** - cykly
* **bl_publisher** - nakladatelský klíč
* **250a** - vydání nebo "1", viz [more_info.md](more_info.md)
* **isbn** - klíč
* **issn** - klíč
* **ismn** - klíč
* **cnb** - klíč
* **ean** - klíč
* **publisher_number** - klíč
* **title** - klíč
* **author_string** - klíč
* **author_auth_key** - klíč
* **245aMultiTest** - pomocný sloupec pro kontrolu duplicit
* **245aSerialTest** - pomocný sloupec pro hledání monograficky zapsaných seriálů, viz [more_info.md](more_info.md)
* **zbytek** - nezdeduplikované z jiných důvodů, viz [more_info.md](more_info.md)

### Když vidíme problém, na který není udělaný sloupec

Můžeme vytvářet vlastní sloupce, které zůstanou v projektu i po restartu.

Můžeme vytvářet fasety, které ale v projektu nezůstanou po restartu. Pokud si ale předtím uložíme **Permalink**, lze je aplikovat znovu bez ručního klikání.

## Custom text facet na neopakovatelné podpole

`value.match(/.*\$**a**([^\$]+).*/)[0]`

## Custom text facet na opakovatelné podpole

`replace(replace(join(sort(value.find(/\$**a**[^\$]+/)),""),/\$./,";"),/^\;/,"")`

## Custom text facet na konkrétní znakové pozice

`value.substring(odPočítánoOdNuly,doPočítánoOdJedné)`