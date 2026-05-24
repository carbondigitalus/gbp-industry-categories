# Industry Categories

Reference data for industries/business categories sourced from the Google Business Profile category list (2026 revision). Provided in multiple formats so it can be consumed by databases, build pipelines, and frontends without re-parsing.

## Files

| File                                | Format                                               | Purpose                                                                                     |
| ----------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `list.txt`                          | Plain text, one entry per line                       | Original source, no slugs. The header line at the top is descriptive prose, not a category. |
| `industry-list.ts`                  | TypeScript module exporting `industryList: string[]` | Drop-in import for TS/JS apps.                                                              |
| `industry-list.json`                | JSON string array                                    | Generic interchange format for any language.                                                |
| `industry-list-postgresql-seed.sql` | Postgres `INSERT` script                             | Seeds a table with `(name, slug)` columns.                                                  |

## Counts

- **4,046** entries in the source.
- **4,045** unique by name (`Swimming pool` appears twice in the source — deduped to one row in the SQL output; preserved as-is in the text/JSON/TS files).
- **0** slug collisions across the deduped set.

## Schema (for the SQL seed)

The SQL script targets a table named `industry_category` with at least these columns:

```sql
CREATE TABLE industry_category (
  id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL
  -- Optional: UNIQUE (slug) — there are zero collisions in the seed data,
  -- so a unique constraint is safe to add.
);
```

If your table is named differently (e.g., `industry_categories` plural, or PascalCase), edit the `INSERT INTO` line at the top of `industry-list-postgresql-seed.sql`.

## Slug rules

Slugs in `industry-list.ts`, the JSON, and the SQL seed are generated with this algorithm:

1. Unicode NFKD normalization, then strip combining marks
   (`Açaí` → `Acai`, `Québécois` → `Quebecois`, `Bahá’í` → `Bahai`).
2. Drop apostrophes — both ASCII `'` and curly `’`
   (`Children’s` → `Childrens`, not `children-s`).
3. Lowercase.
4. Replace runs of non-alphanumeric characters with a single hyphen.
5. Trim leading/trailing hyphens.

Examples:

| Name                                        | Slug                                      |
| ------------------------------------------- | ----------------------------------------- |
| `Açaí shop`                                 | `acai-shop`                               |
| `Bahá’í house of worship`                   | `bahai-house-of-worship`                  |
| `Children’s hospital`                       | `childrens-hospital`                      |
| `Hauptschule (lower-tier secondary school)` | `hauptschule-lower-tier-secondary-school` |
| `3D printing service`                       | `3d-printing-service`                     |
| `Bed & breakfast`                           | `bed-breakfast`                           |
| `Po’ boys restaurant`                       | `po-boys-restaurant`                      |

## Notes

- The source list contains entries that look like duplicates but are intentionally distinct in Google's taxonomy (e.g., `Swimming facility` vs `Swimming pool` vs `Public swimming pool`). Only one true duplicate exists: `Swimming pool` appears twice.
- Several entries use the typographic apostrophe `’` (U+2019) rather than ASCII `'`. Both are stripped by the slugify step, but consumers reading `name` directly should expect U+2019.
- Entries are stored in the order Google publishes them (roughly alphabetical, with a few outliers). The SQL seed preserves source order.
