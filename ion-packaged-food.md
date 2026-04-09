# ION Packaged Food Schema

## Product Fields

| Business Field              | Beckn Schema Path                                                         | Schema Type                | Notes                              |
| --------------------------- | ------------------------------------------------------------------------- | -------------------------- | ---------------------------------- |
| Name                        | `Resource.descriptor.name`                                                | Descriptor                 | "Kue Lapis"                        |
| Brand                       | `RetailResource.identity.brand`                                           | string                     | Brand of the seller/bakery         |
| Description                 | `Resource.descriptor.shortDesc`                                           | Descriptor                 | Short product description          |
| Long Description            | `Resource.descriptor.longDesc`                                            | Descriptor                 | Full marketing copy                |
| Image URL                   | `Resource.descriptor.mediaFile[].url`                                     | MediaFile                  | Product photos                     |
| Quantity                    | `Offer.offerAttributes.customization.groups[code=SIZE]`                   | FoodAndBeverageOffer       | Modeled as size variant selection  |
| Net Weight                  | `RetailResource.physical.weight`                                          | Quantity                   | Weight per size variant            |
| BPOM Registration           | `RetailResource.foodRegulatoryDeclaration.registrations[scheme=BPOM]`     | object                     | BPOM MD/ML number                  |
| Halal (Yes/No)              | `RetailResource.foodRegulatoryDeclaration.registrations[scheme=HALAL]`    | object                     | Presence indicates certified       |
| Halal Certification Number  | `RetailResource.foodRegulatoryDeclaration.registrations[scheme=HALAL].id` | string                     | e.g. "LPPOM-00160123450124"        |
| Halal Certificate Image URL | `RetailResource.credentials[type=HALAL_CERTIFICATE].uri`                  | credential                 | URI to certificate image/PDF       |
| Organic (Yes/No)            | `RetailResource.credentials[type=ORGANIC_CERTIFICATE]`                    | credential                 | Presence indicates certified       |
| Vegetarian (Yes/No)         | `RetailResource.food.classification`                                      | enum                       | `VEG` / `NON_VEG` / `EGG`          |
| Shelf Life                  | `FoodAndBeverageResource.preparation.shelfLife`                           | string (ISO 8601 duration) | e.g. "P5D" for 5 days              |
| Allergen Info               | `FoodAndBeverageResource.allergens`                                       | array of enum              | e.g. `["DAIRY", "EGGS", "GLUTEN"]` |
| Nutritions                |  | | 

## Pricing Fields

| Business Field          | Beckn Schema Path                                                  | Schema Type         | Notes                              |
| ----------------------- | ------------------------------------------------------------------ | ------------------- | ---------------------------------- |
| Base Price              | `RetailOffer.price.value`                                          | PriceSpecification  | Base price of selected size        |
| Store Location          | `RetailResource.availableAt`                                       | Location[]          | GPS + address of bakery            |
| Addons Price            | `FoodAndBeverageOffer.customization.groups[].options[].priceDelta` | object              | Price delta per add-on option      |
| Gift Type               | `FoodAndBeverageOffer.customization.groups[code=GIFT_PACKAGING]`   | customization group | Birthday / Wedding / None          |
| Custom Gift Note        | `FoodAndBeverageOffer.customization.groups[code=GIFT_NOTE]`        | customization group | **`[EXT]` textInput field needed** |
| Total Price             | `PriceSpecification.value` + `components[]`                        | PriceSpecification  | Sum of base + size delta + add-ons |
| Estimated Delivery Time | `Fulfillment.stages[code=DELIVERY].time.duration`                  | string (ISO 8601)   | **`[EXT]` warm-delivery estimate** |

### Fulfillment Fields

| Business Field      | Beckn Schema Path                               | Schema Type           | Notes                                |
| ------------------- | ----------------------------------------------- | --------------------- | ------------------------------------ |
| Delivery Mode       | `Fulfillment.mode.descriptor.code`              | FulfillmentMode       | `HOME_DELIVERY`                      |
| Distance Constraint | `RetailOffer.serviceability.distanceConstraint` | object                | Max delivery radius                  |
| Warm Delivery       | `Fulfillment.fulfillmentAttributes`             | **`[EXT]` WarmChain** | Temperature maintenance for hot cake |
| Distance-based Sort | BAP `search` intent with buyer GPS              | search intent         | BPP returns distance in response     |

## Schema Extensions Maybe Required

The following fields are **not** present in the standard Beckn v2.1 schemas and require extension via the `*Attributes` mechanism.

### `textInput` on Customization Option `[EXT]`

**Problem:** `FoodAndBeverageOffer.customization.groups[].options[]` supports selection of predefined options but has no mechanism for free-text input (needed for gift note messages).

**Extension:** Add an optional `textInput` field to the option schema.

### `warmChain` on Fulfillment Attributes `[EXT]`

**Problem:** Beckn defines cold-chain patterns for frozen/chilled goods, but there is no standard for maintaining warmth during delivery (e.g., freshly baked goods).

**Extension:** Add `warmChain` to `fulfillmentAttributes`, mirroring cold-chain conventions.

### `distanceFromRecipient` on Fulfillment Attributes `[EXT]`

**Problem:** The buyer wants to sort results by distance to the recipient's location. Standard Beckn has `serviceability.distanceConstraint.maxDistance` on the offer (a constraint), but no field to return the computed distance in search results.

**Extension:** Add `distanceFromRecipient` to `fulfillmentAttributes` in `on_search` responses.

### `estimatedDeliveryTime` on Fulfillment Attributes `[EXT]`

**Problem:** The standard `Fulfillment` schema tracks stages and states but does not expose a pre-order estimated delivery duration.

**Extension:** Add `estimatedDeliveryTime` to `fulfillmentAttributes`.

### `sortBy` on Search Intent `[EXT]`

**Problem:** The Beckn `search` intent allows filtering via resource/offer attributes but does not have a standardized sort directive.

**Extension:** Add `sortBy` to the search intent.

## Category Extension

The existing category taxonomy (from `beckn-packaged-foods-catalog.md`) covers packaged/processed foods. Kue Lapis as a fresh bakery product needs a new category branch.

```
Makanan Kemasan / Packaged Foods (cat-packaged-foods)
  +-- ... (existing categories)
  +-- Roti & Kue / Bakery & Cakes (cat-bakery)          [NEW]
        +-- Kue Tradisional / Traditional Cakes (cat-traditional-cakes)  [NEW]
        +-- Roti & Pastry / Bread & Pastry (cat-bread)   [NEW]
        +-- Kue Ulang Tahun / Birthday Cakes (cat-birthday-cakes)  [NEW]
```
