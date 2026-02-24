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

### POST `/auth/setup-2fa`

```json
{ "email": "user@example.com" }
```

**Response 200:**
```json
{
  "secret": "BASE32SECRET",
  "otpAuthUri": "otpauth://totp/CassaConnessa:user@example.com?secret=BASE32SECRET&issuer=CassaConnessa"
}
```

---

### POST `/auth/confirm-2fa`

```json
{ "userId": "uuid", "userType": "CLIENT", "secret": "BASE32SECRET", "code": "123456" }
```

---

## 2. Profilo e Attivita' (`/license`)

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
  "publicCode": "ABC123",
  "twoFactorEnabled": false,
  "activities": [
    {
      "id": "uuid",
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

### Clienti

#### POST `/customers` - Crea cliente

```json
{ "name": "Anna", "surname": "Verdi", "email": "anna@example.com", "phone": "+39123456789" }
```

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
  "activityIds": ["uuid1"]
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
      "productName": "Margherita",
      "categoryId": "cat-uuid",
      "categoryLabel": "Pizze",
      "quantity": 2,
      "unitPrice": 8.00,
      "vat": 10,
      "discountEuro": null
    }
  ],
  "loyaltyCustomerId": "uuid-or-null"
}
```

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
      "productName": "Margherita",
      "categoryId": "cat-uuid",
      "categoryLabel": "Pizze",
      "quantity": 2,
      "unitPrice": 8.00,
      "vat": 10,
      "discountEuro": null
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
| publicCode | String | Codice pubblico licenza |
| twoFactorEnabled | boolean | 2FA attivo |

---

### Activity

| Campo | Tipo | Descrizione |
|---|---|---|
| id | UUID | ID |
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
| points | int | Punti accumulati |
| credit | BigDecimal | Credito (€) |
| discount | int | Sconto personale (%) |
| active | boolean | Attivo |
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
| quantity | int | Quantita' |
| unitPrice | BigDecimal | Prezzo unitario |
| vat | int | IVA (%) |
| discountPercent | Integer | Sconto % — calcolato dal backend (campagna o loyalty) |
| discountEuro | BigDecimal | Sconto manuale cassiere (€, nullable) |

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
