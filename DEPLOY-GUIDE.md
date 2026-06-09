# Deploy-guide: cxdesign.se på Azure Static Web Apps + GitHub

Den här guiden tar sajten från fil till live på din domän. Du gör stegen som kräver
inloggning (GitHub, Azure, Websupport) — jag hjälper dig med allt däremellan.

---

## Översikt

```
Din kod  →  GitHub-repo  →  Azure Static Web Apps  →  cxdesign.se (DNS hos Websupport)
                              (auto-deploy vid push)     (SSL ingår gratis)
```

Filer som redan är förberedda:
- `index.html` – själva sajten
- `staticwebapp.config.json` – Azure-inställningar (fallback, headers)
- `.gitignore`

---

## Steg 1 – Lägg koden i ett GitHub-repo

1. Skapa ett nytt, **tomt** repo på github.com (t.ex. `cxdesign-site`). Inget README/gitignore behövs – vi har redan filer.
2. I mappen med sajtens filer, kör:

   ```bash
   git init
   git add .
   git commit -m "Första versionen av cxdesign.se"
   git branch -M main
   git remote add origin https://github.com/<ditt-användarnamn>/cxdesign-site.git
   git push -u origin main
   ```

---

## Steg 2 – Skapa Azure Static Web App

1. Logga in på **portal.azure.com**.
2. Sök på **Static Web Apps** → **Create**.
3. Fyll i:
   - **Subscription / Resource Group**: välj eller skapa ny (t.ex. `cxdesign`).
   - **Name**: `cxdesign`
   - **Plan type**: **Free**
   - **Region**: West Europe (närmast Sverige)
   - **Source**: **GitHub** → logga in och välj ditt repo + branch `main`.
4. Build details (viktigt för en ren statisk sajt):
   - **Build Presets**: `Custom`
   - **App location**: `/`
   - **Api location**: (lämna tomt)
   - **Output location**: (lämna tomt)
5. **Create**. Azure lägger automatiskt till en GitHub Actions-fil i ditt repo och
   bygger sajten. Efter ~1–2 min är den live på en adress som
   `https://<slumpnamn>.azurestaticapps.net`.

> Varje gång du sedan pushar till `main` deployar Azure om automatiskt.

---

## Steg 3 – Koppla domänen cxdesign.se

I Azure, öppna din Static Web App → **Custom domains** → **+ Add**.

Vi sätter upp **två** saker:

### A) www.cxdesign.se (enklast – CNAME)
- Välj "Custom domain on other DNS" → ange `www.cxdesign.se`.
- Azure ger dig ett **CNAME**-värde (något i stil med `<app>.azurestaticapps.net`).

### B) cxdesign.se (rotdomänen – kräver lite mer)
- Websupport stödjer **inte** ALIAS/ANAME, så rotdomänen pekas med **A-post + TXT-validering**.
- Azure visar exakt vilket **TXT**-värde (`_dnsauth...`) och vilken **IP-adress (A-post)** du ska använda.

👉 **Kopiera de exakta värdena Azure visar och klistra in dem till mig — jag ger dig då
en färdig lista att klistra in i Websupport.**

---

## Steg 4 – DNS hos Websupport (ta bort vidarebefordran)

1. Logna in på Websupports kontrollpanel → välj domänen `cxdesign.se` → **DNS**.
2. **Ta bort** den nuvarande vidarebefordran/omdirigeringen som pekar mot den andra sajten
   (kan ligga som en A-post, en "URL forward/redirect" eller en webbhotell-pekning).
3. Lägg in posterna från Steg 3:

   | Typ   | Namn / Host        | Värde                              |
   |-------|--------------------|------------------------------------|
   | CNAME | `www`              | `<app>.azurestaticapps.net`        |
   | TXT   | `_dnsauth`         | (värdet Azure visar för apex)      |
   | A     | `@` (eller tomt)   | (IP-adressen Azure visar)          |

   *(Exakta värden fyller vi i tillsammans i Steg 3.)*

4. Spara. DNS kan ta allt från några minuter upp till ~72 timmar att slå igenom
   (oftast under en timme).

---

## Steg 5 – Verifiera

- Tillbaka i Azure **Custom domains** – statusen går till **Validated/Ready**.
- Azure utfärdar automatiskt ett gratis **SSL-certifikat** (https).
- Testa `https://cxdesign.se` och `https://www.cxdesign.se`.

---

## Vanliga frågor

- **Mejl då?** Att peka om webben påverkar inte e-post så länge du lämnar **MX-posterna**
  orörda. Rör bara A/CNAME/TXT enligt ovan.
- **Vill hellre slippa A-post på roten?** Alternativ: flytta DNS till **Azure DNS**
  (stödjer ALIAS för roten). Lite mer jobb men "renare". Säg till om du vill det istället.
- **www → utan www (eller tvärtom)?** Vi kan sätta en redirect så att besökare alltid
  hamnar på samma adress. Fixar vi efter att domänen är validerad.
