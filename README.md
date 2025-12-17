## Structure Description

### Top Level

```json
{
  "data": {...},
  "version": 1,
  "generated_at": "ISO 8601 formatted date string",
}
```
#### Fields

- **`data`**: Object keyed by resulting item type names (e.g., "10MN Abyssal Afterburner"). Contains all abyssal item groups.
- **`version`**: Number indicating the version of the data Structure
- **`generated_at`**: ISO 8601 formatted date string (e.g., "2025-12-14T22:49:02.500268+00:00")

### ResultingGroup

Each key in `data` maps to a `ResultingGroup` object representing one type of abyssal item:

```json
{
  "source_mutator_groups": [...],
  "base_types": [...],
  "mutators": [...],
  "varying_attributes": [...],
  "min_max_attributes": [...],
}
```

#### Fields

##### `source_mutator_groups`

Combinations of base items and mutaplasmids that produce this abyssal type. Each entry represents actual items you own from that specific combination.

**Validation:** 
- Each `source_mutator_group[].source_type_id` has a corresponding entry in `base_types[]` with matching `id`.
- Each `source_mutator_group[].mutator_type_id` has a corresponding entry in `mutators[]` with matching `id`.

##### `base_types`

All possible base items that can be mutated to create this abyssal type. Provides reference stats for unmutated items.

##### `mutators`

All possible mutaplasmids that can create this abyssal type. Shows the mutation ranges each mutaplasmid applies.

##### `varying_attributes`

Defines which attributes change when items are mutated.

**Validation:**
- Each attribute ID must be unique

Note: Some attributes may have negative IDs. These are virtual/calculated attributes derived from other base attributes (e.g., Armor Repair Efficiency = Armor Hitpoints Repaired / Activation Cost)

##### `min_max_attributes`

Global minimum and maximum possible values across ALL combinations of base items and mutaplasmids for this abyssal type.

---

### source_mutator_group

Represents one specific combination of base item + mutaplasmid:

```json
{
  "source_type_id": 6005,
  "mutator_type_id": 47750,
  "attributes": [...],
  "dynamics": [...]
}
```

#### Fields

- **`source_type_id`**: Type ID of the base item used for mutation (e.g., 6005 = "10MN Monopropellant Enduring Afterburner")
  - **Validation:** Must exist in `base_types[]` with matching `id`

- **`mutator_type_id`**: Type ID of the mutaplasmid used (e.g., 47750 = "Decayed 10MN Afterburner Mutaplasmid")
  - **Validation:** Must exist in `mutators[]` with matching `id`

- **`attributes`**: Array of `AttributeRange` - theoretical min/max values possible from this specific base+mutator combination
  - Calculated as: `base_value * mutator_multiplier_range`
  - **Validation:** Contains exactly the same attribute IDs as `varying_attributes[]`
  - **Validation:** Each range falls within the corresponding `min_max_attributes[]`

- **`dynamics`**: Array of `DynamicItemData` - your actual owned items created from this combination
  - **Validation:** Each item's `attributes[]` contains exactly the same attribute IDs as `varying_attributes[]`
  - **Validation:** Each item's attribute values fall within the corresponding `attributes[]` range (within floating-point tolerance)

---

### DynamicItemData

Each item in the `dynamics` array represents an actual owned abyssal item:

```json
{
  "item_id": 1045054522562,
  "station_name": "Jita IV - Moon 4 - Caldari Navy Assembly Plant",
  "location_type": "station",
  "location_name": "2025-07-20 #1",
  "attributes": [
    {"id": 6, "value": 66.35},
    {"id": 20, "value": 136.34},
  ]
}
```

#### Fields

- **`item_id`**: Unique identifier for this specific item instance
- **`station_name`**: Name of the station/structure where this item is located
- **`location_type`**: Type of location_name
- **`location_name`**: Custom name of the location
- **`attributes`**: Array of `AttributeValue` - the actual rolled stats for this item
  - **Validation:** Contains exactly the same attribute IDs as `varying_attributes[]`
  - **Validation:** Each value falls within the parent `source_mutator_group.attributes[]` range

---

### BaseItemType

Reference data for unmutated base items:

```json
{
  "id": 6005,
  "name": "10MN Monopropellant Enduring Afterburner",
  "attributes": [
    {"id": 6, "value": 60.0},
    {"id": 20, "value": 125.0},
  ]
}
```

#### Fields

- **`id`**: Type ID of the base item
- **`name`**: Human-readable name of the base item
- **`attributes`**: Array of `AttributeValue` - baseline stats before mutation
  - **Validation:** Contains exactly the same attribute IDs as `varying_attributes[]`

---

### MutatorConcise

Reference data for mutaplasmids:

```json
{
  "id": 47750,
  "name": "Decayed 10MN Afterburner Mutaplasmid",
  "attributes": [
    {"id": 6, "min": 0.85, "max": 1.20},
    {"id": 20, "min": 0.95, "max": 1.10},
  ]
}
```

#### Fields

- **`id`**: Type ID of the mutaplasmid
- **`name`**: Human-readable name of the mutaplasmid
- **`attributes`**: Array of `AttributeRange` - multiplier ranges this mutaplasmid applies to base stats
  - **Validation:** Contains exactly the same attribute IDs as `varying_attributes[]`
  
---

### VaryingAttribute

Metadata about attributes that vary across abyssal items:

```json
{
  "id": 6,
  "name": "Activation Cost",
  "high_is_good": false
}
```

#### Fields
- **`id`**: Dogma attribute ID
- **`name`**: Human-readable attribute name
- **`high_is_good`**: Boolean indicating whether higher values are better
  - `true` = maximize this stat (e.g. "Maximum Velocity Bonus")
  - `false` = minimize this stat (e.g. "CPU Usage")

---

### AttributeValue

Represents a specific stat value:

```json
{
  "id": 6,
  "value": 66.35,
}
```

#### Fields

- **`id`**: Dogma attribute ID (matches an entry in `varying_attributes[]`)
  - **Validation:** The `id` must correspond to an entry in `varying_attributes[]` with matching `id`
- **`value`**: The actual numeric value of this attribute

**Used in:**
- `base_types[].attributes[]` - base item stats
- `dynamics[].attributes[]` - actual rolled stats on items

---

### AttributeRange

Represents min/max range for a stat

```json
{
  "id": 6,
  "min": 51.0,
  "max": 72.0
}
```

#### Fields

- **`id`**: Dogma attribute ID (matches an entry in `varying_attributes[]`)
- **`min`**: Minimum possible value
- **`max`**: Maximum possible value

**Used in:**
- `source_mutator_groups[].attributes[]` - theoretical range for a specific base+mutator combo
- `mutator[].attributes[]` - multiplier ranges applied by the mutaplasmid
- `min_max_attributes[]` - global min/max across all combinations

---

### Validation Rules

#### 1. ID Consistency

- Every `source_type_id` appears in `base_types[].id`
- Every `mutator_type_id` appears in `mutators[].id`

#### 2. Attribute Consistency

All attribute arrays contain exactly the same set of attribute IDs (defined in `varying_attributes[]`):

- `source_mutator_groups[].attributes[]`
- `source_mutator_groups[].dynamics[].attributes[]`
- `base_types[].attributes[]`
- `mutators[].attributes[]`
- `min_max_attributes[]`

#### 3. Value Range Validation

- `dynamics[].attributes[].value` falls within `source_mutator_groups[].attributes[].min/max` (within floating-point tolerance of ±0.000001)
- `source_mutator_groups[].attributes[]` ranges fall within `min_max_attributes[]` ranges
- Relationship: `source_mutator_groups[].attributes[]` = `base_types[].attributes[].value` * `mutators[].attributes[].min/max`
