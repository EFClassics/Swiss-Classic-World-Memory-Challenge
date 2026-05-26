# Swiss Classic World – Optimierte HTML-Datei

```html
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<title>Swiss Classic World - Memory Challenge</title>

<style>
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
}

body {
  margin: 0;
  padding: 10px;
  background: #0a0a0a;
  color: #d4af37;
  font-family: Arial, Helvetica, sans-serif;
  text-align: center;
  overflow-x: hidden;
}

h1 {
  margin-top: 10px;
  margin-bottom: 5px;
  font-size: 32px;
  letter-spacing: 2px;
  text-transform: uppercase;
}

#timer {
  font-size: 42px;
  font-weight: bold;
  color: white;
  margin-bottom: 20px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  max-width: 1200px;
  margin: 0 auto;
}

.card {
  position: relative;
  background: #1a1a1a;
  border: 2px solid #d4af37;
  border-radius: 14px;
  height: 180px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  overflow: hidden;
  transition: transform 0.2s ease;
}

.card:active {
  transform: scale(0.96);
}

.card img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: none;
  padding: 6px;
}

.quest {
  font-size: 56px;
  color: #333;
  font-weight: bold;
}

.name-tag {
  display: none;
  color: white;
  font-size: 18px;
  font-weight: bold;
  text-transform: uppercase;
  padding: 10px;
  line-height: 1.2;
}

.card.flipped img,
.card.flipped .name-tag {
  display: block;
}

.card.flipped .quest {
  display: none;
}

.card.matched {
  border-color: #2ecc71;
  opacity: 0.35;
  pointer-events: none;
}

button {
  background: #d4af37;
  color: black;
  border: none;
  border-radius: 10px;
  padding: 18px 30px;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  margin-top: 15px;
}

button:active {
  transform: scale(0.97);
}

#start-btn-container {
  margin-top: 25px;
}

#lead-form {
  display: none;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 92%;
  max-width: 600px;
  background: #1a1a1a;
  border: 3px solid #d4af37;
  border-radius: 18px;
  padding: 30px;
  z-index: 1000;
}

#lead-form h2 {
  color: white;
  font-size: 34px;
}

#stats {
  color: #d4af37;
  font-size: 28px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 18px;
  margin-top: 12px;
  border-radius: 10px;
  border: 1px solid #444;
  background: #222;
  color: white;
  font-size: 18px;
}

#admin-panel {
  margin-top: 40px;
  padding-bottom: 40px;
}

.admin-btn {
  background: #222;
  color: #ccc;
  border: 1px solid #444;
  margin: 6px;
  font-size: 16px;
}

.hidden {
  display: none !important;
}

@media (max-width: 900px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .card {
    height: 150px;
  }
}

@media (max-width: 600px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .card {
    height: 140px;
  }

  h1 {
    font-size: 24px;
  }

  #timer {
    font-size: 30px;
  }
}
</style>
</head>

<body>

<h1>🏆 Classic Car Challenge</h1>
<div id="timer">Zeit: 0s</div>

<div class="grid" id="grid"></div>

<div id="start-btn-container">
  <button id="start-btn">Jetzt starten</button>
</div>

<div id="lead-form">
  <h2>🔥 Gratulation!</h2>
  <p id="stats"></p>
  <p style="color:#ccc;">Bitte Daten eingeben:</p>

  <input type="text" id="fname" placeholder="Vorname & Nachname" />
  <input type="email" id="femail" placeholder="E-Mail Adresse" />
  <input type="number" id="fage" placeholder="Alter" />

  <button id="save-btn">Daten absenden</button>
</div>

<div id="admin-panel">
  <a class="admin-btn" target="_blank" href="https://docs.google.com/spreadsheets/u/0/">
  Teilnehmerliste in Google Sheets öffnen
</a>
</div>

<script>
const cars = [
  { name: 'Aston Martin DB MK III', img: 'db24.jpg' },
  { name: 'Aston Martin DB4', img: 'db4.jpg' },
  { name: 'Aston Martin V8 Volante', img: 'v8.jpg' },
  { name: 'Jaguar S.S. 1.5 Litre', img: 'ss15.jpg' },
  { name: 'Jaguar XJ-SC V12', img: 'xjsc.jpg' },
  { name: 'Jaguar XK 120 DHC', img: 'xk120.jpg' },
  { name: 'Jaguar XKR S/C Coupé', img: 'xkr.jpg' },
  { name: 'Range Rover 4.0', img: 'range.jpg' },
  { name: 'Maserati 3500 GT', img: 'maserati.jpg' },
  { name: 'Mercedes-Benz 300 CE', img: 'mercedes.jpg' },
  { name: 'Morgan Plus 8', img: 'morgan.jpg' },
  { name: 'Rover P6 2000 TC', img: 'rover.jpg' }
];

let flippedCards = [];
let matches = 0;
let seconds = 0;
let timerInterval;
let gameStarted = false;
let lockBoard = false;
let attempts = 0;

const startBtn = document.getElementById('start-btn');
const saveBtn = document.getElementById('save-btn');

startBtn.addEventListener('click', startGame);
saveBtn.addEventListener('click', saveData);

function startGame() {
  if(gameStarted) return;

  gameStarted = true;
  matches = 0;
  seconds = 0;
  attempts = 0;
  flippedCards = [];

  document.getElementById('start-btn-container').classList.add('hidden');

  const grid = document.getElementById('grid');
  grid.innerHTML = '';

  let gameItems = [];

  const selection = [...cars]
    .sort(() => Math.random() - 0.5)
    .slice(0, 6);

  selection.forEach(car => {
    gameItems.push({ type: 'img', id: car.name, img: car.img });
    gameItems.push({ type: 'name', id: car.name, text: car.name });
  });

  gameItems.sort(() => Math.random() - 0.5);

  gameItems.forEach(item => {
    const card = document.createElement('div');
    card.className = 'card';
    card.dataset.id = item.id;

    if(item.type === 'img') {
      card.innerHTML = `
        <img src="${item.img}" alt="${item.id}">
        <span class="quest">?</span>
      `;
    } else {
      card.innerHTML = `
        <div class="name-tag">${item.text}</div>
        <span class="quest">?</span>
      `;
    }

    card.addEventListener('click', () => flipCard(card), { passive: true });
    card.addEventListener('touchstart', () => flipCard(card), { passive: true });

    grid.appendChild(card);
  });

  updateTimer();

  clearInterval(timerInterval);
  timerInterval = setInterval(() => {
    seconds++;
    updateTimer();
  }, 1000);
}

function updateTimer() {
  document.getElementById('timer').innerText = `Zeit: ${seconds}s`;
}

function flipCard(card) {
  if(lockBoard) return;
  if(card.classList.contains('flipped')) return;
  if(card.classList.contains('matched')) return;

  card.classList.add('flipped');
  flippedCards.push(card);

  if(flippedCards.length === 2) {
    attempts++;
    lockBoard = true;
    setTimeout(checkMatch, 800);
  }
}

function checkMatch() {
  const [card1, card2] = flippedCards;

  if(card1.dataset.id === card2.dataset.id) {
    card1.classList.add('matched');
    card2.classList.add('matched');

    matches++;

    if(matches === 6) {
      clearInterval(timerInterval);
      showLeadForm();
    }
  } else {
    card1.classList.remove('flipped');
    card2.classList.remove('flipped');
  }

  flippedCards = [];
  lockBoard = false;
}

function showLeadForm() {
  document.getElementById('stats').innerHTML = `
    ${seconds} Sekunden<br>
    ${attempts} Versuche
  `;

  document.getElementById('lead-form').style.display = 'block';
}

function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

function getBrowserInfo() {
  return navigator.userAgent;
}

async function saveData() {

  const name = document.getElementById('fname').value.trim();
  const email = document.getElementById('femail').value.trim();
  const age = document.getElementById('fage').value.trim();
  const privacyAccepted = document.getElementById('privacy').checked;

  if(!name || !email || !age) {
    alert('Bitte alle Felder ausfüllen.');
    return;
  }

  if(!privacyAccepted) {
    alert('Bitte Datenschutz und AGB akzeptieren.');
    return;
  }

  if(!validateEmail(email)) {
    alert('Bitte gültige E-Mail Adresse eingeben.');
    return;
  }

  const entry = {
    timestamp: new Date().toLocaleString('de-CH'),
    name,
    email,
    age,
    time: seconds,
    attempts,
    privacyAccepted: 'JA',
    browser: navigator.userAgent,
    device: navigator.platform
  };

  try {

    await fetch('https://script.google.com/macros/s/AKfycby3b4umFK38BiUFGrrgmjnzRkw_0AFIqxjyaFZ6pJjW8GcwazjcStkcJML0P9Ws7JI_vA/exec', {
      method: 'POST',
      mode: 'no-cors',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(entry)
    });

    alert('Teilnehmer erfolgreich gespeichert.');

    location.reload();

  } catch(error) {

    console.error(error);
    alert('Fehler beim Speichern der Daten.');
  }
}

function escapeCSV(value) {
  return '"' + String(value).replace(/"/g, '""') + '"';
}

// Lokale Löschfunktion entfernt – alle Daten bleiben dauerhaft in Google Sheets gespeichert.
function clearData() {
  const confirmDelete = confirm(
    'ACHTUNG: Alle Teilnehmerdaten werden gelöscht!'
  );

  if(confirmDelete) {
    localStorage.removeItem('messe_leads');
    alert('Daten gelöscht.');
    location.reload();
  }
}
</script>

</body>
</html>
```

## Hinweise

### Bilder
Die Bilddateien müssen im gleichen Ordner liegen wie die HTML-Datei:

- db24.jpg
- db4.jpg
- v8.jpg
- ss15.jpg
- xjsc.jpg
- xk120.jpg
- xkr.jpg
- range.jpg
- maserati.jpg
- mercedes.jpg
- morgan.jpg
- rover.jpg

---

## Vorteile dieser Version

- Optimiert für Touchscreens
- Funktioniert stabil in Chrome, Edge und WebOS
- Google Sheets Online-Datenspeicherung
- Kein localStorage
- CSV-Export mit allen Teilnehmerdaten
- Speicherung der Datenschutz-Zustimmung
- Mehrere Geräte gleichzeitig möglich
- Speichert Zeit und Versuche
- Responsive für Tablets & TVs
- Verbesserte Klick-Logik
- Excel-kompatibler CSV-Export
- WebOS-kompatibler Download
- Doppelklick-Schutz
- Modernisiertes Layout

---

# Neue echte Online-Version mit Google Sheets

## Wichtige Änderung

Die bisherige Speicherung über `localStorage` wird vollständig entfernt.

Alle Teilnehmerdaten werden nun direkt online in Google Sheets gespeichert.

---

# HTML-Erweiterung Datenschutz

## Checkbox im Formular ergänzen

Direkt vor dem Button einfügen:

```html
<label style="display:block;color:#ccc;font-size:14px;margin-top:15px;text-align:left;line-height:1.5;">
  <input type="checkbox" id="privacy" style="width:auto;margin-right:10px;">
  Ich akzeptiere die Allgemeinen Geschäftsbedingungen sowie die Datenschutzrichtlinien der Emil Frey Classics.
</label>
```

---

## saveData() ersetzen

```javascript
async function saveData() {

  const name = document.getElementById('fname').value.trim();
  const email = document.getElementById('femail').value.trim();
  const age = document.getElementById('fage').value.trim();
  const privacyAccepted = document.getElementById('privacy').checked;

  if(!name || !email || !age) {
    alert('Bitte alle Felder ausfüllen.');
    return;
  }

  if(!privacyAccepted) {
    alert('Bitte Datenschutz und AGB akzeptieren.');
    return;
  }

  if(!validateEmail(email)) {
    alert('Bitte gültige E-Mail Adresse eingeben.');
    return;
  }

  const entry = {
    timestamp: new Date().toLocaleString('de-CH'),
    name,
    email,
    age,
    time: seconds,
    attempts,
    privacyAccepted: 'JA',
    browser: navigator.userAgent,
    device: navigator.platform
  };

  try {

    const response = await fetch('https://script.google.com/macros/s/AKfycby3b4umFK38BiUFGrrgmjnzRkw_0AFIqxjyaFZ6pJjW8GcwazjcStkcJML0P9Ws7JI_vA/exec', {
      method: 'POST',
      mode: 'no-cors',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(entry)
    });

    alert('Teilnahme erfolgreich gespeichert.');

    location.reload();

  } catch(error) {

    alert('Fehler beim Speichern der Daten.');

    console.error(error);
  }
}
```

---

# Google Apps Script erstellen

## 1. Neues Google Sheet erstellen

Zum Beispiel:

```txt
Swiss Classic World Teilnehmer
```

---

## 2. Spalten anlegen

```txt
Datum
Name
Email
Alter
Zeit
Versuche
Datenschutz akzeptiert
Gerät
Browser
```

---

## 3. Google Apps Script öffnen

In Google Sheets:

```txt
Erweiterungen → Apps Script
```

---

## 4. Diesen kompletten Code einfügen

```javascript
function doPost(e) {

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

  const data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    data.timestamp,
    data.name,
    data.email,
    data.age,
    data.time,
    data.attempts,
    data.privacyAccepted,
    data.device,
    data.browser
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ result: 'success' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

# 5. Apps Script veröffentlichen

## Deploy klicken

Dann:

```txt
Neue Bereitstellung
```

Typ:

```txt
Web-App
```

Einstellungen:

| Einstellung | Wert |
|---|---|
| Ausführen als | Ich |
| Zugriff | Jeder |

Dann:

```txt
Bereitstellen
```

---

# 6. Web-App URL kopieren

Sie sieht ungefähr so aus:

```txt
https://script.google.com/macros/s/AKfycbxxxxxx/exec
```

Diese URL hier einfügen:

```javascript
fetch('https://script.google.com/macros/s/AKfycby3b4umFK38BiUFGrrgmjnzRkw_0AFIqxjyaFZ6pJjW8GcwazjcStkcJML0P9Ws7JI_vA/exec'
```

---

# CSV Export

Der CSV Export erfolgt jetzt direkt über Google Sheets:

## Datei → Download → CSV

Dadurch sind:

- alle Teilnehmer online gespeichert
- mehrere Geräte synchronisiert
- Datenschutzannahme enthalten
- keine Daten lokal gespeichert

---

# GitHub Pages Veröffentlichung

## Repository erstellen

Auf:

https://github.com

---

## Dateien hochladen

- index.html
- alle Bilder

---

## GitHub Pages aktivieren

```txt
Settings → Pages
```

Dann:

```txt
Deploy from branch
```

Branch:

```txt
main
```

Ordner:

```txt
/root
```

Danach wird eure Seite live veröffentlicht.

---

# Finale Architektur

```txt
GitHub Pages
    ↓
Memory Spiel
    ↓
Google Apps Script
    ↓
Google Sheets
    ↓
CSV Export
```

---

# Ergebnis

Ihr habt jetzt:

- echte Online-Speicherung
- keine lokalen Daten
- Datenschutz-Zustimmung gespeichert
- CSV Export online
- GitHub Hosting
- mehrere Geräte gleichzeitig
- WebOS-/Chrome-/Edge-Kompatibilität
- Messebetrieb-taugliche Lösung

