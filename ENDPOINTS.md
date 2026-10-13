# My Festival API — endpoint-dokumentti / endpoint document

> FI: Täytä tämä pohja ja palauta se Moodleen (Markdown tai PDF, 1–2 sivua). Korvaa kaikki `<…>`-kohdat.
> Esimerkkivastaukset kopioit oman julkisen rajapintasi oikeista vastauksista (selain tai `curl.exe`/`curl`).
>
> EN: Fill in this template and submit it to Moodle (Markdown or PDF, 1–2 pages). Replace every `<…>`.
> Copy the example responses from your own public API's real responses (browser or `curl.exe`/`curl`).

**Tekijä / Author:** <Tuukka Järvinen>
**Julkinen osoite / Public URL:** `https://festival-api-festival-tuukkaja.2.rahtiapp.fi`
**GitHub:** `https://github.com/Tukezzi/my-festival-api-TJ`
**Tarkistimen tulos / Checker result:** `<20>/<20> checks passed` (liitä koko tuloste palautukseen / attach the full output)

[PASS] GET /api/artists returns 200
[PASS]   Content-Type is application/json
[PASS]   response is a non-empty JSON list
[PASS]   items have artist_id and name
[PASS] GET /api/days returns day_id + date
[PASS] GET /api/stages returns stage_id + name
[PASS] GET /api/artists/10/events returns a JSON list
[PASS] GET unknown artist returns 404
[PASS] GET /api/timetable returns a JSON list
[PASS] GET /api/timetable?day=2025-07-05 returns only that day
[PASS]   rows are in time order (starts)
[PASS] GET timetable with bad date returns 400
[PASS] POST with missing fields returns 400
[PASS] POST with bad foreign key returns 400
[PASS] POST with bad start_dt format returns 400
[PASS] POST with valid body returns 201 + event_id
[PASS]   new event is returned by GET /api/artists/{id}/events
[PASS] DELETE /api/events/128 returns 204
[PASS]   deleted event is gone
[PASS] DELETE same event again returns 404

20/20 checks passed
---

## 1. Endpointit / Endpoints

| Metodi / Method | Polku / Path | Onnistuu / Success | Virheet / Errors |
|---|---|---|---|
| GET | `/api/artists` | 200 | – |
| GET | `/api/days` | 200 | – |
| GET | `/api/stages` | 200 | – |
| GET | `/api/artists/{id}/events` | 200 | 404 |
| GET | `/api/timetable?day=YYYY-MM-DD` | 200 | 400 |
| POST | `/api/events` | 201 | 400 |
| DELETE | `/api/events/{id}` | 204 | 404 |

## 2. Esimerkit / Examples

Kirjoita jokaisesta endpointista esimerkkikutsu ja (lyhennetty) vastaus. Malli:
For every endpoint, write an example call and a (shortened) response. Example:

### GET /api/artists

```bash
curl https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/artists
```
```json
[{"artist_id":10,"country":"UK","name":"Adele"},{"artist_id":17,"country":"FIN","name":"Alma"}]...
```

### GET /api/artists/{id}/events
```bash
curl https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/artists/10/events
```
```json
[{"day":"2025-07-05","event_id":1,"stage":"Main A","starts":"14:15:00"},{"day":"2025-07-05","event_id":27,"stage":"Club","starts":"17:15:00"}]....
```
Tuntematon id / unknown id → `<tilakoodi ja runko / status code and body>`

```bash
curl https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/artists/10000/events
```
```json
{"error":"artist 10000 not found"}
```

### GET /api/timetable?day=YYYY-MM-DD
```bash
curl https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/timetable?day=2025-07-06
```
```json
[{"artist":"Imagine Dragons","date":"2025-07-06","ends":"14:40:00","stage":"Club","starts":"14:15:00"},{"artist":"Adele","date":"2025-07-06","ends":"15:00:00","stage":"Main A","starts":"14:15:00"}]...
```
Virheellinen päivämäärä / invalid date → `<tilakoodi ja runko / status code and body>`
```bash
curl https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/timetable?day=2025-4323
```
```json
{"error":"invalid date, use YYYY-MM-DD"}
```

### POST /api/events
```bash
curl -i -X POST https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/events \
     -H "Content-Type: application/json" \
     -d '{"artist_id": 1, "stage_id": 1, "day_id": 1, "start_dt": "2026-07-10 20:00:00"}'
```
```json
<artist_id         : 1
day_id            : 1
duration_min      : 45
end_dt            : 2026-07-10 20:45:00
event_id          : 127
event_type_id     : 1
previous_event_id :
stage_id          : 1
start_dt          : 2026-07-10 20:00:00>
```
Puuttuva kenttä / missing field → `{"error":"missing fields: stage_id, day_id"}` · Olematon artist_id / unknown artist_id → `{"error":"integrity error: UNIQUE constraint failed: Event.day_id, Event.stage_id, Event.start_dt"}`

### DELETE /api/events/{id}
```bash
curl -X DELETE https://festival-api-festival-tuukkaja.2.rahtiapp.fi/api/events/10
```
Onnistuu / success → `<>` · Tuntematon id / unknown id → `{"error":"event 10 not found"}`

## 3. Julkaisu Rahtiin / Deploying to Rahti (3–6 lausetta / sentences)

Rahti hakee koodin GitHub-reposta, ja rakensi Dockerfilen perusteella kontti-imagen. Rakennetusta imagesta käynnistyi Pod, jota Deployment ylläpitää, ja Route tarjoaa sille julkisen HTTPS-osoitteen. Osoitteen löysin Topology-näkymästä nuolikuvakkeesta. Sovellusta ajaa gunicorn portisa 8080.

<FI: Kuvaa omin sanoin build → deployment → route: mitä Rahti teki repostasi ja mistä löysit julkisen osoitteen.
Mainitse yksi ongelma, johon törmäsit, ja miten ratkaisit sen.>
<EN: Describe build → deployment → route in your own words: what Rahti did with your repo and where you found
the public address. Mention one problem you ran into and how you solved it.>

## 4. Tietoturva / Security (2–4 lausetta / sentences)

SQL-injektioita ehkäistään käyttämällä parametrisoituja SQL-kyselyitä, jolloin käyttäjän syötteitä, kuten id, day ja POST-kenttiä, ei yhdistetä suoraan SQL-komentoihin. Lisäksi päivämäärät ja start_dt tarkistetaan, ja POST-pyynnöissä varmistetaan syötteiden oikeat tietotyypit ennen tietokantaan tallentamista.

Ennen rajapinnan ottamista tuotantokäyttöön siihen tulisi lisätä käyttäjien tunnistautuminen ja käyttöoikeuksien tarkistus erityisesti POST- ja DELETE-reiteille. Näin estetään luvattomat muutokset ohjelmatietoihin. Lisäksi tarvitaan lokitus ja tapahtumien seuranta. HTTPS-suojaus on jo käytössä Ranchin edge-reitittimen ansiosta.

<FI: Mitä tämä rajapinta tarvitsisi ennen oikeaa tuotantokäyttöä? Miten varmistit, ettei SQL-injektio onnistu?>
<EN: What would this API need before real production use? How did you make sure SQL injection is not possible?>
