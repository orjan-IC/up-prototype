# UPerformance App: prototyp för telefon och iPad

Det här är en klickbar prototyp av UPerformance-patientappen. En och samma kodbas ger två layouter:

- **iPad:** sidomeny i liggande läge och en flytande flikrad i stående läge. Översikten visas i kolumner, och daglig incheckning och hälsoenkät öppnas som ark ovanpå översikten. Hälsoenkäten har en avsnittslista i liggande läge.
- **Telefon:** samma utseende som originalet från Claude Design (UPerformance Patientapp 3), med gradientheader och flikrad.

Layouten väljs automatiskt efter skärmytan. Den blir iPad när ytan är minst 700 × 600 punkter. iPad i Split View får därför telefonlayouten. På en dator visas appen i en telefon- eller iPad-ram, med genvägar till skärmarna.

## Köra demon på en iPad (ser ut som en riktig app)

1. Lägg innehållet i `dist/web/` på en webbserver med https, t.ex. Netlify Drop (dra mappen till app.netlify.com/drop), GitHub Pages eller en egen server.
2. Öppna adressen i Safari på iPaden, tryck på **Dela** och välj **Lägg till på hemskärmen**.
3. Starta appen från ikonen. Den öppnas då i helskärm utan Safaris fält. Statusfältet får appens färg, och appen fungerar offline efter första starten.

Vid demo: stäng av Automatisk låsning och slå på **Guidad åtkomst** (Inställningar → Tillgänglighet), så att ingen råkar lämna appen.

URL-parametrar för test:

- `?frame=0` tvingar helskärm.
- `?frame=1` tvingar ram.
- `?device=phone` eller `?device=ipad` tvingar layout.
- `?lang=en` startar på engelska.

## Utveckla vidare

```
npm install
npm run icons    # appikoner (behöver Playwright/Chromium)
npm run build    # -> dist/web/ (för hosting) och dist/artifact/index.html (en fil)
```

| Fil | Innehåll |
|---|---|
| `src/data/content.js` | Texter (sv/en), incheckningsfrågor, kroppsområden och mätserier, flyttade från originalet |
| `src/data/halsodeklaration.js` | FHIR-formuläret (Idrottsmedicinsk hälsodeklaration) |
| `src/data/extra.js` | Nya texter för iPad-layouten |
| `src/store.js` | Allt tillstånd och all logik. Gemensamt för båda layouterna |
| `src/ui/kit.jsx` | Knappar, kort, ikoner, logotyp och kroppsfigur |
| `src/screens/` | Skärmarnas innehåll. Layoutneutralt |
| `src/layouts/PhoneShell.jsx` | Telefonskalet |
| `src/layouts/TabletShell.jsx` | iPad-skalet |
| `src/Stage.jsx` | Demoscen med enhetsram för datorn |

En ny skärm eller funktion läggs till så här:

1. Lägg logiken i `store.js`.
2. Lägg innehållet i `src/screens/`.
3. Rama in innehållet i båda skalen.

Då hålls telefon- och iPad-versionen i fas.

## 3D-avatar

`src/ui/avatar/` innehåller en egen parametrisk 3D-kropp i hologramstil, byggd med three.js:

- Kroppen byggs av mjukt sammansmälta former (SDF) och ytan tas fram med surface nets. Modellen är vår egen och inga licensierade 3D-filer används.
- Det finns en man och en kvinna. Parametrar för pojke och flicka finns i `BODIES` och kan kopplas på.
- Man vrider kroppen genom att dra och väljer område genom att trycka. Fram och bak synkas med knappen. Områdena färgas per sida efter status.
- Enheter utan WebGL visar den gamla 2D-figuren.

## Ändringar mot originalet

- Appen har ImagineCare-appens ljusa grund: vit bakgrund och vita kort med mjuk skugga. Accentfärgen är UPerformance mörka teal.
- Inloggningen följer ImagineCare-appens inloggning, men utan MitID. BankID-knappen har UPerformance-färgen och Freja eID+ har Frejas lila.
- Flödena för BankID och Freja eID följer BankID:s UI-guide för mobil (developers.bankid.com/ui-resources/ui-guide-mobile):
  - Först visas "Starta BankID-appen", med en tillbaka-knapp och ett alternativ för annan enhet.
  - På annan enhet visas en animerad QR-kod med tid kvar, hjälptexter och ett alternativ för att gå tillbaka till den här enheten.
  - I demon tar man sig vidare med "Fortsätt", eller genom att trycka på QR-koden.
- BankID-märket i `src/assets/brand/` är tillfälligt, utklippt ur en skärmdump. Byt det mot de officiella filerna från BankID:s varumärkessida innan appen visas externt. Freja-logotypen är den fil vi fick.

- Logotypen på inloggningen har fått raden "powered by ImagineCare" under "Performance". Den är satt i Poppins, i mindre storlek.
- Originalets dubblerade områdespost för Bröst är borttagen. Den andra posten gick aldrig att välja.
- **Hälsodata** har sex parametrar: blodtryck med puls, vilopuls, HRV, sömn, lungfunktion och vikt. Varje parameter visas i vyerna Dagar och Veckor, med målband och statusprickar. Detaljsidan har plan, delning och senaste mätningar. Sömn leder till en ämnessida med en artikel.
- **Kontakt** är en chatt med medicinska teamet, fystränaren och lagkansliet. Meddelanden kan skickas och får automatiska svar. På iPad visas en samtalslista bredvid chatten.
- **Profil** följer ImagineCare-mönstret. Personen är påhittad (Theo Spurtén, mittfältare) och har en illustrerad profilbild. Profilen innehåller hälsoplan, mätenheter och inställningar, bland annat kropp i 3D-vyn, språk och utloggning.
- **Daglig incheckning = självskattning.** Frågorna besvaras på en ansiktsskala 1–5, i flödet "Logga mående" med en översikt på slutet. Frågorna gäller mående, sömn och kost, plus en skadefråga. Man kan börja direkt från självskattningskortet på översikten, som är en karusell på telefon och ett rutnät på iPad. Svaren syns i Hälsodata under Självskattning, med ansiktsdiagram och detaljsida. Frågorna och historiken finns i `src/data/selfassess.js` och byts mot exempeldata när den kommer.
- **Videosamtal** kan vara inkommande (Avfärda/Svara), utgående (Ringer…) och pågående, med mikrofon, kamera, högtalare och lägg på. På telefon delas skärmen i överkant och underkant. På iPad visas samtalet som i FaceTime: motparten i helbild, egen bild i en liten ruta och flytande kontroller. Egen bild använder framkameran om den tillåts, annars visas en illustration. Motparten är en animerad illustration. Samtal startas från videobesöket på översikten, från videoknappen i chatten eller med demoscenens genväg 6. Avslutade samtal loggas i chatten.
- "powered by ImagineCare" står under logotypen i alla vyer: inloggning, översiktens header på telefon, sidomenyn på iPad (liggande) och flikraden på iPad (stående).
