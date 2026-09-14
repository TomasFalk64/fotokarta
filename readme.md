# Fotokarta

Fotokarta är en webbsida som skapar interaktiva kartor med GPS-märkta foton, artfynd, polygoner och dokumenttext. Du kan använda en egen GeoTIFF som bakgrund och spara resultatet som en fristående HTML-fil. Fotouppgifter kan också exporteras till Excel som underlag för rapportering till Artportalen.

## Kom igång

1. Öppna `index.html` i en modern webbläsare, direkt från datorn eller via en webbserver. Inget byggsteg eller någon installation behövs.
2. Läs vid behov in en GeoTIFF-karta.
3. Välj GPS-märkta bilder, importera artfynd från Excel och/eller läs in polygoner från GeoJSON.
4. Bifoga dokumenttext och justera storleken på fotomarkörer och artpunkter.
5. Välj **Ladda ner HTML** för att spara kartan eller **Ladda ner Excel** för att exportera fotouppgifter.

Webbsidan hämtar JavaScript-bibliotek från externa CDN-tjänster och behöver därför internet vid laddning. Filerna som du väljer bearbetas lokalt i webbläsaren; appen har ingen server för uppladdning eller lagring av dessa filer. Arbetet sparas inte automatiskt mellan omladdningar.

## Funktioner

### Karta och navigering

- Visa en egen GeoTIFF som kartbakgrund.
- Panorera genom att dra och zooma med mushjul, dubbelklick eller zoomknappar. Återställ vyn med återställningsknappen.
- Visa geografiska objekt även utan GeoTIFF. Då används en neutral bakgrund och en vy anpassad efter objektens positioner.
- Följ inläsning, fel och överhoppade objekt i loggen.
- Töm inläst innehåll med **Rensa**.

### Foton

- Läs in flera foton direkt eller från en ZIP-fil, även från undermappar i arkivet.
- Placera foton efter GPS-information i EXIF. Foton utan användbar GPS-position hoppas över.
- Klicka på en fotomarkör för att visa bilden, zooma i den och skriva en titel. Titeln används som artnamn i Excel-exporten.
- Justera fotomarkörernas storlek och bildkvaliteten vid inläsning. Ändrad bildkvalitet kräver att bilderna läses in igen.
- En ny bildinläsning ersätter tidigare foton.

### Artpunkter

- Importera artfynd från Excel och visa eller dölj hela artlagret.
- Färgsätt punkterna efter värdet i **Rödlistade**. En färgförklaring visar kategorierna som finns i filen.
- Visa högre rödlistningsklass ovanpå lägre vid överlappning: **ingen → DD → NT → VU → EN → CR → RE**. LC, NE, NA och okända värden har lägsta prioritet.
- Samla fynd med exakt samma position i en punkt med antal. Punkten får den högsta klassens färg och alla fynd kan läsas i klickrutan, med högsta klass först.
- Klicka för att visa artnamn, vetenskapligt namn, startdatum, rödlistning och en länk till respektive fynd i Artportalen.
- Ändra punktstorleken mellan **8 och 48 px**. Vald storlek följer med i HTML-exporten.
- Artpunkter utanför GeoTIFF-kartan visas inte och inkluderas inte i den exporterade kartan. De finns kvar bland inlästa fynd om kartan senare byts.
- En ny artfil ersätter tidigare artfynd efter lyckad import.

### Polygoner och dokument

- Läs in polygoner från GeoJSON. Koordinatsystemet läses ur `crs` om det finns, annars tolkas koordinaterna som WGS84 eller SWEREF 99 TM enligt reglerna nedan.
- Polygoner helt utanför kartans utbredning filtreras bort från visningen. En ny polygonfil ersätter det tidigare polygonlagret.
- Bifoga dokumenttext genom att skriva, klistra in från exempelvis Word eller importera TXT/Markdown. Dokumentet öppnas från en knapp vid kartan och följer med i HTML-exporten.

## Indata

### GeoTIFF

| Egenskap | Krav eller beteende |
| --- | --- |
| Filformat | `.tif` eller `.tiff` med georeferering. |
| Innehåll | Raster som kan läsas av georaster-biblioteket. |
| Geografisk information | Kartans utbredning och projektion används för att placera objekt. |
| Koordinatsystem | Projektionen måste kunna hanteras av Proj4. Appen registrerar EPSG:3006 och EPSG:3011–3018 utöver bibliotekets inbyggda system. Saknas projektion använder koden EPSG:4326 som reservvärde. |

### Bilder

Använd bilder som webbläsaren kan avkoda och vars EXIF innehåller GPS-latitud och GPS-longitud. JPEG med bevarad EXIF är ett lämpligt indataformat. Datum, tid och positionsnoggrannhet läses också ur metadata när de finns.

Bilder kan väljas direkt eller packas i `.zip`. En bild blir inte GPS-märkt enbart genom sitt filnamn. Om metadata försvinner vid redigering, delning eller överföring kan bilden inte placeras automatiskt.

### Excel med artfynd

Filformat: **`.xlsx` eller `.xls`**. Importen använder det första blad där kolumnen `Artnamn` hittas. Kolumner får ligga i valfri ordning och extra kolumner ignoreras. Rubrikernas stora/små bokstäver behöver inte matcha exakt.

Följande fem kolumner måste finnas, även om vissa celler får vara tomma:

| Kolumn | Innehåll och krav per rad |
| --- | --- |
| `ID` | Artportalens fynd-ID, endast siffror. Krävs för fyndlänken. |
| `Artnamn` | Svenskt eller annat visningsnamn. Minst detta fält eller `Vetenskapligt namn` måste vara ifyllt. |
| `Vetenskapligt namn` | Vetenskapligt namn. Används också som visningsnamn om `Artnamn` är tomt. |
| `Rödlistade` | Exempelvis DD, NT, VU, EN, CR, RE eller LC. Tomt värde visas som ”Ej angivet”. Även rubriken `Rödlistad` accepteras. |
| `Startdatum` | Text, helst `ÅÅÅÅ-MM-DD`, eller ett Excel-datum. Tomt värde visas som ”Ej angivet”. |

Dessutom krävs ett komplett koordinatpar:

| Koordinatsystem | Kolumner | Enhet och ordning |
| --- | --- | --- |
| SWEREF 99 TM, EPSG:3006 | `Ost` och `Nord` | Meter. Ost 100 000–1 000 000, Nord 6 000 000–8 000 000. |
| WGS84, EPSG:4326 | `Lat` och `Lon` | Decimalgrader. Latitud −90 till 90, longitud −180 till 180. |

Alternativa rubriker är `Latitud`/`Latitude` och `Long`/`Longitud`/`Longitude`. Både decimalpunkt och decimalkomma accepteras. Om båda koordinatparen finns används numeriska Lat/Lon i första hand; saknas någon av dem försöker importen använda Ost/Nord.

Rader med ogiltigt ID, ofullständig/ogiltig position eller utan båda namnen hoppas över och rapporteras i loggen. Helt tomma rader ignoreras.

Exempel på en rad med SWEREF 99 TM:

| ID | Artnamn | Vetenskapligt namn | Rödlistade | Startdatum | Ost | Nord |
| --- | --- | --- | --- | --- | --- | --- |
| 135958232 | Knärot | Goodyera repens | VU | 2026-08-17 | 702734 | 6650509 |

### GeoJSON med polygoner

Filformat: **`.geojson` eller `.json`**. Stödda geometrier är `Polygon` och `MultiPolygon`, även inuti `Feature`, `FeatureCollection` och `GeometryCollection`. Polygonernas inre ringar kan användas för hål. Punkter och linjer importeras inte som polygoner.

Koordinater anges som numeriska par **`[x, y]`**: `[longitud, latitud]` för WGS84 och `[Ost, Nord]` för SWEREF 99 TM.

Om `crs` finns används det angivna systemet. Exempel:

```json
"crs": {
  "type": "name",
  "properties": { "name": "EPSG:3006" }
}
```

Importen accepterar EPSG-namn, vanliga OGC-URN/URL-format, CRS84 och äldre EPSG-objekt med `properties.code`. Ett `crs` på en geometri eller ett objekt kan ersätta ett ärvt system från samlingen. Angivna projektioner måste finnas i appens Proj4-definitioner; okända system ger ett felmeddelande.

När `crs` saknas avgörs systemet per polygon:

- Alla x-värden inom −180 till 180 och y-värden inom −90 till 90: **WGS84**.
- Alla x-värden inom 100 000–1 000 000 och y-värden inom 6 000 000–8 000 000: **SWEREF 99 TM**.
- Övriga värden: importen avbryts med uppmaning att ange `crs`.

Detta är en tolkning av värdena, inte en generell identifiering av alla koordinatsystem. Ange därför `crs` för andra system. Polygonerna omvandlas till WGS84 internt och därefter till kartans system vid visning. Loggen visar vilket system som användes och om det var angivet eller tolkat.

### Dokumenttext

Importera `.txt` eller `.md`, skriv i dokumentrutan eller klistra in text från Word. Direkt import av `.docx` ingår inte. Texten kan få en titel och lagras som rensad HTML i den exporterade kartan.

## Utdata

### Fristående HTML-karta

**Ladda ner HTML** skapar `Fotokarta_<kartnamn>.html`, eller `Fotokarta_karta.html` om ingen GeoTIFF har valts.

Filen innehåller kartbild, exporterade foton och titlar, polygoner, dokumenttext och artfynd inom kartan. Bilder och visningskod bäddas in, så kartan kan öppnas lokalt utan appens externa bibliotek. Länkar till Artportalen och andra externa webbplatser kräver internet.

Panorering, zoomning, klickbara foton och artfynd samt visa/dölj-knappen för artpunkter finns kvar. Fotomarkörernas storlek, artpunkternas storlek och artlagrets synlighet följer med. Filen är en visningskarta; den har inte redigeringssidans importfunktioner.

### Excel med fotouppgifter

**Ladda ner Excel** skapar `Fotokarta_<kartnamn>.xlsx`, eller `Fotokarta_bilder.xlsx` utan GeoTIFF. Arbetsboken innehåller bladet **Observationer**, med en rad per inläst foto, sorterad efter datum och tid.

| Fält | Källa |
| --- | --- |
| `Artnamn` | Titel som skrivits för fotot. |
| `Lokalnamn` | GeoTIFF-filens namn utan filändelse. |
| `lat`, `lon` | Fotots GPS-position i WGS84, avrundad till sju decimaler. |
| `Ost`, `Nord` | GPS-position omvandlad till SWEREF 99 TM och avrundad till hela meter. |
| `Noggrannhet`, `Startdatum`, `Starttid` | Fotots metadata när uppgifterna finns. |
| Övriga fält | Tomma kolumner för komplettering: `Antal`, `Enhet`, `Publik kommentar`, `Biotop`, `Art som substrat`, `Substrat`, `Substrat-beskrivning`, `Aktivitet`, `Ålder-Stadium` och `Kön`. |

Excel-exporten gäller foton, inklusive inlästa foton utanför kartans utbredning. Den exporterar inte de importerade artpunkterna. Filen är ett underlag som kan kompletteras inför rapportering; appen skickar inga observationer till Artportalen. Utdatafilen har andra kolumner än artimporten och är inte avsedd för direkt återimport som artfynd.

## Teknik och licens

Appen består av `index.html` med HTML, CSS och JavaScript. Den använder Proj4, georaster, exif-js, SheetJS och JSZip via CDN. Ingen backend eller databas behövs. Praktisk filstorlek och antal objekt begränsas av webbläsarens minne och datorns prestanda.

Projektets kod distribueras under **MIT-licensen**, se [LICENSE](LICENSE). Externa bibliotek och inlästa bilder, kartor och data omfattas av sina respektive licenser och rättigheter.
