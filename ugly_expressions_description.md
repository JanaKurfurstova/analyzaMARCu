# Popis funkce složitějších výrazů a prostor pro návrhy jejich vylepšení:
Proto že OpenRefine nemá rád komentáře v JSONu.

## 6180: Konsolidace českých hodnot v 655$

`if(contains(replace(value,/\\s\\(.*\\)/,\"\"),/\\s/),replace(replace(replace(value,/\\s\\(.*\\)/,\"\"),/[\\-\\p{L}]+([áéůý]|[cč]í|(?<!(vydá|cviče|jedná|sděle|naříze))ní|(?<!pomů)cky|(?<!(de|ti))sky|fi|antasy)([^\\p{L}]|$)/,\"\"),/^a\\s/,\"\").match(/^(\\p{L}+).*/)[0],replace(value,/\\s\\(.*\\)/,\"\"))`

### Co to dělá:
Snaží se vykuchat podstatná jména (aby "německé romány" a "české romány" byly "romány").

### Co s tím lze dělat dál:
Můžou tam lézt slova, která nejsou tím hlavním podstatným jménem v termínu.
Na ty je třeba přidat výjimku.

**Současné výjimky jsou:**
- vydání
- cvičení
- jednání
- sdělení
- nařízení
- pomůcky
- desky
- tisky
- sci-fi
- fantasy

## 6427: ISBN a ISSN pro exporty

`if(isNonBlank(value),if(isNonBlank(cells[\"020\"].value),if(length(replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),if(isNonBlank(cells[\"022\"].value),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),null),null)),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),null)),null)`

### Co to dělá:
Pokud je v 020$a nebo 022$a po odstranění všech znaků kromě číslic, spojovníků a písmene X řetězec delší než 6 znaků, pak se tato hodnota zapíše do sloupce.
Slouží to pro exporty, tj. aby bylo v seznamu vidět, co je tam skutečně napsané, i kdyby to byl paskvil.

### Co může dělat problémy:
Pokud tam knihovna píše nějaké jiné divné kódy než ISBN či ISSN, pak jsou ve sloupci jen fragmenty.
Pokud se v podpoli vyskytují čísla, spojovníky nebo písmena Xx ještě někde dál v řetězci, pak výstup taky nedává smysl.

## 6492: Indikátor chybějících roků

`if(and(isNonBlank(cells[\"OAI\"].value),isNonBlank(cells[\"nezdedup\"].value),or(isBlank(cells[\"no996\"].value),cells[\"format\"].value.contains(/.*ARTICLES.*/)),isBlank(cells[\"rok\"].value),isBlank(cells[\"260\"].value.match(/.*\\$c([^\\$]+).*/)[0]),isBlank(cells[\"264\"].value.match(/.*\\$c([^\\$]+).*/)[0]),isBlank(cells[\"008\"].value.substring(7,11).match(/([0-9]{2}[u]{2}|[0-9]{3}u)/)[0])),true,null)`

### Co to dělá:
Označuje záznamy, kde jsou více než 3 "u" v 008|07-10 a neexistuje obsah v 260$c nebo 264$c.
Bere jen nezdeduplikované věci s jednotkama a články.

## 6505: Indikátor nevalidních ISBN, ISSN a ISMN

`if(and(isBlank(cells[\"no996\"].value),or(value.contains(/.*BOOKS.*/),value.contains(/.*PERIODICALS.*/))),if(and(isBlank(cells[\"isbn\"].value),isNonBlank(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0])),if(length(replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9xX\\-]/,\"\"))>6,true,null),if(and(isBlank(cells[\"issn\"].value),isNonBlank(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0])),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9xX\\-]/,\"\"))>6,true,null),null)),null)`

### Co to dělá:
Označuje záznamy, kde je nějaký shluk znaků, které se podobají ISBN, ISSN nebo ISMN, ale neutvořil se z nich klíč, protože jsou nevalidní.

## 6544: CD, MC, LP, DVD v názvových údajích

`if(and(isNonBlank(cells[\"nezdedup\"].value),cells[\"format\"].value.contains(/.*BOOKS.*/),isBlank(cells[\"no996\"].value)),value.match(/.*(^|[^\\p{L}])(CD|MC|LP|DVD)([^\\p{L}]|$).*/)[1],null)`

### Co to dělá:
Viz nadpis.

### Co s tím lze dělat dál:
Lze sem přidat jakýkoliv jiný nežádoucí výraz, který se vyskytuje v názvech jako samostatné slovo.

## 6557: Příprava sloupce na odchycení monograficky popsaných seriálů

`if(and(isNonBlank(value),isBlank(cells[\"no996\"].value),isBlank(cells[\"autor\"].value),isBlank(cells[\"isbn\"].value),cells[\"format\"].value.contains(/.*(BOOKS|PERIODICALS).*/),cells[\"titul\"].value.contains(/^\\S.+\\s[0-9]+.*/)),if(cells[\"245aMultiTest\"].value.contains(/(^|\\s)(díl|část|sv|svazek|zákon[y]?|ency[\\p{L}]lop[\\p{L}]+di[\\p{L}]+|slovník|zpráva|rozpočet|ročenka|almanach|sborník)(\\s|$)/),null,if(cells[\"245aMultiTest\"].value.contains(/[\\p{L}]+\\s(č|číslo|r|roč|ročník)(\\s|$)/),cells[\"245aMultiTest\"].value,null)),null)`

### Co to dělá:
Hledá podezřelá slova, která by mohla naznačovat, že jde o monograficky popsaný seriál, přičemž vylučuje slova, která jsou typická u vícedílných monografií.

### Co s tím lze dělat dál:
Lze přidávat další výjimky do "blacklistu" či "whitelistu".

**Blacklist:**
- díl
- část
- sv
- svazek
- zákon, zákony
- různé podoby slova "encyklopedie"
- slovník
- zpráva
- rozpočet
- ročenka
- almanach
- sborník

**Whitelist:**
- č
- číslo
- r
- roč
- ročník