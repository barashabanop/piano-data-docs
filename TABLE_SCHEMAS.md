# Piano BigQuery Table Schemas

> Complete DDL for all bronze and gold tables.

## Table of Contents

- [Bronze Tables](#bronze-tables)
  - [piano_subscription_log](#piano_subscription_log)
  - [piano_conversion_events (NEW)](#piano_conversion_events-new)
  - [piano_conversion_exposures_* (9 tables)](#piano_conversion_exposures-9-tables)
- [Gold Tables](#gold-tables)
  - [fact_piano_conversions](#fact_piano_conversions)
  - [fact_piano_subscriptions](#fact_piano_subscriptions)
  - [fact_piano_exposures](#fact_piano_exposures)
  - [dim_piano_terms](#dim_piano_terms)
  - [dim_piano_resources](#dim_piano_resources)
  - [dim_piano_promotions](#dim_piano_promotions)

---

## Bronze Tables

### piano_subscription_log

Source: VX `schedule/vx/subscriptionLog` export

```sql
CREATE TABLE IF NOT EXISTS bronze.piano_subscription_log (
  -- Subscription identity
  subscription_id                STRING NOT NULL,
  summary                        STRING,

  -- Dates
  create_date                    TIMESTAMP,
  start_date                     TIMESTAMP,
  end_date                       TIMESTAMP,
  days_subscribed                INT64,

  -- Status
  subscription_status            STRING,
  upgrade_status                 STRING,
  trial_status                   STRING,
  trial_period_end_date          TIMESTAMP,
  trial_price                    FLOAT64,
  regular_price                  FLOAT64,
  auto_renew                     BOOLEAN,
  auto_renew_disablement_date    TIMESTAMP,
  last_active_date               TIMESTAMP,

  -- Engagement (analytics-level, export-only)
  logins_last_30_days            INT64,
  sessions_last_30_days          INT64,
  pageviews_last_30_days         INT64,

  -- Billing
  billing_period                 STRING,
  total_charged                  FLOAT64,
  charge_count                   INT64,
  total_refunded                 FLOAT64,
  payment_source                 STRING,
  last_billing_date              TIMESTAMP,
  next_billing_date              TIMESTAMP,
  renewed                        BOOLEAN,

  -- Grace period
  currently_in_grace_period      BOOLEAN,
  grace_period_start_date        TIMESTAMP,
  grace_period_extended_to       TIMESTAMP,

  -- User
  user_id_uid                    STRING,
  user_email                     STRING,
  first_name                     STRING,
  last_name                      STRING,

  -- Access
  access_expiration_date         TIMESTAMP,
  resource_id_rid                STRING,
  resource_name                  STRING,

  -- Term
  term_id                        STRING,
  term_name                      STRING,
  term_type                      STRING,

  -- Attribution
  template_id                    STRING,
  template_name                  STRING,
  offer_id                       STRING,
  offer_name                     STRING,
  experience_id                  STRING,
  experience_name                STRING,
  campaign_codes                 STRING,
  promo_code                     STRING,

  -- Conversion context
  conversion_city                STRING,
  conversion_state               STRING,
  conversion_country             STRING,
  device                         STRING,
  url                            STRING,
  cleaned_url                    STRING,
  utm_parameters                 STRING,

  -- Address (PII - consider exclusion)
  company_name                   STRING,
  name_first_and_last            STRING,
  address_1                      STRING,
  address_2                      STRING,
  address_city                   STRING,
  address_state                  STRING,
  address_country                STRING,
  postal_code                    STRING,
  phone                          STRING,

  -- Tax
  psc_subscriber_number          STRING,
  tax                            FLOAT64,
  tax_base                       FLOAT64,
  tax_rate                       FLOAT64,
  tax_country                    STRING,
  tax_state_province             STRING,
  billing_postal_code            STRING,

  -- Misc
  shared_subscriptions           INT64,
  migrated_to_piano              BOOLEAN,
  migrated_date                  TIMESTAMP,
  created_via_upgrade            BOOLEAN,
  upgrade_from_subscription_id   STRING,
  period_name                    STRING,
  access_period                  STRING,
  period_count                   INT64,

  -- Pipeline metadata
  report_date                    DATE NOT NULL,
  region                         STRING,
  site_name                      STRING,
  ingested_at                    TIMESTAMP,
  batch                          STRING
)
PARTITION BY report_date;
```

### piano_conversion_events (NEW)

Source: `/conversion/list` enriched with `/conversion/data/get`

```sql
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_events (
  -- Conversion identity
  term_conversion_id             STRING NOT NULL,
  type                           STRING,           -- Registration / Dynamic
  aid                            STRING,
  create_date                    TIMESTAMP,

  -- User (from /conversion/list)
  uid                            STRING,
  user_email                     STRING,
  user_first_name                STRING,
  user_last_name                 STRING,

  -- Term (from /conversion/list)
  term_id                        STRING,
  term_name                      STRING,
  term_type                      STRING,
  term_description               STRING,
  resource_rid                   STRING,
  resource_name                  STRING,

  -- Access (from /conversion/list)
  access_id                      STRING,
  access_granted                 BOOLEAN,
  access_start_date              TIMESTAMP,
  access_expire_date             TIMESTAMP,

  -- Subscription (from /conversion/list, null for registrations)
  subscription_id                STRING,
  subscription_status            STRING,
  subscription_auto_renew        BOOLEAN,
  subscription_payment_method    STRING,
  subscription_charge_count      INT64,
  subscription_is_in_trial       BOOLEAN,
  subscription_start_date        TIMESTAMP,
  subscription_end_date          TIMESTAMP,

  -- Payment (from /conversion/list, null for registrations)
  payment_user_payment_id        STRING,
  payment_amount                 FLOAT64,
  payment_currency               STRING,
  payment_method                 STRING,
  payment_state                  STRING,
  payment_renewal                BOOLEAN,

  -- Promo (from /conversion/list)
  promo_code                     STRING,

  -- Schedule/Period (from /conversion/list)
  schedule_name                  STRING,
  period_name                    STRING,
  period_begin_date              TIMESTAMP,
  period_end_date                TIMESTAMP,

  -- Behavioral context (from /conversion/data/get)
  url                            STRING,
  browser                        STRING,
  platform                       STRING,           -- mobile / desktop / tablet
  operating_system               STRING,
  user_country                   STRING,
  user_region                    STRING,
  user_city                      STRING,
  zip                            STRING,
  latitude                       STRING,
  longitude                      STRING,
  user_agent                     STRING,
  locale                         STRING,
  hour                           STRING,
  tags                           STRING,
  template_id                    STRING,
  offer_id                       STRING,
  offer_template_id              STRING,
  campaigns                      STRING,           -- comma-separated
  content_created                STRING,
  content_author                 STRING,
  content_section                STRING,

  -- Tracking
  browser_id                     STRING,

  -- Pipeline metadata
  report_date                    DATE NOT NULL,
  region                         STRING,
  site_name                      STRING,
  ingested_at                    TIMESTAMP,
  batch                          STRING
)
PARTITION BY report_date;
```

### piano_conversion_exposures_* (9 tables)

Each table keeps its current column structure. Only the names change.

```sql
-- conversion-device.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_device (
  device           STRING,
  operating_system STRING,
  browser          STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  currency         STRING,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-page.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_page (
  url              STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-location.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_location (
  country          STRING,
  location         STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  currency         STRING,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-tag.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_tag (
  tag              STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-template.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_template (
  template_name         STRING,
  template_variant_name STRING,
  exposures             INT64,
  conversions           INT64,
  conversion_rate       FLOAT64,
  currency              STRING,
  report_date           DATE NOT NULL,
  region                STRING,
  site_name             STRING,
  ingested_at           TIMESTAMP,
  batch                 STRING
) PARTITION BY report_date;

-- conversion-term.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_term (
  template         STRING,
  offer            STRING,
  term             STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  currency         STRING,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-termType.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_term_type (
  term_type        STRING,
  term_name        STRING,
  exposures        INT64,
  conversions      INT64,
  conversion_rate  FLOAT64,
  currency         STRING,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-promotion.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_promotion (
  promotion        STRING,
  conversions      INT64,
  report_date      DATE NOT NULL,
  region           STRING,
  site_name        STRING,
  ingested_at      TIMESTAMP,
  batch            STRING
) PARTITION BY report_date;

-- conversion-campaignCode.csv
CREATE TABLE IF NOT EXISTS bronze.piano_conversion_exposures_campaign_code (
  campaign_code                       STRING,
  attributed_conversions              INT64,
  average_time_to_conversion_seconds  FLOAT64,
  report_date                         DATE NOT NULL,
  region                              STRING,
  site_name                           STRING,
  ingested_at                         TIMESTAMP,
  batch                               STRING
) PARTITION BY report_date;
```

---

## Gold Tables

### fact_piano_conversions

```sql
CREATE TABLE IF NOT EXISTS gold.fact_piano_conversions (
  term_conversion_id    STRING NOT NULL,
  conversion_date       DATE NOT NULL,
  conversion_timestamp  TIMESTAMP,
  aid                   STRING,
  site_name             STRING,
  region                STRING,

  -- User
  uid                   STRING,

  -- Term
  term_id               STRING,
  term_name             STRING,
  term_type             STRING,
  resource_name         STRING,

  -- Subscription
  subscription_id       STRING,
  subscription_status   STRING,
  is_auto_renew         BOOLEAN,
  is_trial              BOOLEAN,

  -- Revenue
  payment_amount        FLOAT64,
  payment_currency      STRING,
  payment_method        STRING,
  is_renewal            BOOLEAN,

  -- Behavioral context
  page_url              STRING,
  device_type           STRING,
  browser               STRING,
  operating_system      STRING,
  country               STRING,
  region_state          STRING,
  city                  STRING,
  zip_code              STRING,
  latitude              FLOAT64,
  longitude             FLOAT64,

  -- Attribution
  template_id           STRING,
  template_name         STRING,         -- resolved from dim_piano_terms or exports
  offer_id              STRING,
  offer_name            STRING,         -- resolved from dim or exports
  campaign_code         STRING,
  promo_code            STRING,
  tags                  STRING,

  -- Content metadata
  content_author        STRING,
  content_section       STRING
)
PARTITION BY conversion_date;
```

### fact_piano_subscriptions

```sql
CREATE TABLE IF NOT EXISTS gold.fact_piano_subscriptions (
  subscription_id       STRING NOT NULL,
  report_date           DATE NOT NULL,
  aid                   STRING,
  site_name             STRING,
  region                STRING,

  -- User
  uid                   STRING,

  -- Lifecycle
  status                STRING,
  start_date            DATE,
  end_date              DATE,
  days_subscribed       INT64,
  is_auto_renew         BOOLEAN,
  is_trial              BOOLEAN,
  is_upgraded           BOOLEAN,
  is_renewed            BOOLEAN,
  is_in_grace_period    BOOLEAN,

  -- Revenue
  regular_price         FLOAT64,
  total_charged         FLOAT64,
  charge_count          INT64,
  total_refunded        FLOAT64,
  payment_source        STRING,
  billing_period        STRING,

  -- Behavioral context
  page_url              STRING,
  device_type           STRING,
  country               STRING,
  region_state          STRING,

  -- Attribution
  term_id               STRING,
  term_name             STRING,
  term_type             STRING,
  resource_id           STRING,
  resource_name         STRING,
  template_name         STRING,
  offer_name            STRING,
  campaign_code         STRING,
  promo_code            STRING,

  -- Engagement (export-only metrics)
  logins_last_30d       INT64,
  sessions_last_30d     INT64,
  pageviews_last_30d    INT64
)
PARTITION BY report_date;
```

### fact_piano_exposures

```sql
CREATE TABLE IF NOT EXISTS gold.fact_piano_exposures (
  report_date           DATE NOT NULL,
  aid                   STRING NOT NULL,
  site_name             STRING,
  region                STRING,
  dimension_type        STRING NOT NULL,    -- device / page / location / tag / template / term / term_type / promotion / campaign_code
  dimension_key         STRING NOT NULL,    -- composite key (e.g. "mobile|android|chrome")
  dimension_value_1     STRING,             -- first dimension (e.g. "mobile" or URL or country)
  dimension_value_2     STRING,             -- second dimension (e.g. "android" or region)
  dimension_value_3     STRING,             -- third dimension (e.g. "chrome")
  exposures             INT64,
  conversions           INT64,
  conversion_rate       FLOAT64,
  currency              STRING
)
PARTITION BY report_date;
```

### dim_piano_terms

```sql
CREATE OR REPLACE TABLE gold.dim_piano_terms (
  term_id               STRING NOT NULL,
  aid                   STRING,
  name                  STRING,
  type                  STRING,
  description           STRING,
  resource_id           STRING,
  resource_name         STRING,
  payment_currency      STRING,
  payment_first_price   FLOAT64,
  billing_descriptor    STRING,
  collect_address       BOOLEAN,
  create_date           TIMESTAMP,
  update_date           TIMESTAMP,
  synced_at             TIMESTAMP
);
```

### dim_piano_resources

```sql
CREATE OR REPLACE TABLE gold.dim_piano_resources (
  rid                   STRING NOT NULL,
  aid                   STRING,
  name                  STRING,
  description           STRING,
  type                  STRING,
  type_label            STRING,
  external_id           STRING,
  publish_date          TIMESTAMP,
  create_date           TIMESTAMP,
  update_date           TIMESTAMP,
  disabled              BOOLEAN,
  deleted               BOOLEAN,
  synced_at             TIMESTAMP
);
```

### dim_piano_promotions

```sql
CREATE OR REPLACE TABLE gold.dim_piano_promotions (
  promotion_id          STRING NOT NULL,
  aid                   STRING,
  name                  STRING,
  discount_type         STRING,
  percentage_discount   FLOAT64,
  start_date            TIMESTAMP,
  end_date              TIMESTAMP,
  new_customers_only    BOOLEAN,
  uses_allowed          INT64,
  unlimited_uses        BOOLEAN,
  can_be_applied_on_renewal BOOLEAN,
  synced_at             TIMESTAMP
);
```
