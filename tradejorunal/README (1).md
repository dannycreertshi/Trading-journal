# LEDGER — Trade Journal (Dizzyfutes)

Eén bestand: `index.html`. Geen build-tools, geen server nodig, geen dependencies om te installeren. Alles staat lokaal in je browser (localStorage) — er wordt niks naar internet verzonden.

## Lokaal draaien
Dubbelklik gewoon op `index.html` en het opent in je browser. Klaar.

Wil je het via een lokale server draaien (soms nodig als je later dingen zoals uploads toevoegt):
```
cd tradejournal
python3 -m http.server 8000
```
en ga naar `http://localhost:8000`.

## Online zetten (gratis)
Het is een static site — één HTML-bestand. Opties:
- **Netlify Drop**: sleep de map naar https://app.netlify.com/drop
- **Vercel**: `vercel deploy` in de map (of sleep de map in de dashboard UI)
- **GitHub Pages**: zet `index.html` in een repo, zet Pages aan op de `main` branch

## Let op: data per browser/device
De data (trades, accounts) staat in `localStorage` van de browser waarin je het opent. Dat betekent:
- Als je het lokaal én online gebruikt, zijn dat **twee gescheiden datasets** (ander domein = andere localStorage).
- Wissen van browserdata/cache verwijdert ook je trades.
- Er is geen sync tussen devices in deze versie.

Wil je dat later oplossen (bv. via een gratis backend als Supabase, zodat je overal dezelfde data ziet en het ook op je telefoon werkt), zeg het maar — dat is een kleine uitbreiding op deze basis.

## Wat erin zit
- **Dashboard**: totale PnL, win rate, balans, groei %, equity curve grafiek, laatste trades. Filter per account.
- **Trades**: volledige tabel van alle trades (entry, close, setup, PnL $ en %), toevoegen/bewerken/verwijderen.
- **Accounts**: per account (met presets voor FTMO, Apex, TopStep, The5ers, MyFundedFutures, of eigen account) zie je live voortgang tegen de eisen: profit target, max drawdown, daily drawdown, min. trading dagen, consistency rule — met een duidelijke ✓/✗ of het account in aanmerking komt voor payout/funded.
- **Deel-kaartje**: bij elke trade een "↗ Deel" knop die een downloadbare PNG genereert in twee stijlen (AXIOM-stijl wit kaartje, TROJAN-stijl donker kaartje met %), met je eigen handle en referral-tekst — geïnspireerd op de screenshots die je aanleverde.

## Volgende stappen (optioneel, laat het weten)
- Screenshot/afbeelding uploaden per trade
- Meer prop firm presets of eigen regels (bv. scaling plan)
- Losse export/import van je data (JSON) zodat je 'm makkelijk kan back-uppen of overzetten
- Cloud sync zodat desktop + telefoon dezelfde data zien

## Commissie / fee & automatische PnL-berekening
Per account stel je nu een **commissie per lot/contract** en een **standaard puntwaarde** in (bij "Nieuw account" / "Account bewerken"). De puntwaarde hangt af van de asset die je op dat account traded — bv. NQ ≈ $20 per punt per contract, ES ≈ $50, EURUSD 1 lot ≈ $10 per pip.

Bij het invoeren van een trade:
- Vul je entry, close en lot/contracts in — de app berekent de **bruto PnL** automatisch (`(close - entry) × lot × puntwaarde`, met het teken omgedraaid bij short).
- De **fee** wordt berekend als `commissie per lot × lot`.
- De **gerealiseerde (netto) PnL** = bruto PnL − fee. Dit is het bedrag dat je accountbalans, win rate, kalender en groei-curve gebruiken.
- Je kunt het PnL-veld ook gewoon handmatig invullen (bv. voor crypto/CFD's waar je alleen het netto resultaat weet) — dan wordt dat als bruto PnL gebruikt en gaat de fee er nog vanaf.

Op het deel-kaartje zie je de volledige rekenketen: bruto PnL → fee → gerealiseerde PnL (het grote getal bovenaan), plus open/close prijs en het totale accountsaldo.

## Cloud sync (Supabase) — over meerdere apparaten
De app is nu gekoppeld aan een Supabase-project voor sync. Zo werkt het:

1. **Open de app** → je krijgt een inlogscherm. Vul je e-mail in en klik "Stuur magic link".
2. Check je inbox, klik op de link → je bent ingelogd en je data wordt automatisch gesynced.
3. Open de app op een ander apparaat, log in met **hetzelfde e-mailadres** → je ziet dezelfde trades, accounts en payouts.

Wil je de app alleen lokaal gebruiken (geen account)? Klik op "Zonder account verdergaan" — dan werkt alles zoals voorheen, puur in de browser van dat apparaat.

### Belangrijk: host de app ergens met een echte URL
Voor de magic link moet Supabase je kunnen terugsturen naar de app. Dat werkt alleen betrouwbaar als je de app **online hebt gezet** (Netlify Drop / Vercel / GitHub Pages, zie hierboven) — niet als je het bestand lokaal opent via `Bestand → Openen`. Gebruik op elk apparaat dezelfde gehoste URL.

### Eenmalige Supabase-instelling: redirect URL toestaan
In je Supabase-project: ga naar **Authentication → URL Configuration** en zet daar je gehoste URL (bv. `https://jouw-site.netlify.app`) bij **Site URL** en/of **Redirect URLs**. Zonder dit kan de magic link je niet terugsturen naar de app.

### Waar je data staat
Je volledige dataset (accounts, trades, payouts, setups, entries) staat als één document in de `ledger_data`-tabel in je Supabase-project, gekoppeld aan je ingelogde gebruiker. Alleen jij (ingelogd met je eigen account) kunt erbij — dat is geregeld via de Row Level Security policies die je eerder hebt aangemaakt.
