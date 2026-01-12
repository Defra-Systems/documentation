# CassaStore Backend API Documentation

Documentazione completa delle API REST del sistema CassaStore Backend.

**Base URL**: `http://localhost:8080/api/v1`

---

## Indice

1. [Autenticazione](#autenticazione)
2. [Licenze](#licenze)
3. [Categorie](#categorie)
4. [Prodotti](#prodotti)
5. [Statistiche](#statistiche)
6. [Loyalty (Programma Fedeltà)](#loyalty-programma-fedeltà)
7. [Rate Limiting](#rate-limiting)
8. [Sicurezza](#sicurezza)

---

## Autenticazione

### Login Client
**POST** `/auth/client/login`

Login per clienti.

**Request Body:**
```json
{
  "email": "client@example.com",
  "password": "Password123!@#"
}
```




**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Status Codes:**
- `200 OK` - Login riuscito
- `401 Unauthorized` - Credenziali non valide
- `403 Forbidden` - Account disabilitato
- `429 Too Many Requests` - Troppi tentativi (max 5/min)
---

## Licenze

### Verifica Licenza
**GET** `/license/{userId}/verify`

Verifica lo stato di una licenza.

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `userId` (UUID) - ID della licenza

**Response:**
```json
{
  "valid": true,
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "licenseExpireAt": "2026-12-31",
  "modules": ["vendite", "magazzino", "loyalty"],
  "activities": ["Negozio1", "Negozio2"]
}
```

---

## Categorie

### Crea Categoria
**POST** `/license/{userId}/categories`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `userId` (UUID) - ID della licenza

**Request Body:**
```json
{
  "name": "Elettronica",
  "activityName": "Negozio1",
  "description": "Prodotti elettronici"
}
```

**Response:**
```json
{
  "id": 1,
  "name": "Elettronica",
  "activityName": "Negozio1",
  "description": "Prodotti elettronici",
  "createdAt": "2026-01-12T10:00:00",
  "updatedAt": "2026-01-12T10:00:00"
}
```

---

### Ottieni Tutte le Categorie
**GET** `/license/{userId}/categories?activityName=Negozio1`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required) - Nome dell'activity

---

### Ottieni Categoria per ID
**GET** `/license/{userId}/categories/{categoryId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `categoryId` (Long) - ID della categoria

---

### Aggiorna Categoria
**PUT** `/license/{userId}/categories/{categoryId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `categoryId` (Long) - ID della categoria

**Request Body:**
```json
{
  "name": "Elettronica Avanzata",
  "description": "Prodotti elettronici di ultima generazione"
}
```

---

### Elimina Categoria
**DELETE** `/license/{userId}/categories/{categoryId}`

**Headers:**
```
Authorization: Bearer {token}
```

Soft delete di una categoria.

---

## Prodotti

### Crea Prodotto
**POST** `/license/{userId}/products`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `userId` (UUID) - ID della licenza

**Request Body:**
```json
{
  "name": "iPhone 15",
  "activityName": "Negozio1",
  "categoryId": 1,
  "sku": "IPH15-001",
  "barcode": "1234567890123",
  "description": "iPhone 15 128GB",
  "price": 899.99,
  "cost": 650.00,
  "stock": 10,
  "minStock": 2
}
```

**Response:**
```json
{
  "id": 1,
  "name": "iPhone 15",
  "activityName": "Negozio1",
  "categoryId": 1,
  "categoryName": "Elettronica",
  "sku": "IPH15-001",
  "barcode": "1234567890123",
  "price": 899.99,
  "cost": 650.00,
  "stock": 10,
  "minStock": 2
}
```

---

### Ottieni Tutti i Prodotti
**GET** `/license/{userId}/products?activityName=Negozio1`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required) - Nome dell'activity

---

### Ottieni Prodotti per Categoria
**GET** `/license/{userId}/products?activityName=Negozio1&categoryId=1`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `categoryId` (required)

---

### Cerca Prodotti
**GET** `/license/{userId}/products/search?activityName=Negozio1&keyword=iPhone`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `keyword` (required)

**Note:** La ricerca viene effettuata su nome, SKU e barcode.

---

### Cerca Prodotto per Barcode
**GET** `/license/{userId}/products/barcode/{barcode}?activityName=Negozio1`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `barcode` - Codice a barre

**Query Parameters:**
- `activityName` (required)

---

### Ottieni Prodotto per ID
**GET** `/license/{userId}/products/{productId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `productId` (Long)

---

### Aggiorna Prodotto
**PUT** `/license/{userId}/products/{productId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `productId` (Long)

**Request Body:**
```json
{
  "name": "iPhone 15 Pro",
  "price": 999.99,
  "stock": 15
}
```

---

### Aggiorna Stock
**PATCH** `/license/{userId}/products/{productId}/stock`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `productId` (Long)

**Request Body:**
```json
{
  "quantity": 5
}
```

**Note:** Usare numeri positivi per aggiungere, negativi per sottrarre.

---

### Elimina Prodotto
**DELETE** `/license/{userId}/products/{productId}`

**Headers:**
```
Authorization: Bearer {token}
```

Soft delete di un prodotto.

---

## Statistiche

### Registra Vendita
**POST** `/license/{userId}/statistics`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `userId` (UUID)

**Request Body:**
```json
{
  "activityName": "Negozio1",
  "amount": 899.99,
  "paymentMethod": "CARD",
  "items": 1,
  "notes": "Vendita iPhone 15"
}
```

**Payment Methods:**
- `CASH` - Contanti
- `CARD` - Carta
- `BANK_TRANSFER` - Bonifico
- `OTHER` - Altro

---

### Ottieni Statistiche per Activity
**GET** `/license/{userId}/statistics?activityName=Negozio1`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)

---

### Ottieni Statistiche per Periodo
**GET** `/license/{userId}/statistics/period?activityName=Negozio1&startDate=2026-01-01&endDate=2026-01-31`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `startDate` (required, formato: YYYY-MM-DD)
- `endDate` (required, formato: YYYY-MM-DD)

---

### Ottieni Totale per Periodo
**GET** `/license/{userId}/statistics/total?activityName=Negozio1&startDate=2026-01-01&endDate=2026-01-31`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `startDate` (required)
- `endDate` (required)

**Response:**
```json
{
  "total": 15789.50,
  "count": 42,
  "averageTransaction": 376.18
}
```

---

### Ottieni Statistica per ID
**GET** `/license/{userId}/statistics/{statisticId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `statisticId` (Long)

---

### Aggiorna Statistica
**PUT** `/license/{userId}/statistics/{statisticId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `statisticId` (Long)

---

### Elimina Statistica
**DELETE** `/license/{userId}/statistics/{statisticId}`

**Headers:**
```
Authorization: Bearer {token}
```

Soft delete di una statistica.

---

## Loyalty (Programma Fedeltà)

### Crea Cliente Fedeltà
**POST** `/license/{userId}/loyalty`

**Headers:**
```
Authorization: Bearer {token}
```

Crea un nuovo cliente nel programma fedeltà.

**Path Parameters:**
- `userId` (UUID) - ID della licenza

**Request Body:**
```json
{
  "activityName": "Negozio1",
  "name": "Giovanni",
  "surname": "Verdi",
  "email": "giovanni.verdi@example.com",
  "phone": "3331234567",
  "initialCredito": 50.00,
  "sconto": 10,
  "active": true
}
```

**Note:**
- Un cliente può avere UNA SOLA carta fedeltà per activity (vincolo univoco)
- `initialCredito` è opzionale (default: 0)
- `sconto` è opzionale, valore da 0 a 100 (default: 0)

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "activityName": "Negozio1",
  "name": "Giovanni",
  "surname": "Verdi",
  "email": "giovanni.verdi@example.com",
  "phone": "3331234567",
  "points": 0,
  "credito": 50.00,
  "sconto": 10,
  "enrolledAt": "2026-01-12T10:00:00",
  "active": true
}
```

**Status Codes:**
- `201 Created` - Cliente creato
- `409 Conflict` - Cliente già esistente per questa activity

---

### Ottieni Tutti i Clienti Fedeltà
**GET** `/license/{userId}/loyalty?activityName=Negozio1`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)

**Response:**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "name": "Giovanni",
    "surname": "Verdi",
    "points": 150,
    "credito": 75.50,
    "sconto": 10,
    "active": true
  }
]
```

---

### Ottieni Solo Clienti Attivi
**GET** `/license/{userId}/loyalty?activityName=Negozio1&activeOnly=true`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `activeOnly` (required) - true per solo attivi

---

### Cerca Cliente per Email
**GET** `/license/{userId}/loyalty/search?activityName=Negozio1&email=giovanni.verdi@example.com`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `email` (required)

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "name": "Giovanni",
  "surname": "Verdi",
  "email": "giovanni.verdi@example.com",
  "points": 150,
  "credito": 75.50,
  "sconto": 10
}
```

---

### Cerca Cliente per Telefono
**GET** `/license/{userId}/loyalty/search?activityName=Negozio1&phone=3331234567`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `phone` (required)

---

### Cerca Clienti per Nome
**GET** `/license/{userId}/loyalty/search?activityName=Negozio1&name=Giovanni`

**Headers:**
```
Authorization: Bearer {token}
```

**Query Parameters:**
- `activityName` (required)
- `name` (required)

**Note:** Cerca sia nel nome che nel cognome (case-insensitive).

---

### Ottieni Cliente per ID
**GET** `/license/{userId}/loyalty/{customerId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `customerId` (String)

---

### Aggiorna Cliente Fedeltà
**PUT** `/license/{userId}/loyalty/{customerId}`

**Headers:**
```
Authorization: Bearer {token}
```

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "name": "Giovanni",
  "surname": "Verdi",
  "email": "newemail@example.com",
  "phone": "3337654321",
  "active": true
}
```

**Note:** I campi `points`, `credito` e `sconto` NON possono essere modificati tramite questo endpoint.

---

### Aggiungi Punti
**POST** `/license/{userId}/loyalty/{customerId}/points/add`

**Headers:**
```
Authorization: Bearer {token}
```

Aggiunge punti al cliente.

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "points": 50
}
```

**Validazione:**
- `points` >= 1

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "name": "Giovanni",
  "surname": "Verdi",
  "points": 200,
  "credito": 75.50,
  "sconto": 10,
  "lastTransactionAt": "2026-01-12T11:30:00"
}
```

---

### Sottrai Punti
**POST** `/license/{userId}/loyalty/{customerId}/points/subtract`

**Headers:**
```
Authorization: Bearer {token}
```

Sottrae punti dal cliente.

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "points": 30
}
```

**Validazione:**
- `points` >= 1
- Il cliente deve avere abbastanza punti

**Status Codes:**
- `200 OK` - Punti sottratti
- `400 Bad Request` - Punti insufficienti

---

### Ricarica Credito
**POST** `/license/{userId}/loyalty/{customerId}/credito/add`

**Headers:**
```
Authorization: Bearer {token}
```

Ricarica credito sulla carta fedeltà.

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "importo": 25.00
}
```

**Validazione:**
- `importo` > 0.00

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "name": "Giovanni",
  "surname": "Verdi",
  "points": 170,
  "credito": 100.50,
  "sconto": 10,
  "lastTransactionAt": "2026-01-12T11:40:00"
}
```

---

### Usa Credito
**POST** `/license/{userId}/loyalty/{customerId}/credito/use`

**Headers:**
```
Authorization: Bearer {token}
```

Utilizza credito dalla carta fedeltà.

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "importo": 15.00
}
```

**Validazione:**
- `importo` > 0.00
- Il cliente deve avere abbastanza credito

**Status Codes:**
- `200 OK` - Credito utilizzato
- `400 Bad Request` - Credito insufficiente

---

### Aggiorna Sconto
**PUT** `/license/{userId}/loyalty/{customerId}/sconto`

**Headers:**
```
Authorization: Bearer {token}
```

Aggiorna lo sconto personalizzato del cliente.

**Path Parameters:**
- `customerId` (String)

**Request Body:**
```json
{
  "sconto": 15
}
```

**Validazione:**
- `sconto` tra 0 e 100

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "name": "Giovanni",
  "surname": "Verdi",
  "points": 170,
  "credito": 85.50,
  "sconto": 15
}
```

---

### Elimina Cliente Fedeltà
**DELETE** `/license/{userId}/loyalty/{customerId}`

**Headers:**
```
Authorization: Bearer {token}
```

Soft delete di un cliente fedeltà.

**Path Parameters:**
- `customerId` (String)

**Status Codes:**
- `204 No Content`

---

## Rate Limiting

- **Login endpoint**: 5 richieste/minuto per IP
- **Tutti gli altri endpoint**: 100 richieste/minuto per IP

**Response quando si supera il limite:**
```
Status: 429 Too Many Requests
```

---

## Sicurezza

### Autenticazione JWT

Tutti gli endpoint (tranne login) richiedono un token JWT:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Password Policy

- Minimo 8 caratteri
- Almeno una lettera maiuscola
- Almeno una lettera minuscola
- Almeno un numero
- Almeno un carattere speciale (@$!%*?&)

### Security Headers

- Strict-Transport-Security (HSTS)
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY
- X-XSS-Protection
- Content-Security-Policy
- Referrer-Policy
- Permissions-Policy

---

## Gestione Errori

### Formato Standard

```json
{
  "status": 400,
  "message": "Descrizione errore",
  "timestamp": "2026-01-12T10:00:00"
}
```

### Codici di Stato

- **200 OK** - Richiesta riuscita
- **201 Created** - Risorsa creata
- **204 No Content** - Richiesta riuscita senza contenuto
- **400 Bad Request** - Dati non validi
- **401 Unauthorized** - Autenticazione richiesta/fallita
- **403 Forbidden** - Accesso negato
- **404 Not Found** - Risorsa non trovata
- **409 Conflict** - Conflitto (es. email duplicata)
- **429 Too Many Requests** - Rate limit superato
- **500 Internal Server Error** - Errore interno

### Errori di Validazione

```json
{
  "status": 400,
  "errors": {
    "email": "L'email deve essere valida",
    "password": "La password deve contenere almeno una maiuscola"
  },
  "timestamp": "2026-01-12T10:00:00"
}
```

---

## Note Importanti

1. **Soft Delete**: La maggior parte delle eliminazioni sono soft delete (marcati come eliminati, non rimossi).

2. **Multi-tenancy**: Tutti i dati sono isolati per userId + activityName.

3. **UUID come Licenza**: L'UUID del client funge da chiave di licenza univoca.

4. **Unique Constraint Loyalty**: Un cliente può avere UNA SOLA carta fedeltà per activity.

5. **Gestione Punti/Credito/Sconto**: Questi campi possono essere modificati SOLO tramite endpoint dedicati.

6. **Audit Logging**: Tutte le operazioni critiche vengono registrate in modo asincrono.

---

**Versione API**: 1.0
**Ultimo aggiornamento**: 12 Gennaio 2026
