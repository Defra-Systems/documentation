# CassaBackend - API Reference

Base URL: `https://backend.cassaconnessa.com/api/v1`

---

## Autenticazione

Tutti gli endpoint `/license/**` richiedono header:
```
Authorization: Bearer {accessToken}
```

### Formato errori

```json
{
  "status": 404,
  "message": "Descrizione errore",
  "timestamp": "2026-02-15T10:30:00"
}
```

| Codice | Quando |
|---|---|
| 400 | Dati non validi |
| 401 | Credenziali errate / token scaduto |
| 403 | Account disabilitato |
| 404 | Risorsa non trovata |
| 409 | Conflitto (duplicato, stato errato) |
| 429 | Account bloccato (troppi tentativi) |
| 500 | Errore interno |

---

## 1. Autenticazione (`/auth`)

### POST `/auth/login/client`

```json
{ "email": "user@gmail.com", "password": "Password123!" }
```

**Response 200 (senza 2FA):**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "uuid",
  "userId": "uuid",
  "userType": "CLIENT"
}
```

**Response 200 (con 2FA):**
```json
{ "requiresTwoFactor": true, "userId": "uuid" }
```

---

### POST `/auth/verify-2fa`

```json
{ "userId": "uuid", "userType": "CLIENT", "code": "123456" }
```

**Response 200:** stessa struttura di login.

---

### POST `/auth/refresh`

```json
{ "refreshToken": "uuid" }
```

**Response 200:**
```json
{ "accessToken": "eyJ...", "refreshToken": "new-uuid" }
```

---

### POST `/auth/logout`

```json
{ "refreshToken": "uuid" }
```

**Response:** `204 No Content`

---

### POST `/auth/password-reset/request`

```json
{ "email": "user@example.com", "userType": "CLIENT" }
```

---

### POST `/auth/password-reset/reset`

```json
{ "token": "reset-token-uuid", "newPassword": "newPassword123" }
```

---

### POST `/auth/setup-2fa` *(richiede Bearer token)*

Nessun body. L'email viene letta dal JWT.

**Response 200:**
```json
{
  "secret": "BASE32SECRET",
  "otpAuthUri": "otpauth://totp/CassaConnessa:user@example.com?secret=BASE32SECRET&issuer=CassaConnessa"
}
```

---

### POST `/auth/confirm-2fa` *(richiede Bearer token)*

userId e userType vengono letti dal JWT — non vanno nel body.

```json
{ "secret": "BASE32SECRET", "code": "123456" }
```

**Response 200:**
```json
{ "message": "2FA enabled" }
```

---

## 2. Profilo e Attivita' (`/license`)

### GET `/license/verify?publicCode=ABC123` *(nessun auth)*

Verifica la validita' di un codice pubblico licenza.

**Response 200:**
```json
{ "valid": true, "clientId": "uuid", "expiresAt": "2027-12-31" }
```

---

### GET `/license/profile`

Profilo completo dell'utente autenticato.

**Response 200:**
```json
{
  "uuid": "uuid",
  "name": "Mario",
  "surname": "Rossi",
  "email": "mario@example.com",
  "licenseExpireAt": "2027-12-31",
  "enabled": true,
  "twoFactorEnabled": false,
  "modules": ["tavoli", "food_cost", "fidelity"],
  "activities": [
    {
      "id": "uuid",
      "publicCode": "ABC123",
      "name": "Pizzeria Mario",
      "vatID": "IT12345678901",
      "type": "SOLE_PROPRIETORSHIP",
      "businessName": "Mario Rossi",
      "address": "Via Roma 1",
      "city": "Milano",
      "postalCode": "20100",
      "province": "MI",
      "phone": "+39 02 1234567",
      "email": "info@pizzeriamario.it",
      "sdiCode": "XXXXXXX",
      "pec": "mario@pec.it",
      "loyaltyConfig": {
        "spendThreshold": 5.00,
        "spendAmount": 10.00,
        "pointsAwarded": 1,
        "enabled": true
      }
    }
  ]
}
```

> `modules` contiene i moduli abilitati per la licenza. Se non configurati, il backend restituisce tutti: `["tavoli", "food_cost", "fidelity"]`.

---

### PUT `/license/{userId}/activities/{activityId}`

```json
{
  "businessName": "Mario Rossi S.r.l.",
  "address": "Via Roma 1",
  "city": "Milano",
  "postalCode": "20100",
  "province": "MI",
  "phone": "+39 02 1234567",
  "email": "info@pizzeriamario.it",
  "sdiCode": "XXXXXXX",
  "pec": "mario@pec.it"
}
```

---

### GET `/license/{userId}/activities/{activityId}/loyalty-config`

**Response 200:**
```json
{ "spendThreshold": 5.00, "spendAmount": 10.00, "pointsAwarded": 1, "enabled": true }
```

---

### PUT `/license/{userId}/activities/{activityId}/loyalty-config`

```json
{ "spendThreshold": 5.00, "spendAmount": 10.00, "pointsAwarded": 1, "enabled": true }
```

> Ogni `spendAmount` euro spesi = `pointsAwarded` punti, con soglia minima `spendThreshold`.

---

## 3. Categorie (`/license/{userId}/activities/{activityId}/categories`)

### POST - Crea categoria

```json
{
  "name": "Pizze",
  "description": "Tutte le pizze",
  "parentCategoryId": null,
  "defaultVat": 10
}
```

**Response:** `201 Created`

---

### GET - Tutte le categorie

### GET `/root` - Solo categorie radice

### GET `/{categoryId}` - Singola categoria

### GET `/{categoryId}/subcategories` - Sottocategorie

---

### PUT `/{categoryId}` - Aggiorna categoria

```json
{
  "name": "Pizze Speciali",
  "description": "Pizze premium",
  "parentCategoryId": null,
  "defaultVat": 10,
  "color": "#FF0000",
  "icon": "pizza",
  "imageUrl": "https://...",
  "sortOrder": 1,
  "vatNature": null,
  "visibleRetail": true,
  "visibleRestaurant": true,
  "visibleTakeaway": false,
  "hidden": false,
  "active": true,
  "fiscal": true
}
```

---

### DELETE `/{categoryId}` - Elimina categoria

**Response:** `204 No Content` — errore `409` se ha prodotti associati.

---

## 4. Prodotti (`/license/{userId}/activities/{activityId}/products`)

### POST - Crea prodotto

```json
{
  "categoryId": "uuid",
  "name": "Margherita",
  "description": "Pizza classica",
  "barcode": "1234567890",
  "imageUrl": "https://...",
  "color": "#FFA500",
  "basePrice": 8.00,
  "iva": 10
}
```

**Response:** `201 Created`

---

### GET - Tutti i prodotti

### GET `?categoryId={uuid}` - Per categoria

### GET `?barcode={code}` - Per barcode

### GET `/{productId}` - Singolo prodotto

---

### PUT `/{productId}` - Aggiorna prodotto

```json
{
  "name": "Margherita XL",
  "description": "Pizza classica grande",
  "barcode": "1234567890",
  "imageUrl": "https://...",
  "color": "#FFA500",
  "basePrice": 10.00,
  "priceList2": 12.00,
  "priceList3": 11.00,
  "priceList4": 0,
  "iva": 10,
  "sortOrder": 1,
  "vatNature": null,
  "active": true,
  "fiscal": true
}
```

---

### DELETE `/{productId}` - Elimina prodotto

**Response:** `204 No Content`

---

### POST `/{productId}/ingredients` - Aggiungi ingrediente (magazzino)

```json
{ "warehouseItemId": "uuid", "quantityUsed": 0.3 }
```

**Response:** `201 Created`

---

### GET `/{productId}/ingredients`

### DELETE `/{productId}/ingredients/{warehouseItemId}` → `204 No Content`

---

### POST `/{productId}/extra-costs` - Aggiungi costo extra (food cost)

```json
{ "name": "Imballaggio", "costPerUnit": 0.30 }
```

**Response:** `201 Created`

---

### GET `/{productId}/extra-costs`

### PUT `/{productId}/extra-costs/{id}`

```json
{ "name": "Imballaggio XL", "costPerUnit": 0.50 }
```

### DELETE `/{productId}/extra-costs/{id}` → `204 No Content`

---

## 5. Varianti (`/license/{userId}/activities/{activityId}/variants`)

### POST - Crea variante

```json
{
  "name": "Extra mozzarella",
  "pricePlus": 2.00,
  "priceMinus": 0,
  "categoryId": "uuid-or-null",
  "productId": null
}
```

> Se `categoryId` e' valorizzato, la variante si applica a tutti i prodotti di quella categoria. Se `productId` e' valorizzato, solo a quel prodotto.

**Response:** `201 Created`

---

### GET - Lista varianti

Filtri opzionali: `?categoryId={uuid}` oppure `?productId={uuid}`

### GET `/{variantId}` - Singola variante

---

### PUT `/{variantId}` - Aggiorna variante

```json
{
  "name": "Extra mozzarella di bufala",
  "pricePlus": 2.50,
  "priceMinus": 0,
  "categoryId": null,
  "productId": "uuid",
  "sortOrder": 1,
  "active": true
}
```

---

### DELETE `/{variantId}` → `204 No Content`

---

## 6. Magazzino (`/license/{userId}/activities/{activityId}/warehouse`)

### POST - Crea articolo

```json
{ "name": "Farina 00", "quantity": 50.0, "costPrice": 1.20 }
```

**Response:** `201 Created`

---

### GET - Tutti gli articoli

### GET `/below-minimum` - Articoli sotto soglia minima

### GET `/{itemId}` - Singolo articolo

---

### PUT `/{itemId}` - Aggiorna articolo

```json
{
  "name": "Farina 00",
  "description": "Farina tipo 00",
  "barcode": "123456",
  "unit": "kg",
  "costPrice": 1.50,
  "minQuantity": 5.0,
  "active": true
}
```

---

### POST `/{itemId}/add-stock`

```json
{ "amount": 25.0 }
```

---

### POST `/{itemId}/deduct-stock`

```json
{ "amount": 10.0 }
```

Errore `400` se quantita' insufficiente.

---

### DELETE `/{itemId}` → `204 No Content`

---

## 7. Fidelizzazione (`/license/{userId}/activities/{activityId}/loyalty`)

### Categorie loyalty

#### POST `/categories` - Crea categoria loyalty

```json
{ "name": "VIP", "color": "#FFD700" }
```

`color` opzionale.

**Response:** `201 Created`

---

#### GET `/categories` - Lista categorie

#### DELETE `/categories/{categoryId}` → `204 No Content`

---

### Clienti

#### POST `/customers` - Crea cliente

```json
{
  "name": "Anna",
  "surname": "Verdi",
  "email": "anna@example.com",
  "phone": "+39123456789",
  "birthDate": "1990-05-15",
  "gender": "F"
}
```

`birthDate` e `gender` opzionali.

**Response:** `201 Created` — errore `400` se email o telefono gia' registrati.

---

#### GET `/customers` - Lista clienti

`?activeOnly=true` per filtrare solo attivi.

---

#### GET `/customers/search`

- `?email=anna@example.com` → cliente singolo
- `?phone=+39123456789` → cliente singolo
- `?name=Anna` → array di clienti

---

#### GET `/customers/{customerId}`

#### PUT `/customers/{customerId}` - Aggiorna cliente

```json
{
  "name": "Anna",
  "surname": "Verdi",
  "email": "anna@example.com",
  "phone": "+39123456789",
  "active": true,
  "activityIds": ["uuid1"],
  "categoryIds": ["cat-uuid1"],
  "birthDate": "1990-05-15",
  "gender": "F"
}
```

Tutti i campi opzionali (null = non modificare).

---

#### POST `/customers/{customerId}/points/add`

```json
{ "amount": 10 }
```

#### POST `/customers/{customerId}/points/subtract`

```json
{ "amount": 5 }
```

Errore `400` se punti insufficienti.

---

#### POST `/customers/{customerId}/credit/add`

```json
{ "amount": 50.00 }
```

#### POST `/customers/{customerId}/credit/use`

```json
{ "amount": 20.00 }
```

Errore `400` se credito insufficiente.

---

#### PUT `/customers/{customerId}/discount`

```json
{ "discount": 15 }
```

---

#### POST `/customers/{customerId}/rewards/{rewardId}/redeem`

Riscatta un premio sottraendo i punti necessari. Se tipo `CREDIT`, aggiunge il valore come credito al cliente.

**Response 200:** LoyaltyCustomer aggiornato.

---

#### DELETE `/customers/{customerId}` → `204 No Content`

---

### Campagne

#### POST `/campaigns` - Crea campagna sconto

```json
{
  "name": "Estate 2026",
  "discountPercent": 20,
  "global": false,
  "categoryIds": ["uuid1", "uuid2"],
  "startDate": "2026-06-01T00:00:00",
  "endDate": "2026-08-31T23:59:59"
}
```

- `global: true` → sconto su tutto il catalogo
- `endDate: null` → nessuna scadenza

**Response:** `201 Created`

---

#### GET `/campaigns` - Campagne attive

#### DELETE `/campaigns/{campaignId}` → `204 No Content`

---

### Premi

#### POST `/rewards` - Crea premio

```json
{
  "pointsCost": 100,
  "rewardType": "CREDIT",
  "value": 10.00,
  "description": "10 euro di credito"
}
```

| rewardType | Effetto al riscatto |
|---|---|
| `CREDIT` | Aggiunge `value` euro come credito |
| `PURCHASE_DISCOUNT` | `value` = percentuale sconto |
| `VOUCHER` | `value` = valore del buono |

**Response:** `201 Created`

---

#### GET `/rewards` - Premi attivi

---

## 8. Scontrini (`/license/{userId}/activities/{activityId}/receipts`)

### POST `/preview` - Anteprima con sconti calcolati

Chiamare prima di battere lo scontrino. Il backend calcola automaticamente gli sconti applicabili (campagne attive + sconto personale del cliente loyalty) e restituisce gli articoli con `discountPercent` valorizzato.

**Request:**
```json
{
  "items": [
    {
      "productId": "uuid",
      "quantity": 2,
      "unitPrice": 8.00,
      "vat": 10,
      "discountEuro": null
    }
  ],
  "loyaltyCustomerId": "uuid-or-null"
}
```

> `productName`, `categoryId`, `categoryLabel` sono opzionali: il backend li popola automaticamente dal prodotto.

**Response 200:** array di ReceiptItem con sconti applicati:
```json
[
  {
    "productId": "uuid",
    "productName": "Margherita",
    "categoryId": "cat-uuid",
    "categoryLabel": "Pizze",
    "quantity": 2,
    "unitPrice": 8.00,
    "vat": 10,
    "discountPercent": 10,
    "discountEuro": null
  }
]
```

---

### POST - Salva scontrino

Il backend ricalcola gli sconti prima di salvare. Non e' necessario passare `discountPercent` — viene sempre ignorato e ricalcolato. Passare solo `discountEuro` per sconti manuali del cassiere.

**Request:**
```json
{
  "documentType": "FISCAL",
  "items": [
    {
      "productId": "uuid",
      "quantity": 2,
      "unitPrice": 8.00,
      "vat": 10,
      "discountEuro": null,
      "customName": "Nome temporaneo visualizzato"
    }
  ],
  "loyaltyCustomerId": "uuid-or-null",
  "receiptNo": 1,
  "rtReceiptNo": null,
  "operatorCode": "OP01",
  "operatorName": "Mario",
  "paymentMethod": "CONTANTI"
}
```

> `customName` e' opzionale: se valorizzato, viene mostrato al posto di `productName` nelle analitiche e nelle ricevute. Utile per note personalizzate (es. "Pizza senza glutine").

| paymentMethod | |
|---|---|
| `CONTANTI` | Contanti |
| `ELETTRONICO` | Carta / POS |
| `NON_RISCOSSO` | Non riscosso |
| `BONIFICO` | Bonifico |
| `ASSEGNO` | Assegno |

| documentType | Scarico magazzino | Punti loyalty |
|---|---|---|
| `FISCAL` | Si | Si (se loyalty config attiva) |
| `NON_FISCAL` | No | No |

**Response:** `201 Created` con oggetto Receipt.

---

### GET - Scontrini per attivita'

### GET `/{receiptId}` - Singolo scontrino

---

## 9. Statistiche (`/license/{userId}/activities/{activityId}/statistics`)

Le statistiche vengono aggiornate automaticamente ad ogni scontrino salvato. Solo lettura.

### GET - Statistiche per attivita'

Filtri opzionali:
- `?statisticType=DAILY_REVENUE`
- `?startDate=2026-01-01&endDate=2026-01-31`
- Senza parametri: tutte

**Formato campo `data` per `DAILY_REVENUE`:**
```json
{
  "total": 1250.00,
  "transactions": 38,
  "avgTicket": 32.89,
  "fiscalTotal": 1100.00,
  "fiscalTransactions": 35,
  "cashTotal": 600.00,
  "cardTotal": 500.00
}
```

---

## 10. Analytics (`/license/{userId}/activities/{activityId}/analytics`)

Calcolate in tempo reale dagli scontrini. Tutti gli endpoint accettano `?startDate=2026-01-01&endDate=2026-01-31`.

### GET `/overview`

```json
{
  "totalRevenue": 5200.00,
  "totalTransactions": 143,
  "averageTicket": 36.36,
  "totalItemsSold": 410
}
```

### GET `/by-category`

```json
[
  { "categoryId": "uuid", "categoryLabel": "Pizze", "revenue": 2100.00, "quantity": 180 }
]
```

### GET `/by-product`

```json
[
  { "productId": "uuid", "productName": "Margherita", "revenue": 960.00, "quantity": 120 }
]
```

### GET `/by-hour`

```json
[
  { "hour": 12, "revenue": 820.00, "transactions": 22 },
  { "hour": 13, "revenue": 1100.00, "transactions": 30 }
]
```

### GET `/daily`

```json
[
  { "date": "2026-01-15", "revenue": 380.00, "transactions": 11 }
]
```

---

## 11. Food Cost (`/license/{userId}/activities/{activityId}/food-cost`)

### GET `/products`

Lista di tutti i prodotti con costo teorico calcolato da ingredienti + costi extra.

### GET `/products/{productId}`

Dettaglio food cost per singolo prodotto:
```json
{
  "productId": "uuid",
  "productName": "Margherita",
  "basePrice": 8.00,
  "ingredientCost": 1.20,
  "extraCost": 0.30,
  "totalCost": 1.50,
  "margin": 6.50,
  "marginPercent": 81.25
}
```

### GET `/report?startDate=2026-01-01&endDate=2026-01-31`

Report food cost aggregato per il periodo: costo teorico totale basato sulle quantita' vendute.

---

## 12. Tavoli e Sessioni (`/license/{userId}/activities/{activityId}/table-sessions`)

Sistema di codici sessione per gli ordini da tavolo. Il cliente scansiona il menu e inserisce il codice per associare l'ordine al tavolo corretto.

### POST - Apri sessione

```json
{ "tableNumber": "5" }
```

> `tableNumber` e' una stringa.

**Response `201 Created`:**
```json
{
  "id": "uuid",
  "activityId": "uuid",
  "tableNumber": "5",
  "code": "HK3T7R",
  "status": "ACTIVE",
  "createdAt": "2026-03-02T20:00:00",
  "closedAt": null
}
```

> Il codice e' di 6 caratteri alfanumerici (senza O, 0, I, 1 per evitare ambiguita').

---

### GET - Sessioni attive

Lista di tutte le sessioni con `status: ACTIVE`.

---

### DELETE `/{sessionId}` - Chiudi sessione

**Response 200:** oggetto sessione con `status: CLOSED` e `closedAt` valorizzato.

---

### GET `/waiter-calls` - Tavoli che hanno chiamato il cameriere

**Response 200:** `Set<String>` — insieme dei numeri tavolo con chiamata attiva.

```json
["3", "7", "12"]
```

---

### DELETE `/waiter-calls/{tableNumber}` - Rimuovi chiamata cameriere → `204 No Content`

---

### POST `/public/menu/{publicCode}/validate-session` *(pubblico, nessun auth)*

Validazione lato cliente (menuconnesso) prima di inviare un ordine.

```json
{ "code": "HK3T7R" }
```

**Response 200:**
```json
{ "sessionId": "uuid", "tableNumber": "5" }
```

Errore `404` se il codice non esiste o la sessione e' chiusa.

---

## 13. GDPR (`/license/{userId}/activities/{activityId}/gdpr`)

### GET `/customers/{customerId}` - Esporta dati (Art. 15)

Restituisce tutti i dati personali del cliente in formato JSON (diritto di accesso e portabilita').

**Response 200:**
```json
{
  "exportedAt": "2026-03-02T14:30:00",
  "customerId": "uuid",
  "firstName": "Anna",
  "lastName": "Verdi",
  "email": "anna@example.com",
  "phone": "+39123456789",
  "birthDate": "1990-05-15",
  "gender": "F",
  "points": 120,
  "credit": 15.00,
  "discountPercent": 0,
  "memberSince": "2025-01-10T10:00:00",
  "lastTransaction": "2026-02-20T18:45:00",
  "anonymized": false,
  "anonymizedAt": null
}
```

---

### DELETE `/customers/{customerId}` - Cancella dati (Art. 17)

Anonimizza il cliente: nome → "Anonimo", cognome → "GDPR", email/telefono/dati anagrafici → null. I dati contabili (scontrini) vengono conservati.

**Response 200:**
```json
{
  "customerId": "uuid",
  "erasedAt": "2026-03-02T14:30:00",
  "message": "Dati personali cancellati con successo"
}
```

Errore `409` se i dati sono gia' stati cancellati.

---

## 14. Admin (`/admin`) *(solo ruolo ADMIN)*

### Clienti

#### GET `/admin/clients` - Lista tutti i clienti

#### GET `/admin/clients/{clientId}` - Singolo cliente

#### POST `/admin/clients` - Crea cliente

```json
{
  "name": "Mario",
  "surname": "Rossi",
  "email": "mario@example.com",
  "password": "Password123!",
  "licenseExpireAt": "2027-12-31"
}
```

#### PUT `/admin/clients/{clientId}` - Aggiorna cliente

```json
{
  "name": "Mario",
  "surname": "Rossi",
  "email": "mario@example.com",
  "licenseExpireAt": "2027-12-31",
  "enabled": true,
  "modules": ["tavoli", "food_cost", "fidelity"]
}
```

#### POST `/admin/clients/{clientId}/disable` → ClientUser

#### POST `/admin/clients/{clientId}/enable` → ClientUser

#### POST `/admin/clients/{clientId}/disable-2fa` → ClientUser

#### DELETE `/admin/clients/{clientId}` → `204 No Content`

#### PUT `/admin/clients/{clientId}/modules`

```json
{ "modules": ["tavoli", "fidelity"] }
```

---

### Moduli disponibili

#### GET `/admin/clients/modules`

Lista statica di tutti i moduli esistenti nel sistema.

**Response 200:**
```json
[
  { "id": "tavoli",    "label": "Tavoli",    "description": "Gestione tavoli, sessioni codice e ordini da menu" },
  { "id": "food_cost", "label": "Food Cost", "description": "Calcolo food cost teorico da ingredienti e costi extra" },
  { "id": "fidelity",  "label": "Fidelity",  "description": "Programma fedelta': punti, credito, campagne sconto" }
]
```

---

### Audit Log

#### GET `/admin/audit-logs`

Cronologia di tutte le azioni significative nel sistema. Ordinati per data decrescente.

**Query params (tutti opzionali):**
| Param | Tipo | Descrizione |
|---|---|---|
| `from` | Date (`2026-01-01`) | Data inizio (default: 30 giorni fa) |
| `to` | Date (`2026-03-02`) | Data fine (default: oggi) |
| `userId` | UUID | Filtra per utente specifico |
| `action` | String | Filtra per azione (es. `LOGIN`, `CLIENT_CREATE`) |
| `status` | String | `SUCCESS` oppure `FAILURE` |

**Response 200:**
```json
[
  {
    "id": "uuid",
    "userId": "uuid",
    "userRole": "CLIENT",
    "action": "LOGIN",
    "resourceType": "AUTH",
    "resourceId": "",
    "ipAddress": "93.12.34.56",
    "details": "",
    "status": "SUCCESS",
    "createdAt": "2026-03-02T20:15:30"
  }
]
```

**Azioni loggiate:**

| Azione | Chi | Quando |
|---|---|---|
| `LOGIN` | CLIENT / ADMIN | Login riuscito o fallito |
| `2FA_VERIFY` | CLIENT / ADMIN | Verifica codice TOTP |
| `2FA_ENABLE` | CLIENT / ADMIN | Attivazione 2FA |
| `CLIENT_CREATE` | ADMIN | Creazione cliente |
| `CLIENT_UPDATE` | ADMIN | Modifica cliente |
| `CLIENT_DELETE` | ADMIN | Eliminazione cliente |
| `CLIENT_DISABLE` | ADMIN | Disabilitazione account |
| `CLIENT_ENABLE` | ADMIN | Riabilitazione account |
| `CLIENT_DISABLE_2FA` | ADMIN | Reset 2FA cliente |
| `CLIENT_UPDATE_MODULES` | ADMIN | Modifica moduli cliente |

---

## 15. Menu Digitale (`/license/{userId}/activities/{activityId}/menu`)

### Categorie menu

#### GET `/categories` → `List<MenuCategory>`

#### POST `/categories` - Crea categoria menu

```json
{
  "name": "Antipasti",
  "description": "...",
  "color": "#FF0000",
  "icon": "utensils",
  "imageUrl": "https://...",
  "sortOrder": 1
}
```

Tutti i campi opzionali tranne `name`.

**Response 201:** oggetto `MenuCategory` con `id, name, description, color, icon, imageUrl, sortOrder, active`

---

#### PUT `/categories/{categoryId}`

```json
{
  "name": "Antipasti",
  "description": "...",
  "color": "#FF0000",
  "icon": "utensils",
  "imageUrl": "https://...",
  "sortOrder": 1,
  "active": true
}
```

---

#### DELETE `/categories/{categoryId}` → `204 No Content`

---

### Voci menu

#### GET `/items[?categoryId={uuid}]` → `List<MenuItem>`

#### POST `/items` - Crea voce menu

```json
{
  "menuCategoryId": "uuid",
  "name": "Bruschetta",
  "price": 5.50,
  "description": "...",
  "imageUrl": "https://...",
  "allergens": ["glutine", "pomodoro"],
  "vegetarian": true,
  "vegan": false,
  "glutenFree": false,
  "spicy": false,
  "displayOrder": 1
}
```

**Response 201:** oggetto `MenuItem` con `id, menuCategoryId, name, description, price, imageUrl, allergens, vegetarian, vegan, glutenFree, spicy, available, displayOrder`

---

#### PUT `/items/{itemId}`

Stessi campi di POST piu' `"available": true/false`.

---

#### DELETE `/items/{itemId}` → `204 No Content`

---

### Import da POS

#### POST `/import-from-pos`

Crea una categoria menu e importa prodotti POS selezionati come voci menu. Se `productIds` e' vuoto, importa tutti i prodotti attivi dell'attivita'.

```json
{ "menuCategoryName": "Pizze", "productIds": ["uuid1", "uuid2"] }
```

**Response 201:** `List<MenuItem>` importati

---

## 16. Ordini da Menu (`/license/{userId}/activities/{activityId}/menu-orders`)

### GET `/[?status=PENDING]` → `List<MenuOrder>`

Valori status: `PENDING`, `PREPARING`, `READY`, `DELIVERED`, `CANCELLED`

---

### PUT `/{orderId}/status`

```json
{ "status": "PREPARING" }
```

**Response 200:** oggetto `MenuOrder` aggiornato

---

## Menu Pubblico (`/public/menu/{publicCode}`) *(nessun auth)*

Endpoint pubblici usati da menuconnesso.

### GET `/public/menu/{publicCode}`

Restituisce il menu pubblico dell'attivita' (prodotti e categorie visibili).

### POST `/public/menu/{publicCode}/orders`

Invia un ordine dal menu digitale.

```json
{
  "tableNumber": "5",
  "sessionCode": "HK3T7R",
  "items": [
    { "productId": "uuid", "quantity": 2, "notes": "senza cipolla" }
  ]
}
```

> `sessionCode` e' opzionale. Se presente, viene validato e il `tableNumber` viene ricavato dalla sessione (ignorando quello nel body).

### POST `/public/menu/{publicCode}/validate-session`

Vedi sezione 12.

---

### POST `/public/menu/{publicCode}/call-waiter?table=5` *(pubblico)*

Segnala la richiesta di un cameriere per il tavolo indicato. La chiamata viene accodata lato backend.

**Response:** `200 OK`

---

### DELETE `/public/menu/{publicCode}/call-waiter?table=5` *(pubblico)*

Rimuove la chiamata cameriere per il tavolo indicato (es. dopo che il cameriere ha risposto).

**Response:** `200 OK`

---

## Modelli Dati

### ClientUser

| Campo | Tipo | Descrizione |
|---|---|---|
| uuid | UUID | ID |
| name | String | Nome |
| surname | String | Cognome |
| email | String | Email |
| licenseExpireAt | LocalDate | Scadenza licenza |
| enabled | boolean | Account attivo |
| twoFactorEnabled | boolean | 2FA attivo |
| modules | List\<String\> | Moduli abilitati |

---

### Activity

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| publicCode | String | Codice pubblico usato nell'URL del menu digitale |
| name | String | Nome attivita' |
| vatID | String | Partita IVA |
| type | ActivityType | Tipo societa' |
| businessName | String | Ragione sociale |
| address | String | Indirizzo |
| city | String | Citta' |
| postalCode | String | CAP |
| province | String | Provincia |
| phone | String | Telefono |
| email | String | Email |
| sdiCode | String | Codice SDI |
| pec | String | PEC |
| loyaltyConfig | LoyaltyConfig | Configurazione punti |

**ActivityType:** `SOLE_PROPRIETORSHIP`, `SIMPLE_PARTNERSHIP`, `GENERAL_PARTNERSHIP`, `LIMITED_PARTNERSHIP`, `LIMITED_LIABILITY_COMPANY`, `JOINT_STOCK_COMPANY`, `COOPERATIVE`

---

### Category

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID (`@JsonProperty("id")`) |
| activityId | UUID | Attivita' |
| parentCategoryId | UUID | Categoria padre (null = radice) |
| name | String | Nome |
| description | String | Descrizione |
| defaultVat | Integer | IVA predefinita |
| color | String | Colore hex |
| icon | String | Icona |
| imageUrl | String | URL immagine |
| sortOrder | int | Ordinamento |
| vatNature | String | Natura IVA (ES, NI, AL, VI) |
| visibleRetail | boolean | Visibile modalita' retail |
| visibleRestaurant | boolean | Visibile modalita' ristorante |
| visibleTakeaway | boolean | Visibile modalita' asporto |
| hidden | boolean | Nascosta |
| active | boolean | Attiva |
| fiscal | boolean | Fiscale |

---

### Product

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID (`@JsonProperty("id")`) |
| categoryId | UUID | Categoria |
| name | String | Nome |
| description | String | Descrizione |
| barcode | String | Codice a barre |
| imageUrl | String | URL immagine |
| color | String | Colore |
| basePrice | BigDecimal | Prezzo base (listino 1) |
| priceList2 | BigDecimal | Prezzo listino 2 |
| priceList3 | BigDecimal | Prezzo listino 3 |
| priceList4 | BigDecimal | Prezzo listino 4 |
| iva | int | IVA (%) |
| sortOrder | int | Ordinamento |
| vatNature | String | Natura IVA |
| active | boolean | Attivo |
| fiscal | boolean | Fiscale |

---

### Variant

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| name | String | Nome (es. "Extra mozzarella") |
| pricePlus | BigDecimal | Sovrapprezzo |
| priceMinus | BigDecimal | Sconto |
| categoryId | UUID | Categoria associata (nullable) |
| productId | UUID | Prodotto associato (nullable) |
| active | boolean | Attiva |
| sortOrder | int | Ordinamento |

---

### LoyaltyConfig

| Campo | Tipo | Descrizione |
|---|---|---|
| spendThreshold | BigDecimal | Spesa minima per guadagnare punti |
| spendAmount | BigDecimal | Ogni X euro = 1 scaglione |
| pointsAwarded | int | Punti per scaglione |
| enabled | boolean | Programma attivo |

---

### LoyaltyCustomer

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityIds | List\<UUID\> | Attivita' associate |
| name, surname | String | Anagrafica |
| email, phone | String | Contatti (nullable) |
| birthDate | LocalDate | Data di nascita (nullable) |
| gender | String | Sesso (nullable) |
| categoryIds | List\<UUID\> | Categorie loyalty assegnate |
| points | int | Punti accumulati |
| credit | BigDecimal | Credito (€) |
| discount | int | Sconto personale (%) |
| active | boolean | Attivo |
| gdprErasedAt | LocalDateTime | Data anonimizzazione GDPR (null = non anonimizzato) |
| createdAt | LocalDateTime | Data creazione |
| lastTransactionAt | LocalDateTime | Ultima transazione |

---

### DiscountCampaign

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| name | String | Nome campagna |
| discountPercent | int | Sconto (%) |
| global | boolean | Applicata a tutto il catalogo |
| categoryIds | List\<UUID\> | Categorie specifiche (se non global) |
| startDate | LocalDateTime | Inizio |
| endDate | LocalDateTime | Fine (null = senza scadenza) |
| active | boolean | Attiva |

---

### PointsReward

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| pointsCost | int | Punti necessari |
| rewardType | RewardType | CREDIT / PURCHASE_DISCOUNT / VOUCHER |
| value | BigDecimal | Valore (€ o %) |
| description | String | Descrizione |
| active | boolean | Attivo |

---

### Receipt

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| documentType | DocumentType | FISCAL / NON_FISCAL |
| items | List\<ReceiptItem\> | Righe |
| loyaltyCustomerId | UUID | Cliente loyalty (nullable) |
| receiptNo | Integer | Progressivo scontrino app |
| rtReceiptNo | String | Numero scontrino RT (nullable) |
| operatorCode | String | Codice cassiere (nullable) |
| operatorName | String | Nome cassiere (nullable) |
| paymentMethod | String | Metodo di pagamento |
| invoiced | boolean | Fatturato |
| invoiceId | UUID | ID fattura (nullable) |
| total | BigDecimal | Totale calcolato |
| createdAt | LocalDateTime | Data |

---

### ReceiptItem

| Campo | Tipo | Descrizione |
|---|---|---|
| productId | UUID | Prodotto |
| productName | String | Nome prodotto |
| categoryId | UUID | Categoria (nullable) |
| categoryLabel | String | Nome categoria (nullable) |
| customName | String | Nome personalizzato temporaneo (nullable) |
| quantity | int | Quantita' |
| unitPrice | BigDecimal | Prezzo unitario |
| vat | int | IVA (%) |
| discountPercent | Integer | Sconto % — calcolato dal backend (campagna o loyalty) |
| discountEuro | BigDecimal | Sconto manuale cassiere (€, nullable) |

> `displayName()`: se `customName` e' valorizzato viene usato al posto di `productName` nelle visualizzazioni.
> Totale riga: `unitPrice * quantity`, poi `-discountPercent%`, poi `-discountEuro`.

---

### Statistic

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| date | LocalDate | Data |
| statisticType | String | Tipo (es. `DAILY_REVENUE`) |
| data | String | JSON con i dati |
| createdAt | LocalDateTime | Creazione |
| updatedAt | LocalDateTime | Ultimo aggiornamento |

---

### AuditLog

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| userId | UUID | Utente che ha eseguito l'azione |
| userRole | String | `ADMIN` o `CLIENT` |
| action | String | Azione eseguita (es. `LOGIN`) |
| resourceType | String | Tipo risorsa coinvolta (es. `AUTH`, `CLIENT`) |
| resourceId | String | ID risorsa (nullable) |
| ipAddress | String | IP del client (nullable) |
| details | String | Dettagli aggiuntivi (nullable) |
| status | AuditStatus | `SUCCESS` o `FAILURE` |
| createdAt | LocalDateTime | Timestamp |

---

### MenuCategory

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| name | String | Nome |
| description | String | Descrizione (nullable) |
| color | String | Colore hex (nullable) |
| icon | String | Icona (nullable) |
| imageUrl | String | URL immagine (nullable) |
| sortOrder | int | Ordinamento |
| active | boolean | Attiva |

---

### MenuItem

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| menuCategoryId | UUID | Categoria menu |
| name | String | Nome |
| description | String | Descrizione (nullable) |
| price | BigDecimal | Prezzo |
| imageUrl | String | URL immagine (nullable) |
| allergens | List\<String\> | Allergeni |
| vegetarian | boolean | Vegetariano |
| vegan | boolean | Vegano |
| glutenFree | boolean | Senza glutine |
| spicy | boolean | Piccante |
| available | boolean | Disponibile |
| displayOrder | int | Ordinamento |

---

### MenuOrder

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| tableNumber | String | Numero tavolo |
| generalNotes | String | Note generali (nullable) |
| status | MenuOrderStatus | Stato ordine |
| items | List\<MenuOrderItem\> | Righe ordine |
| createdAt | LocalDateTime | Creazione |
| updatedAt | LocalDateTime | Ultimo aggiornamento (nullable) |

**MenuOrderStatus:** `PENDING`, `PREPARING`, `READY`, `DELIVERED`, `CANCELLED`

**Esempio MenuOrder:**
```json
{
  "id": "uuid",
  "activityId": "uuid",
  "tableNumber": "5",
  "generalNotes": "senza cipolla",
  "status": "PENDING",
  "items": [
    {
      "menuItemId": "uuid",
      "name": "Bruschetta",
      "quantity": 2,
      "unitPrice": 5.50,
      "notes": "extra olio"
    }
  ],
  "createdAt": "2026-03-02T20:15:00",
  "updatedAt": null
}
```

---

### MenuOrderItem

| Campo | Tipo | Descrizione |
|---|---|---|
| menuItemId | UUID | Voce menu |
| name | String | Nome al momento dell'ordine |
| quantity | int | Quantita' |
| unitPrice | BigDecimal | Prezzo unitario |
| notes | String | Note (nullable) |

---

### TableSession

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
| activityId | UUID | Attivita' |
| tableNumber | String | Numero tavolo (stringa) |
| code | String | Codice 6 caratteri (es. `HK3T7R`) |
| status | SessionStatus | `ACTIVE` o `CLOSED` |
| createdAt | LocalDateTime | Apertura sessione |
| closedAt | LocalDateTime | Chiusura sessione (nullable) |
