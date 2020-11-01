# Popis funkce složitějších nebo méně zřejmých výrazů a prostor pro návrhy jejich vylepšení:

**PŘI ÚPRAVÁCH POZOR!!!**
Ptáme-li se, zda NULL obsahuje něco, výsledkem není FALSE (boolean), ale chyba.

Je-li v podmínce ověřování obsahu pole, je třeba jej napřed otestovat na prázdnost.

Sloupce, které tady nemají popsaný způsob tvorby, mají dostačující vysvětlení přímo ve skriptu (řádky "description").

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

Bere v potaz i rozpoznaná periodika, protože i takové záznamy mohou být chybně rozepsány po kusech.

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

## 6596: Detekce monograficky popsaných seriálů na základě opakujícího se názvu (sloupec `245aSerialTest`)

`if(and(isNonBlank(value),facetCount(value,'value','245aSerialTest')>1),true,null)`

### Co to dělá:
Za kusově popsané pokračující zdroje se považují opakující se tituly (pouze písmena), protože to bývá v těchto případech obvyklé.

### Co může dělat problémy:
Je-li v knihovně např. jen jediný svázaný ročník nějakého titulu, případně příloha či zvlášní vydání, faseta jej neodchytí.

### Co s tím lze dělat dál:
Ještě před aplikací fasety sjednotit zápis ročníků a čísel (tj. "číslo" na "č", "roč" a "ročník" na "r"). Nepomůže to u seriálů, které jsou v knihovně opravdu jen jednou, ale pomůže zachytit různě zapsaná čísla téhož periodika.

Pokud sem bude padat velké množství vícedílných monografií, lze zvyšovat minimální požadovaný počet opakovaných hodnot. Samozřejmě pak přibudou neodchycené seriály.

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

`if(and(isNonBlank(cells[\"OAI\"].value),isBlank(cells[\"no996\"].value),cells[\"format\"].value.contains(/BOOKS|VISUAL|OTHER/)),if(or(if(isNonBlank(cells[\"genre\"].value),cells[\"genre\"].value.contains(/(mapy|autoatlasy)$/),false),cells[\"titul\"].value.contains(/[0-9]+\\s*\\:\\s*\\[0-9]+/),and(contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(^|\\s)((auto)?map[ay]|soubor[y]? map|autoatlas|atlas světa|plán[y]? měst[a]?)(\\s|$)/),not(contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(myšlenkov|t[ée]matick)/)))),true,null),null)`

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

`if(and(isNonBlank(cells[\"OAI\"].value),isBlank(cells[\"no996\"].value),cells[\"format\"].value.contains(/BOOKS/)),if(or(if(isNonBlank(cells[\"genre\"].value),cells[\"genre\"].value.contains(/(hudebniny|partitury|zpěvníky)/),false),if(isNonBlank(cells[\"653\"].value),contains(toLowercase(cells[\"653\"].value),/\\$a(hudebniny|partitury|zpěvníky|noty)(\\s|$)/),false),contains(trim(toLowercase(replace(replace(cells[\"titul\"].value,/[^\\p{L}]/,\" \"),/\\s+/,\" \"))),/(^|\\s)(zpěvník|škola hry na)(\\s|$)/)),true,null)\n,null)`

### Co to dělá:
Pokud žánr, název nebo volné klíčové slovo naznačuje, že jde o hudebninu, pak TRUE.

Odstraněno pravidlo pro odchytávání 245$h hudebnin bez kódovaných údajů (006|00 zde už nebudeme požadovat).

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

## 6739: Indikátor šedé literatury - 1. krok (sloupec `oldORgrey`)

`if(isBlank(cells[\"OAI\"].value),null,if(or(isBlank(cells[\"nezdedup\"].value),isNonBlank(cells[\"no996\"].value),not(cells[\"format\"].value.contains(/BOOKS/)),isNonBlank(cells[\"noYear\"].value),isNonBlank(cells[\"yearMismatch\"].value),isNonBlank(cells[\"245aSerialTest\"].value),isNonBlank(cells[\"titulNosic\"].value),isNonBlank(cells[\"neMapy\"].value),isNonBlank(cells[\"neNoty\"].value),isNonBlank(cells[\"zCyklu\"].value))\n,false,if(cells[\"rok\"].value<1890,true,null)))`

### Co to dělá:
Vyrobí sloupec a v něm označí jako FALSE záznamy, které jsou aspoň jedno z: 
* zdeduplikované
* bez jednotek
* ne-knihy
* bez data vydání
* s vnitřně konfilktním datem vydání
* ne-knihy zapsané jako knihy (na základě předchozích sloupců)
* cykly
Tyhle věci jsou buď v pořádku, nezajímají nás, nebo se nezdeduplikovaly z již rozpoznaných důvodů.

Za zvážení stojí vyloučit z šedé literatury i vše, co má ISBN, protože to pak není šedá literatura. Pořád to ale můžou být nějaké regionální věci a vlastní náklady, kde nebyl dodán povinný výtisk. Navíc to zatím nevadí dalším výpočtům.

FALSE hodnoty se nakonec vynulují, ale napřed se ještě použijí pro jiný výpočet (hledání záznamů bez ISBN).

Jako TRUE označí záznamy:
* 1890 a starší

### Co s tím lze dělat dál:
Může se ukázat, že dělat společný sloupec pro šedou a starší literaturu nebyl dobrý nápad.

## 6752: Indiátor šedé literatury - 2. krok (sloupec `oldORgrey`)

`if(and(isNonBlank(cells[\"titul\"].value),isBlank(value)),if(contains(trim(toLowercase(replace(cells[\"titul\"].value,/[^\\p{L}]+/,\" \"))),/(^|\\s)(katalog|plán|program|(prů|vý)zkum|report|ročenka|sborník|soupis|studie|úplné znění|zpráva)(\\s|$)/),true,null),value)`

### Co to dělá:
Jako TRUE označí dosud ne-TRUE a ne-FALSE záznamy, které mají v názvu:
* katalog
* plán
* program
* (prů|vý)zkum
* report
* ročenka
* sborník
* soupis
* studie
* úplné znění
* zpráva 

### Co s tím lze dělat dál:
První nastavení podmínek bylo udělané převážně podle slov z formálních deskriptorů, které se docela často vyskytují v názvech šedé literatury. Samozřejmě to všechno nemusí být šedá literatura.

Tj. lze přidávat či ubírat slova, případně jejich předpony a koncovky.

## 6765: Indikátor šedé literatury - 3. krok (sloupec `oldORgrey`)

`if(and(isNonBlank(cells[\"bl_publisher\"].value),isBlank(value)),if(cells[\"bl_publisher\"].value.contains(/(archiv|asociace|druzst|gymna[zs]i|(?<!knizni)klub|knihov|kraj|mest|mu[sz]e(a|jni|um)|obc[ei]|obec|okres|region|sdruzen|spol[e]?k|ucilist|(zakladni|stredni|(vyso|umelec)k(a|e|ych))skol)/),true,null),value)`

### Co to dělá:
Jako TRUE označí dosud ne-TRUE a ne-FALSE záznamy, které mají ve vydavatelském klíči:
* archiv
* asociace
* druzst
* gymna[zs]i
* (?<!knizni)klub *tj. různé kluby, kromě Knižního klubu*
* knihov
* kraj
* mest
* mu[sz]e(a|jni|um)
* obc[ei]
* obec
* okres
* region
* sdruzen
* spol[e]?k
* ucilist
* (zakladni|stredni|(vyso|umelec)k(a|e|ych))skol

### Co s tím lze dělat dál:
Ubírat a přidávat výrazy dle potřeby. Určitě sem napadá spousta ne-šedé literatury - když jí bude příliš mnoho, bude to třeba přehodnotit.

## 6778: Indikátor šedé literatury - 4. krok (sloupec `oldORgrey`)

`if(and(isBlank(value),isNonBlank(cells[\"OAI\"].value),isNonBlank(cells[\"genre\"].value)),if(contains(join(row.record.cells[\"genre\"].value,\" \"),/(brožury|disertace|dokumenty|hry|katalogy|normy|práce|programy|přednášky|ročenky|sborníky|separáty|skripta|soupisy|studie|texty|tisky|zákony|zprávy)/),true,null),value)`

### Co to dělá:
Jako TRUE označí dosud ne-TRUE a ne-FALSE záznamy, které mají žánr/formu obsahující:
* brožury
* disertace
* dokumenty
* hry
* katalogy
* normy
* práce
* programy
* přednášky
* ročenky
* sborníky
* separáty
* skripta
* soupisy
* studie
* texty
* tisky
* zákony
* zprávy

### Co s tím lze dělat dál:
Vybrány byly výrazy, které se (skoro všechny) vyskytovaly v nějakém ne úplně aktuálním seznamu formálních deskriptorů (třeba brožury tam nebyly, i když existují jako fd). Z nich pak byly vybrány ty, které se v nějakém nezanedbatelném množství vyskytují v datech (>10 000).

Lze tedy přidávat a ubírat výrazy, přičemž je třeba mít na paměti, že se pracuje se sloupcem `genre`, který neobsahuje smysluplné hodnoty u cizojazyčných termínů.

## 6791: Indikátor šedé literatury - 5. krok (sloupec `oldORgrey`)

`if(and(isBlank(value),isNonBlank(cells[\"OAI\"].value),isNonBlank(cells[\"650\"].value)),if(contains(trim(toLowercase(replace(replace(join(join(row.record.cells[\"650\"].value,\" \").find(/\\$a[^\\$]+/),\" \"),/\\$a/,\" \"),/[^\\p{L}]+/,\" \"))),/(region|města|obce)/),true,null),value)`

### Co to dělá:
Jako TRUE označí dosud ne-TRUE a ne-FALSE záznamy, které mají v podpolích 650$a:
* region
* města
* obce

### Co s tím lze dělat dál:
První tři podmínky jsou jen hrubý nástřel pro začátek. Ale možná s tím ani netřeba dělat nic dalšího. Pole 650 mívají "slušné", často přebrané záznamy, které jsou zdeduplikované. To by musela být knihovna, kde si skutečně dávají práci s hledáním správných věcných autorit při vlastnoruční katalogizaci.

## 6804: Indikátor šedé literatury - 6. krok (sloupec `oldORgrey`)

`if(and(isBlank(value),isNonBlank(cells[\"OAI\"].value),isNonBlank(cells[\"653\"].value)),if(contains(trim(toLowercase(replace(replace(join(join(row.record.cells[\"653\"].value,\" \").find(/\\$a[^\\$]+/),\" \"),/\\$a/,\" \"),/[^\\p{L}]+/,\" \"))),/(region|okres|obecní|obce|obec(\\s|$)|kraj|oblast|vlastivěd|muzej|muzea|archiv|spolky|výzkum|knihov)/),true,null),value)`

### Co to dělá:
Jako TRUE označí dosud ne-TRUE a ne-FALSE záznamy, které mají v podpolích 653$a:
* region
* okres
* obecní
* obce
* obec
* kraj
* oblast
* vlastivěd
* muzej
* muzea
* archiv
* spolky
* výzkum
* knihov

### Co s tím lze dělat dál:
Hodně věcí. Pole 653 bývá v malých knihovnách univerzálním polem na označování čehokoliv.

Je ale třeba dávat pozor. Už je vyzkoušeno, že přidání výrazů pro mapy nebo hudebniny nedělá dobrotu, i když je to v některých jediný způsob, jak poznat všechny záznamy pro tyto typy dokumentu. U takových knihoven už si radši udělejme filtr ručně.

## 6817: Hledání záznamů s pravděpodobně chybějícím ISBN (sloupec `noISN`)

`if(isBlank(cells[\"ISN\"].value),if(isNonBlank(cells[\"rok\"].value),if(cells[\"rok\"].value<1989,null,if(or(isNonBlank(cells[\"oldORgrey\"].value),contains(cells[\"title\"].value,/[^0-9]+[0-9]{4}/)),null,true)),null),null)`

### Co to dělá:
Hledá nezdeduplikované knihy od 1989, které nemají ISBN a nebyl u nich rozpoznán jiný problém. Vylučuje záznamy s aspoň čtyřciferným číslem následujícím za nečíslicí v názvovém klíči (kvůli vyloučení harlekýnek, které se často píší s rokem v názvu, příloh a zvláštních vydání).

### Co s tím lze dělat dál:
Udělat něco i na ISSN, ale tam se to hůř poznává u dlouho vycházejících titulů.

Udělat to i na zdeduplikované věci. Teď to není, protože využívá předchozího výpočtu, kde byly vyloučeny zdeduplikované záznamy.

## 6830: Neurčené důvody nezdeduplikování (sloupec `zbytek`)

`if(isNonBlank(value),if(or(isBlank(cells[\"bl_language\"].value),and(isNonBlank(cells[\"bl_language\"].value),cells[\"bl_language\"].value.contains(/(cze|slo|und|zxx)/))),if(and(isBlank(cells[\"oldORgrey\"].value),isBlank(cells[\"displaced245c\"].value),isBlank(cells[\"invalidISN\"].value),isBlank(cells[\"noISN\"].value)),true,null),null),null)`

### Co to dělá:
Jako TRUE označí nezdeduplikované české, slovenské či bezjazykové záznamy, u kterých se nenašel žádný společný důvod nezdeduplikování.

Neoznačí záznamy, které se nebraly v potaz ani při hledání šedé a staré literatury. Kromě nich vyloučí ještě záznamy s neplatným ISBN či ISSN, s chybějícím ISBN (dle předchozího výpočtu) a s podezřením na přítomnost údajů o odpovědnosti v názvu.

Po skončení tohoto kroku se vynulují FALSE hodnoty ve sloupci oldORgrey.