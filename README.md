# Bludiště stanice Orion — Online

Multiplayer bludiště pro 3 hráče, běží na statickém webu (GitHub Pages) + Cloud Firestore pro sdílený stav lobby a hry — stejný vzor jako u vašich ostatních her (`rooms`, `mazegames`, `numberduel3`...).

## 1) Firebase — Cloud Firestore

Podle tvého screenshotu už máš ve Firebase projektu `test` Cloud Firestore zapnutý a používaný pro ostatní hry, takže:

1. Otevři **Project settings → General → Your apps** a zkopíruj `firebaseConfig` (apiKey, authDomain, projectId, appId...). Realtime Database `databaseURL` tady není potřeba, hra teď jede přes Firestore.
2. V `index.html` najdi blok `const firebaseConfig = {...}` úplně nahoře ve `<script>` a nahraď placeholdery svými skutečnými hodnotami.
3. Ve **Firestore → Rules** přidej k těm stávajícím (rooms, mazegames, numberduel3, wavelength2, thisorthat2) nový match blok pro tuhle hru:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId} { allow read, write: if true; }
    match /mazegames/{roomId} { allow read, write: if true; }
    match /numberduel3/{roomId} { allow read, write: if true; }
    match /wavelength2/{roomId} { allow read, write: if true; }
    match /thisorthat2/{roomId} { allow read, write: if true; }
    match /orionmaze/{roomId} {
      allow read, write: if true;
      match /pos/{slot} {
        allow read, write: if true;
      }
    }
  }
}
```

Kolekce se jmenuje `orionmaze` a každá lobby je dokument (kód lobby = ID dokumentu), pozice hráčů jsou poddokumenty v `pos/{0,1,2}`.

## 2) GitHub Pages

1. Založ nový repozitář na GitHubu (např. `orion-bludiste`).
2. Nahraj do něj obsah tohoto adresáře (`index.html` + složku `assets/` se třemi fotkami postav):
   ```
   git init
   git add .
   git commit -m "Bludiště stanice Orion online"
   git branch -M main
   git remote add origin https://github.com/TVUJ_UCET/orion-bludiste.git
   git push -u origin main
   ```
3. V repozitáři: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Po chvíli bude hra dostupná na `https://TVUJ_UCET.github.io/orion-bludiste/`.

## 3) Test

- Otevři odkaz, vytvoř lobby, kód pošli kamarádům.
- Kamarádi otevřou stejný odkaz, dají "Připojit se" a zadají kód.
- Synchronizace jede přes Firestore real-time listenery (`onSnapshot`) a přiřazení postav / ready stav / vyhodnocení vítěze přes `runTransaction`, takže i když dva lidi kliknou "hotovo" ve stejnou chvíli, vyhraje spolehlivě jen jeden.

## Poznámky

- Fotky postav jsou v `assets/` jako samostatné soubory (ne base64 v HTML).
- `localStorage` slouží jen k tomu, aby ses po refreshi stránky vrátil na svůj slot v lobby.
- Otevřená pravidla (`allow read, write: if true`) znamenají, že kdokoliv se znalostí kódu lobby (5 náhodných znaků) může do dat zasáhnout — pro partu kamarádů v pohodě, ale je to stejná úroveň zabezpečení jako u vašich ostatních her.
