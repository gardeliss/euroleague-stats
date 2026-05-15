# Euroleague CORS Proxy — Cloudflare Worker

Ένας απλός Cloudflare Worker που proxάρει requests στο
`live.euroleague.net` API και προσθέτει CORS headers,
ώστε το browser app σου (GitHub Pages) να μπορεί να φέρει δεδομένα.

---

## Deploy (βήμα-βήμα)

### 1. Δημιούργησε λογαριασμό Cloudflare
Αν δεν έχεις: https://dash.cloudflare.com/sign-up  
Το free plan αρκεί (100,000 requests/ημέρα).

### 2. Δημιούργησε Worker
1. Πήγαινε στο **Workers & Pages** (αριστερό menu)
2. Κλικ **Create** → **Create Worker**
3. Δώσε ένα όνομα, π.χ. `euroleague-proxy`
4. Κλικ **Deploy** (deploy ό,τι υπάρχει, δεν πειράζει)

### 3. Επικόλλησε τον κώδικα
1. Στη σελίδα του worker → κλικ **Edit code**
2. Σβήσε τα πάντα από τον editor
3. Επικόλλησε το περιεχόμενο του `worker.js`
4. **Σημαντικό:** Βρες τη γραμμή `ALLOWED_ORIGINS` και πρόσθεσε το GitHub Pages URL σου:
   ```js
   "https://<your-username>.github.io",
   ```
5. Κλικ **Deploy**

### 4. Πάρε το URL σου
Μετά το deploy θα δεις κάτι σαν:
```
https://euroleague-proxy.<your-subdomain>.workers.dev
```
Αυτό είναι το base URL που θα χρησιμοποιεί η web app σου.

---

## Endpoints που proxάρονται

| Endpoint | Παράμετροι | Περιγραφή |
|---|---|---|
| `/api/Standings` | `seasoncode=E2024` | Βαθμολογία |
| `/api/Teams` | `seasoncode=E2024` | Λίστα ομάδων |
| `/api/TeamStatistics` | `seasoncode=E2024&teamcode=PAO` | Stats ομάδας |
| `/api/PlayerStatistics` | `seasoncode=E2024&teamcode=PAO` | Stats παικτών |
| `/api/Results` | `seasoncode=E2024` | Αποτελέσματα |
| `/api/Schedule` | `seasoncode=E2024&gameday=1` | Πρόγραμμα |

**Season codes:** `E2024` = σεζόν 2024-25, `E2023` = 2023-24, κτλ.

**Team codes (παραδείγματα):**
- `PAO` = Panathinaikos
- `OLY` = Olympiacos
- `MAD` = Real Madrid
- `BAR` = Barcelona
- `ULK` = Fenerbahce

---

## Test με curl

```bash
# Standings σεζόν 2024-25
curl "https://euroleague-proxy.<your-subdomain>.workers.dev/api/Standings?seasoncode=E2024"

# Teams σεζόν 2024-25
curl "https://euroleague-proxy.<your-subdomain>.workers.dev/api/Teams?seasoncode=E2024"

# Player stats Panathinaikos
curl "https://euroleague-proxy.<your-subdomain>.workers.dev/api/PlayerStatistics?seasoncode=E2024&teamcode=PAO"
```

---

## Χρήση από JavaScript (fetch)

```js
const PROXY = "https://euroleague-proxy.<your-subdomain>.workers.dev";

async function getStandings(season = "E2024") {
  const res = await fetch(`${PROXY}/api/Standings?seasoncode=${season}`);
  const data = await res.json();
  return data;
}

async function getTeamPlayers(season, teamCode) {
  const res = await fetch(
    `${PROXY}/api/PlayerStatistics?seasoncode=${season}&teamcode=${teamCode}`
  );
  const data = await res.json();
  return data;
}
```

---

## Σημειώσεις

- **Cache:** Οι απαντήσεις cache-άρονται για 5 λεπτά (Cloudflare edge cache)
- **Rate limits:** Free plan = 100K requests/ημέρα — αρκετά για personal use
- **Whitelist:** Μόνο τα endpoints της λίστας `ALLOWED_ENDPOINTS` επιτρέπονται
- **CORS origins:** Μόνο τα origins της λίστας `ALLOWED_ORIGINS` γίνονται δεκτά
