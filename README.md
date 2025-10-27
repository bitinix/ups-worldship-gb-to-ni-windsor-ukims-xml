# UPS WorldShip XML: GB → Northern Ireland (Windsor Framework & UKIMS)

A practical guide (with copy‑paste XML) for generating UPS **WorldShip** shipments from **Great Britain to Northern Ireland** using the **Windsor Framework** rules and **UK Internal Market Scheme (UKIMS)**.

> **Who is this for?**
> Developers and ops teams importing XML into UPS WorldShip (XML Auto Import) who need their parcels/freight GB→NI treated correctly under the Windsor Framework ("green lane" when eligible).

---

## Table of contents

* [Quick start](#quick-start)
* [Minimal working XML](#minimal-working-xml)
* [Required fields (Windsor/UKIMS)](#required-fields-windsorukims)
* [ComplianceIDs reference](#complianceids-reference)
* [Country & address rules](#country--address-rules)
* [Schema (OpenShipments.xdr) pathing](#schema-openshipmentsxdr-pathing)
* [WorldShip folders & how to find ](#worldship-folders--how-to-find-openshipmentsxdr)[`OpenShipments.xdr`](#worldship-folders--how-to-find-openshipmentsxdr)
* [Troubleshooting & common errors](#troubleshooting--common-errors)
* [PowerShell snippets](#powershell-snippets)
* [FAQ](#faq)
* [Change log](#change-log)
* [Disclaimer](#disclaimer)
* [License](#license)

---

## Quick start

1. **Use ************************************************************************************`NB`************************************************************************************ for Northern Ireland** in `<ShipTo><CountryTerritory>NB</CountryTerritory>`.
2. Inside `<ShipmentInformation>`, add the **International Compliance** flags:

   * `<InternationalComplianceShipmentReceiver>` = `B` (business) or `C` (consumer)
   * `<InternationalComplianceRiskLevel>` = `NotAtRisk` | `AtRisk` | `RiskUnknown`
3. Add a `<ComplianceIDs>` block with the relevant **UKIMS/EORI** IDs.
4. Ensure your XML root references the **OpenShipments.xdr** schema where it actually exists.

---

## Minimal working XML

```xml
<?xml version="1.0" encoding="windows-1252"?>
<OpenShipments xmlns="x-schema:OpenShipments.xdr">
  <OpenShipment ProcessStatus="" ShipmentOption="">

    <ShipTo>
      <CompanyOrName>Bitinix Ltd</CompanyOrName>
      <Attention>Joe Bradbury</Attention>
      <Address1>1 Example Road</Address1>
      <CityOrTown>Belfast</CityOrTown>
      <PostalCode>BT1 1AA</PostalCode>
      <CountryTerritory>NB</CountryTerritory>
      <Telephone>02890000000</Telephone>
      <EmailAddress>receiving@example.com</EmailAddress>
    </ShipTo>

    <ShipmentInformation>
      <ServiceType>ST</ServiceType>
      <PackageType>CP</PackageType>
      <ProcessAsPaperless>Y</ProcessAsPaperless>

      <!-- Windsor Framework fields -->
      <InternationalComplianceShipmentReceiver>B</InternationalComplianceShipmentReceiver>
      <InternationalComplianceRiskLevel>NotAtRisk</InternationalComplianceRiskLevel>
    </ShipmentInformation>

    <!-- Goods lines as normal -->
    <Goods>
      <GoodsLine>
        <DescriptionOfGoods>Example widgets</DescriptionOfGoods>
        <UnitPrice>10.00</UnitPrice>
        <UnitOfMeasure>EA</UnitOfMeasure>
        <Quantity>2</Quantity>
        <CountryOfOrigin>GB</CountryOfOrigin>
        <CommodityCode>12345678</CommodityCode>
        <Weight>0.5</Weight>
      </GoodsLine>
    </Goods>

    <!-- Compliance IDs for Windsor/UKIMS -->
    <ComplianceIDs>
      <!-- Optional: shipper VAT / Tax ID -->
      <ComplianceID>
        <ComplianceIDOwner>S</ComplianceIDOwner>
        <ComplianceIDComplianceCountry>GB</ComplianceIDComplianceCountry>
        <ComplianceIDTypeCode>DD00</ComplianceIDTypeCode>
        <ComplianceIDNumber>GB123456789</ComplianceIDNumber>
        <ComplianceFieldType>01</ComplianceFieldType>
      </ComplianceID>

      <!-- UKIMS number for the authorised party (often the importer) -->
      <ComplianceID>
        <ComplianceIDOwner>I</ComplianceIDOwner>
        <ComplianceIDComplianceCountry>NB</ComplianceIDComplianceCountry>
        <ComplianceIDTypeCode>DD65</ComplianceIDTypeCode>
        <ComplianceIDNumber>ZIUKIM12345678901234567890</ComplianceIDNumber>
        <ComplianceFieldType>01</ComplianceFieldType>
      </ComplianceID>

      <!-- Matching EORI for that same party -->
      <ComplianceID>
        <ComplianceIDOwner>I</ComplianceIDOwner>
        <ComplianceIDComplianceCountry>NB</ComplianceIDComplianceCountry>
        <ComplianceIDTypeCode>DD01</ComplianceIDTypeCode>
        <ComplianceIDNumber>GB123456789000</ComplianceIDNumber>
        <ComplianceFieldType>02</ComplianceFieldType>
      </ComplianceID>

      <!-- Optional: Duty Deferment Account number -->
      <ComplianceID>
        <ComplianceIDOwner>I</ComplianceIDOwner>
        <ComplianceIDComplianceCountry>NB</ComplianceIDComplianceCountry>
        <ComplianceIDTypeCode>DD66</ComplianceIDTypeCode>
        <ComplianceIDNumber>1234567</ComplianceIDNumber>
        <ComplianceFieldType>03</ComplianceFieldType>
      </ComplianceID>
    </ComplianceIDs>

    <Package>
      <PackageType>CP</PackageType>
      <Weight>1.0</Weight>
    </Package>
  </OpenShipment>
</OpenShipments>
```

> **Order matters**: The nodes must be in the order dictated by `OpenShipments.xdr`. Keep the International Compliance nodes **inside** `<ShipmentInformation>`.

---

## Required fields (Windsor/UKIMS)

* **`<ShipTo><CountryTerritory>`**** = ****`NB`** (Northern Ireland). Do **not** use `GB`.
* **Receiver type**: `<InternationalComplianceShipmentReceiver>`: `B` (Business) or `C` (Consumer).
* **Risk level**: `<InternationalComplianceRiskLevel>`: `NotAtRisk` when eligible under UKIMS; otherwise `AtRisk` or `RiskUnknown`.
* **Evidence & records**: Keep documentation supporting “not at risk” determinations (typical requirement: retain for 5 years).

---

## ComplianceIDs reference

A few commonly used UPS WorldShip codes for GB→NI:

| Purpose                             | Owner | Country | TypeCode | FieldType | Notes                               |
| ----------------------------------- | ----- | ------- | -------- | --------- | ----------------------------------- |
| Tax ID / VAT (Shipper)              | S     | GB      | DD00     | 01        | Optional but helpful                |
| **UKIMS number** (authorised party) | I     | **NB**  | **DD65** | 01        | Required for green-lane eligibility |
| **EORI** (matching UKIMS holder)    | I     | **NB**  | **DD01** | 02        | Must match the declared UKIMS party |
| Duty Deferment Account (optional)   | I     | NB      | DD66     | 03        | If applicable                       |

**Owner codes**: `S` = Shipper, `I` = Importer, `R` = Receiver (rare here).
**Country**: use `NB` for Northern Ireland items associated with the NI importer/receiver.

---

## Country & address rules

* **NI country code is ****`NB`** in WorldShip. This drives Windsor logic.
* UK postcodes for NI start with **`BT`** (e.g., BT1 1AA). Make sure `<PostalCode>` is a valid BT format.
* City/Town and Telephone should be provided; Email is recommended.

---

## Troubleshooting & common errors

Ensure you have the correct version.  I used 28.0.705  ( Oct 2025 )  - Filename: WS28_0_705_0_ENU.exe

### “element content is invalid according to the DTD/schema”

* The International Compliance nodes are **outside** `<ShipmentInformation>` or in the wrong order.
* Your WorldShip build is old and the shipped `OpenShipments.xdr` **doesn’t include** the Windsor fields yet → update WorldShip.
* The `xmlns` path doesn’t point to a **real** `OpenShipments.xdr`.

### “International Compliance ID Owner has invalid characters”

* Using `GB` not `NB` for NI‑specific Compliance IDs.
* Mismatch between the **UKIMS owner** and the **EORI owner**.

### “The Merchandise Description field has been left empty or is invalid.”

* Ensure each `GoodsLine` has a **clean** `<DescriptionOfGoods>` (letters/digits/spaces/hyphens are safest). Avoid unusual symbols and trim to sensible length (e.g., ≤ 65 chars).

### Encoding / special characters

* The XML declaration commonly uses `encoding="windows-1252"`. If you generate XML in Node.js, the built‑in `fs.writeFileSync` doesn’t support that label; either omit the explicit encoding (write as UTF‑8) or use a library like `iconv-lite` to output Windows‑1252.

---

## FAQ

**Q: Do I always need UKIMS for GB→NI?**
A: No. UKIMS applies to B2B movements to qualify goods as **NotAtRisk**. B2C parcels have simplified handling (no full customs declaration), but you still provide key data so the carrier can apply the correct process.

**Q: Who should the UKIMS belong to—shipper or importer?**
A: Either party can be UKIMS‑authorised; the **EORI** you provide should match the **same party** you list as holding UKIMS in the `ComplianceIDs` block.

**Q: What code should I use for NI in WorldShip?**
A: `NB`.

**Q: Need help? I'll help where I can, you can email me.**
A: [bitinix@gmail.com](mailto:bitinix@gmail.com)

---

## Change log

* **2025‑10‑27**: Initial version.

---

## Disclaimer

This repository is a developer aid. Always follow your carrier account instructions and current HMRC/UPS guidance.

---

## License

MIT License – see `LICENSE` file.
