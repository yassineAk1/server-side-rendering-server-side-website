# Snappthis: Server-Side Website

## Inhoudsopgave

- [Beschrijving](#beschrijving)
- [Gebruik](#gebruik)
- [Kenmerken](#kenmerken)
- [Installatie](#installatie)
- [Bronnen](#bronnen)
- [Licentie](#licentie)


## Beschrijving


<!-- Voeg hier een screenshot toe van de app op mobiel formaat, bijvoorbeeld de snappmap-pagina met foto's erin. Dit geeft meteen een goed beeld van hoe de app eruitziet. -->

<!-- Voeg hier een link toe naar de live website zodra die gehost is -->

Sprint 8 draaide volledig om server-side rendering. De opdracht: bouw een werkende webapplicatie voor Snappthis, een platform waar gebruikers foto's plaatsen binnen thematische collecties genaamd snappmaps.

De uitdaging was om niet langer statische HTML te schrijven, maar om een Express-server op te zetten die live data uit een externe REST API ophaalt en die via Liquid-templates omzet naar dynamische webpagina's. Elke keer dat iemand een pagina bezoekt, wordt de meest recente data opgehaald en weergegeven.

Ik heb deze sprint de volgende paginas gebouwd:

- **Snappmap**: de hoofdpagina waar alle foto's van een geselecteerde snappmap worden getoond. Bovenin kun je via een dropdown wisselen naar een andere snappmap.
- **Groepenoverzicht**: een overzicht van alle beschikbare groepen, elk met de mogelijkheid om leden of snaps toe te voegen.
- **Groep detailpagina**: een pagina die je laat zien welke snappmaps er binnen een specifieke groep beschikbaar zijn.

![Free iPhone 17 Pro (1)](https://github.com/user-attachments/assets/dcaf927e-b258-49c2-861d-e2bbad828539)


## Gebruik



De app is gebouwd voor mobiel gebruik en werkt als volgt:

Je landt op de homepagina en ziet direct de meest recente snappmap, gevuld met alle gemaakte foto's. Staat er nog geen foto bij een snap? Dan verschijnt er automatisch een placeholder. Bovenaan de pagina staat de naam van de snappmap als knop: klik erop om een dropdown te openen met alle andere snappmaps. Zo wissel je eenvoudig tussen collecties.

Onder de snappmap-naam staat een gekleurde balk met de naam van de groep waar die snappmap bij hoort. Via die balk navigeer je naar de groepspagina, waar alle snappmaps van die groep op een rij staan. Vanuit het groepenoverzicht (bereikbaar via de navigatiebalk onderaan) zie je alle groepen in één keer.


## Kenmerken

De applicatie is gebouwd met de volgende technieken:

**Back-end:** NodeJS + Express
**Templating:** LiquidJS
**Data:** REST API via FDND Directus
**Front-end:** HTML, CSS (mobile-first), SVG-iconen, custom font Bariol

### Routes en datastromen

De server draait op poort `8000` en verwerkt de volgende routes:

| Route | Wat er gebeurt |
|---|---|
| `GET /` | Haalt alle snappmaps op. De eerste in de lijst wordt automatisch geopend met al zijn snaps en gekoppelde groep. |
| `GET /snapmap/:uuid` | Haalt één specifieke snappmap op via de UUID in de URL, inclusief alle snaps en de bijbehorende groep. |
| `GET /groepen` | Haalt alle groepen op uit de database en geeft die door aan het groepenoverzicht. |
| `GET /groep/:uuid` | Haalt één groep op met alle bijbehorende snappmaps, zodat de gebruiker er naartoe kan navigeren. |

Om de server zo min mogelijk te laten wachten, worden API-aanroepen die onafhankelijk van elkaar zijn tegelijk verstuurd via `Promise.all`. Zo hoeft de server niet te wachten tot de eerste fetch klaar is voordat de tweede begint:

```js
// server.js: regel 52-55
const [snapmap, snapmaps] = await Promise.all([
  fetchData(`https://fdnd-agency.directus.app/items/snappthis_snapmap/${req.params.uuid}?fields=*,snaps.*,groups.snappthis_group_uuid.*`),
  fetchData('https://fdnd-agency.directus.app/items/snappthis_snapmap'),
])
```

### Liquid-templates

De views staan in de map `/views`. Gedeelde onderdelen zoals de `<head>` en de footer/navigatiebalk zijn opgesplitst in aparte partials (`head.liquid` en `foot.liquid`) en worden via `{% render %}` hergebruikt in elke pagina.

Data uit de API wordt als object meegegeven vanuit de route en is daarna direct beschikbaar in de template. Een voorbeeld: de foto's van een snappmap worden weergegeven via een loop. Als een snap geen foto heeft, valt de template terug op een placeholder:

```liquid
<!-- views/snapmap.liquid: regel 36-44 -->
{% for snap in snaps %}
  <li class="photo-item">
    {% if snap.picture %}
      <img src="https://fdnd-agency.directus.app/assets/{{ snap.picture }}" alt="snap">
    {% else %}
      <img src="/placeholder.jpeg" alt="">
    {% endif %}
  </li>
{% endfor %}
```

De actieve pagina in de navigatie wordt bijgehouden via de variabele `activePage`, die vanuit elke route als waarde wordt meegegeven (bijv. `'home'` of `'groepen'`) en in `foot.liquid` vergeleken wordt om de juiste knop te markeren.

### Styling en huisstijl

<!-- Voeg hier een screenshot toe van de navigatiebalk onderaan en de groepspagina, zodat de huisstijl goed zichtbaar is. -->
![Free iPhone Air](https://github.com/user-attachments/assets/5cee4ba0-c435-4970-b9ca-2b3f940566f5)

<img width="471" height="71" alt="Schermafbeelding 2026-04-05 140628" src="https://github.com/user-attachments/assets/cfc2e2f4-640b-4841-9e34-17c03b75d252" />


De CSS volgt de visuele identiteit van Snappthis. Het lettertype **Bariol** (bold, regular en italic) is als lokaal font geladen via `@font-face`. Alle iconen zijn SVG-bestanden die rechtstreeks uit de huisstijl komen, zoals de camera, het grid-icoon en de groepsiconen. De layout is mobile-first opgebouwd: smal en verticaal, precies zoals een mobiele webapp verwacht wordt te zijn.


## Installatie

Wil je lokaal aan dit project werken? Volg dan de stappen hieronder.

**Vereiste:** zorg dat [NodeJS](https://nodejs.org/en/download) geïnstalleerd is op jouw computer. Kies de LTS-versie.


**1. Repository forken en clonen**

Fork dit project via de GitHub-knop rechtsboven, clone daarna jouw fork naar je computer en open de map in je code editor.

```bash
git clone https://github.com/jouw-gebruikersnaam/server-side-rendering-server-side-website.git
```


**2. Packages installeren**

Open de terminal in je code editor en voer uit:

```bash
npm install
```

Dit installeert alle benodigde packages (Express, LiquidJS, Nodemon) in de map `node_modules`.


**3. Server starten**

```bash
npm start
```

De terminal toont: `Application started on http://localhost:8000`
Open die URL in je browser om de app te bekijken.


**4. Live herladen (optioneel)**

Open een tweede terminal en voer uit:

```bash
npm run dev
```

Dit start Browser Sync, zodat de browser automatisch ververst zodra je CSS of Liquid-bestanden aanpast.


## Bronnen

- [FDND Directus API: Snappthis](https://fdnd-agency.directus.app/items/snappthis_snapmap)
- [LiquidJS: officiële documentatie](https://liquidjs.com/tutorials/intro-to-liquid.html)
- [Express.js: officiële documentatie](https://expressjs.com/)
- [MDN: Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
- [Directus: filter rules](https://directus.io/docs/guides/connect/filter-rules)


## Licentie

This project is licensed under the terms of the [MIT license](./LICENSE).
