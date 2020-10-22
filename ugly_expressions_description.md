# Popis funkce složitějších výrazů a prostor pro návrhy jejich vylepšení:
Proto že OpenRefine nemá rád komentáře v JSONu.

**PŘI ÚPRAVÁCH POZOR!!!**
Ptáme-li se, zda NULL obsahuje něco, výsledkem není FALSE (boolean), ale chyba.
Je-li v podmínce ověřování obsahu pole, je třeba jej napřed otestovat na prázdnost.

## 6180: Konsolidace českých hodnot v 655$ (sloupec `genre`)

`if(contains(replace(value,/\\s\\(.*\\)/,\"\"),/\\s/),replace(replace(replace(value,/\\s\\(.*\\)/,\"\"),/[\\-\\p{L}]+([áéůý]|[cč]í|(?<!(vydá|cviče|jedná|sděle|naříze))ní|(?<!pomů)cky|(?<!(de|ti))sky|fi|antasy)([^\\p{L}]|$)/,\"\"),/^a\\s/,\"\").match(/^(\\p{L}+).*/)[0],replace(value,/\\s\\(.*\\)/,\"\"))`

### Co to dělá:
Snaží se vykuchat podstatná jména (aby "německé romány" a "české romány" byly "romány").

### Co s tím lze dělat dál:
Můžou tam lézt slova, která nejsou tím hlavním podstatným jménem v termínu.
Na ty je třeba přidat výjimku.

**Současné výjimky jsou:**
* vydání
* cvičení
* jednání
* sdělení
* nařízení
* pomůcky
* desky
* tisky
* sci-fi
* fantasy

## 6427: ISBN a ISSN pro exporty (sloupec `ISN`)

`if(isNonBlank(value),if(isNonBlank(cells[\"020\"].value),if(length(replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),if(isNonBlank(cells[\"022\"].value),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),null),null)),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"))>6,replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9Xx\\-]/,\"\"),null)),null)`

### Co to dělá:
Pokud je v 020$a nebo 022$a po odstranění všech znaků kromě číslic, spojovníků a písmene X řetězec delší než 6 znaků, pak se tato hodnota zapíše do sloupce.
Slouží to pro exporty, tj. aby bylo v seznamu vidět, co je tam skutečně napsané, i kdyby to byl paskvil.

### Co může dělat problémy:
Pokud tam knihovna píše nějaké jiné divné kódy než ISBN či ISSN, pak jsou ve sloupci jen fragmenty.
Pokud se v podpoli vyskytují čísla, spojovníky nebo písmena Xx ještě někde dál v řetězci, pak výstup taky nedává smysl.

## 6492: Indikátor chybějících roků (sloupec `noYear`)

`if(and(isNonBlank(cells[\"OAI\"].value),isNonBlank(cells[\"nezdedup\"].value),or(isBlank(cells[\"no996\"].value),cells[\"format\"].value.contains(/.*ARTICLES.*/)),isBlank(cells[\"rok\"].value),isBlank(cells[\"260\"].value.match(/.*\\$c([^\\$]+).*/)[0]),isBlank(cells[\"264\"].value.match(/.*\\$c([^\\$]+).*/)[0]),isBlank(cells[\"008\"].value.substring(7,11).match(/([0-9]{2}[u]{2}|[0-9]{3}u)/)[0])),true,null)`

### Co to dělá:
Označuje záznamy, kde jsou více než 3 "u" v 008|07-10 a neexistuje obsah v 260$c nebo 264$c.
Bere jen nezdeduplikované věci s jednotkama a články.

## 6505: Indikátor nevalidních ISBN, ISSN a ISMN (sloupec `invalidISN`)

`if(and(isBlank(cells[\"no996\"].value),or(value.contains(/.*BOOKS.*/),value.contains(/.*PERIODICALS.*/))),if(and(isBlank(cells[\"isbn\"].value),isNonBlank(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0])),if(length(replace(cells[\"020\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9xX\\-]/,\"\"))>6,true,null),if(and(isBlank(cells[\"issn\"].value),isNonBlank(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0])),if(length(replace(cells[\"022\"].value.match(/.*\\$a([^\\$]+).*/)[0],/[^0-9xX\\-]/,\"\"))>6,true,null),null)),null)`

### Co to dělá:
Označuje záznamy, kde je nějaký shluk znaků, které se podobají ISBN, ISSN nebo ISMN, ale neutvořil se z nich klíč, protože jsou nevalidní.

## 6544: CD, MC, LP, DVD v názvových údajích (sloupec `titulNosic`)

`if(and(isNonBlank(cells[\"nezdedup\"].value),cells[\"format\"].value.contains(/.*BOOKS.*/),isBlank(cells[\"no996\"].value)),value.match(/.*(^|[^\\p{L}])(CD|MC|LP|DVD)([^\\p{L}]|$).*/)[1],null)`

### Co to dělá:
Viz nadpis.

### Co s tím lze dělat dál:
Lze sem přidat jakýkoliv jiný nežádoucí výraz, který se vyskytuje v názvech jako samostatné slovo.

## 6557: Úprava sloupce na odchycení monograficky popsaných seriálů (sloupec `245aSerialTest`)

`if(and(isNonBlank(value),isBlank(cells[\"no996\"].value),isBlank(cells[\"autor\"].value),isBlank(cells[\"isbn\"].value),cells[\"format\"].value.contains(/.*(BOOKS|PERIODICALS).*/),cells[\"titul\"].value.contains(/^\\S.+\\s[0-9]+.*/)),if(cells[\"245aMultiTest\"].value.contains(/(^|\\s)(díl|část|sv|svazek|zákon[y]?|ency[\\p{L}]lop[\\p{L}]+di[\\p{L}]+|slovník|zpráva|rozpočet|ročenka|almanach|sborník)(\\s|$)/),null,if(cells[\"245aMultiTest\"].value.contains(/[\\p{L}]+\\s(č|číslo|r|roč|ročník)(\\s|$)/),cells[\"245aMultiTest\"].value,null)),null)`

### Co to dělá:
Hledá podezřelá slova, která by mohla naznačovat, že jde o monograficky popsaný seriál, přičemž vylučuje slova, která jsou typická u vícedílných monografií.

### Co s tím lze dělat dál:
Lze přidávat další výjimky do "blacklistu" či "whitelistu".

**Blacklist:**
* díl
* část
* sv
* svazek
* zákon, zákony
* různé podoby slova "encyklopedie"
* slovník
* zpráva
* rozpočet
* ročenka
* almanach
* sborník

**Whitelist:**
* č
* číslo
* r
* roč
* ročník

## 6609: Příprava sloupce pro hledání zatoulaných 245$c (sloupec `displaced245c`)

`if(and(isBlank(cells[\"no996\"].value),if(isBlank(cells[\"bl_language\"].value),true,cells[\"bl_language\"].value.contains(/(cze|slo|und|zxx)/))),trim(replace(replace(value,/[^\\p{L}0-9\\s]/,\" \"),/\\s+/,\" \")),null)`

### Co to dělá:
Vezme české, slovenské nebo bezjazykové věci s jednotkami a zmanipuluje název pro další zpracování.

Další krok bude hledat údaje o odpovědnosti v podpolích 245$abnp. Problém mohl někdy vznikat při nějaké konverzi, kdy se část původního 245$c dostane do 245$abnp. Ale občas je to i u novějších záznamů.

### Co s tím lze dělat dál:
Lze to omezit jen na nezdeduplikované věci, ale následně řešený problém se týká i mnoha zdeduplikovaných záznamů (tj. je to podobné jako nevalidní ISBN - i u zdeduplikovaných záznamů je pořád nevalidní).

Lze ubrat či přidat jazyky (v kontextu dalšího kroku).

## 6622: Indikátor přítomnosti údajů o odpovědnosti v názvových údajích (sloupec `displaced245c`)

`if(isNonBlank(value),if(value.contains(/.*\\S+\\s([Pp]řel(ož)?|[Pp]řeklad(e|em|y|ů|ům|ech)?|([Pp]řeklad|[Pp]řispěv|[Vv]ydav)atel(e|i|em|ka|ky|ce|ku|kou|ek|kám|kách|kami)?|(([Šš]éf)?[Rr]edakto[rř]|[Ee]dito[rř])(a|ovi|em|y|ů|ům|ech|ka|ky|ce|ku|kou|ek|kám|kách|kami)?|([Pp]řekladatelsk|[Vv]ydavatelsk|[Kk]ritick)(ý|ého|ému|ém|ým|é|ých|ým|ými|á|ou)|[Ii]l|[Ii]lustr|[Ii]lustrac(e|emi)|([Dd]oslov|[Úú]vod|[Zz]ávěr)(u|em|y|ů|ům|ech)?|[Pp]ředmluv(a|y|ě|u|ou|ám|ách|ami)|([Pp]oznám|[Vv]ysvětliv|[Oo]bál)(ka|ky|ce|ku|kou|ek|kám|kách|kami)|[Ee]dičn(í|ího|ímu|ím|ích|ími|ě)|([Kk]omentář|[Rr]ejstřík)(e|i|em|ů|ům|ích)?|[Rr]esumé|[Rr]ediguje|([Nn]avrhl|([Nn]a)?[Kk]reslil|[Ii]lustroval|[Pp]ře(ložil|vyprávěl|básnil)|([Nn]a|[Ss]e)psal|[Pp]ři(pravil|spěl)|[Uu](spořádal|pravil)|[Oo]patřil|([Vv]y|[Ss]e)bral|[Ss]estavil|[Pp]rovedl|([Ss]polu|[Zz])pracoval|([Vv]y)?[Ff]otografoval|[Dd]o(provodil|plnil)|[Rr]e(digoval|cenzoval)|[Zz]hotovil|[Pp]ořídil|[Vv]y(dal|zdobil))[aiy]?)\\s\\p{Lu}\\S*\\s\\p{Lu}\\S+.*/),true,null),null)`

### Co to dělá:
Pokud předpřiravený slouped obsahuje *SLOVO cokoliv/nic PODEZŘELÉ_SLOVO Nějaké Jméno*, pak z něj udělá TRUE.

### Co může dělat problémy:
Podezřelá slova následovaná slovy začínajícími velkým písmenem mohou být samozřejmě i ve zcela korektním názvu.

### Co s tím lze dělat dál:
Přidávat či ubírat podezřelá slova. Současný seznam byla sestaven podle MKHOL, kde jich bylo opravdu hodně. Určitě ale není úplný. A některá slova v něm mohou nadělat víc škody než užitku.

**Nyní se odchytávají:**
* [Pp]řel(ož)?
* [Pp]řeklad(e|em|y|ů|ům|ech)?
* ([Pp]řeklad|[Pp]řispěv|[Vv]ydav)atel(e|i|em|ka|ky|ce|ku|kou|ek|kám|kách|kami)?
* (([Šš]éf)?[Rr]edakto[rř]|[Ee]dito[rř])(a|ovi|em|y|ů|ům|ech|ka|ky|ce|ku|kou|ek|kám|kách|kami)?
* ([Pp]řekladatelsk|[Vv]ydavatelsk|[Kk]ritick)(ý|ého|ému|ém|ým|é|ých|ým|ými|á|ou)
* [Ii]l
* [Ii]lustr
* [Ii]lustrac(e|emi)
* ([Dd]oslov|[Úú]vod|[Zz]ávěr)(u|em|y|ů|ům|ech)?
* [Pp]ředmluv(a|y|ě|u|ou|ám|ách|ami)
* ([Pp]oznám|[Vv]ysvětliv|[Oo]bál)(ka|ky|ce|ku|kou|ek|kám|kách|kami)
* [Ee]dičn(í|ího|ímu|ím|ích|ími|ě)
* ([Kk]omentář|[Rr]ejstřík)(e|i|em|ů|ům|ích)?
* [Rr]esumé
* [Rr]ediguje
* (
    * [Nn]avrhl|([Nn]a)?[Kk]reslil
      [Ii]lustroval
      [Pp]ře(ložil|vyprávěl|básnil)
      ([Nn]a|[Ss]e)psal
      [Pp]ři(pravil|spěl)
      [Uu](spořádal|pravil)
      [Oo]patřil
      ([Vv]y|[Ss]e)bral
      [Ss]estavil
      [Pp]rovedl
      ([Ss]polu|[Zz])pracoval
      ([Vv]y)?[Ff]otografoval
      [Dd]o(provodil|plnil)
      [Rr]e(digoval|cenzoval)
      [Zz]hotovil
      [Pp]ořídil
      [Vv]y(dal|zdobil)
* )[aiy]?

## 6635: Číslo vydání (sloupec `250a`)

`if(isNonBlank(cells[\"OAI\"].value),if(isBlank(value),\"1\",if(contains(value.match(/.*\\$a([^\\$]+).*/)[0],/[0-9]/),value.match(/.*\\$a[^0-9]*([0-9]+).*/)[0],trim(toLowercase(replace(replace(value.match(/.*\\$a([^\\$]+).*/)[0],/[^\\p{L}]/,\" \"),/\\s+/,\" \"))))),null)`

### Co to dělá:
Sloupec pro kontrolu duplicit. Knihovny nemůžou za sloučení různých vydání z téhož roku.
Extrahuje první posloupnost čísel v podpoli 250$a.
Není-li v 250$a číslo, extrahuje text.
Je-li 250$a prázdné, dosadí "1".

## 6648: Indikátor neurčených kartografických dokumentů (sloupec `neMapy`)

`if(and(isNonBlank(cells[\"OAI\"].value),cells[\"format\"].value.contains(/BOOKS|VISUAL|OTHER/)),if(or(if(isNonBlank(cells[\"genre\"].value),cells[\"genre\"].value.contains(/(mapy|autoatlasy)$/),false),cells[\"titul\"].value.contains(/[0-9]+\\s*\\:\\s*\\[0-9]+/),and(contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(^|\\s)((auto)?map[ay]|soubor[y]? map|autoatlas|atlas světa|plán[y]? měst[a]?)(\\s|$)/),not(contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(myšlenkov|t[ée]matick)/)))),true,null),null)`

### Co to dělá:
Pokud žánr nebo název naznačuje, že jde o kartografický dokument, pak TRUE.
V názvu se hledá měřítko nebo podezřelé slovo.
Rezignace na rozpoznávání podle 653 (praxe měnších knihoven), protože pak tam leze cokoliv, co obsahuje třeba jen jednu mapu ve spoustě textu.
Bere i zdeduplikované záznamy (schválně).

### Co s tím lze dělat dál:
Lze upravovat **podmínky pro žánr"**
* ?mapy
* autoatlasy

Lze upravovat podmínky pro názvové údaje.

**Whitelist:**
* (auto)?map[ay]
* soubor[y]? map
* autoatlas
* atlas světa
* plán[y]? měst[a]?

**Blacklist:**
* myšlenkov
* t[eé]matick

## 6661: Indikátor neurčených hudebnin (sloupec `neNoty`)

`if(and(isNonBlank(cells[\"OAI\"].value),cells[\"format\"].value.contains(/BOOKS/)),if(or(if(isNonBlank(cells[\"genre\"].value),cells[\"genre\"].value.contains(/(hudebniny|partitury|zpěvníky)/),false),if(isNonBlank(cells[\"653\"].value),contains(toLowercase(cells[\"653\"].value),/\\$a(hudebniny|partitury|zpěvníky|noty)(\\s|$)/),false),contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(^|\\s)(zpěvník|škola hry na)(\\s|$)/)),true,null)\n,null)`

### Co to dělá:
Pokud žánr, název nebo volné klíčové slovo naznačuje, že jde o hudebninu, pak TRUE.
Bere i zdeduplikované záznamy (schválně).

### Co s tím lze dělat dál:
Upravovat podmínky.

**Žánr:**
* hudebniny
* partitury
* zpěvníky

**Volně tvořená klíčová slova (653$a):**
* hudebniny
* partitury
* zpěvníky
* noty

**Název:**
* zpěvník
* škola hry na
Pozor, nedávat věci jako "etudy", "pro klavír" apod. Jednak je toho moc, jednak to mohou obsahovat i nehudebniny (Kreutzerova sonáta od Tolstého) a taky by tam napadaly špatně určená audia (na ty lze ale dodělat filtr).