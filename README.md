mkdir -p openapi schemas mapping .github/workflows

# README tartalom beírása
cat > README.md << 'EOF'
# NDC API Contracts

Ez a repository tartalmazza az NDC API szerződéseket (OpenAPI és XSD fájlok), a mapping dokumentációkat és a changelogot.

## 📌 Általános információk
- **API neve:** NDC Order & Shopping API
- **Verzió:** v21.3
- **Use case-ek:**
  - AirShopping (járat/ajánlat keresés)
  - OfferPrice (ajánlat árazás)
  - OrderCreate (foglalás)
  - OrderView/Retrieve (foglalás lekérés)
  - TicketIssue (jegykiállítás)
- **Állapot:** Aktív
- **Deprecation policy:** v20.2 – EOL: 2025-12-31

## 🔑 Autentikáció és biztonság
- OAuth2 Client Credentials + mTLS
- Kötelező headerek:
  - `Authorization: Bearer <token>`
  - `Idempotency-Key: <uuid>` (minden OrderCreate/Change POST)
  - `X-Correlation-ID: <traceId>`
- PII/PCI: tokenization, PAN-free logging, PCI DSS

## 🌐 Endpoints (példa)
| Endpoint                     | Metódus | Leírás                 |
|-----------------------------|---------|------------------------|
| `/ndc/v21.3/airshopping`    | POST    | Járat/ajánlat keresés  |
| `/ndc/v21.3/offers/price`   | POST    | Ajánlat árazás         |
| `/ndc/v21.3/orders`         | POST    | Foglalás létrehozása   |
| `/ndc/v21.3/orders/{id}`    | GET     | Foglalás lekérése      |
| `/ndc/v21.3/tickets`        | POST    | Jegykiállítás          |

## 📑 API szerződések
- OpenAPI: `openapi/ndc_order_v21.3.yaml`
- XML Schemas (XSD): `schemas/OrderCreateRQ.xsd`, `schemas/OrderCreateRS.xsd`

## ⚙️ Nem-funkcionális követelmények
- **SLA:** 99.9% availability
- **SLO:** AirShopping válaszidő ≤ 2000 ms (95%)
- **Rate limit:** 20 RPS (shopping), 2 RPS (order)
- **Retry policy:** exponenciális backoff + Idempotency-Key

## 📊 Mapping dokumentáció
- `mapping/ndc_to_pss_mapping.md`

## 📜 Changelog
Lásd: [CHANGELOG.md](CHANGELOG.md)

## 🧪 CI (javaslat)
- OpenAPI és XSD validáció minden push/PR esetén (GitHub Actions).
EOF

# minimális OpenAPI és XSD “placeholderek” (később cseréled a valódira)
cat > openapi/ndc_order_v21.3.yaml << 'EOF'
openapi: 3.0.3
info:
  title: NDC Order API
  version: 21.3
paths:
  /ndc/v21.3/orders:
    post:
      summary: Create Order
      parameters:
        - in: header
          name: Idempotency-Key
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/xml:
            schema:
              # Itt az XSD-re hivatkoztok a doksiban; a tényleges validációt a gateway/app végzi
              type: string
      responses:
        '201':
          description: Created
        '400':
          description: Bad Request
        '429':
          description: Rate limit exceeded
EOF

cat > schemas/OrderCreateRQ.xsd << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="OrderCreateRQ">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="Offer" type="xs:string" minOccurs="1" maxOccurs="1"/>
        <xs:element name="Passenger" minOccurs="1" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="PTC" type="xs:string"/>
              <xs:element name="Name" type="xs:string"/>
            </xs:sequence>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
EOF

cat > schemas/OrderCreateRS.xsd << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="OrderCreateRS">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="OrderID" type="xs:string"/>
        <xs:element name="BookingReference" type="xs:string"/>
        <xs:element name="Status" type="xs:string"/>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
EOF

# üres fájlok a jövőbeli tartalomhoz
echo "" > CHANGELOG.md
echo "" > mapping/ndc_to_pss_mapping.md
