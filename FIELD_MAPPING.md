# Field-by-Field Mapping

> Maps every column from every CSV export to its API equivalent and final BQ column name.

## Subscription Log CSV -> API -> BQ


| #     | CSV Column                  | API Equivalent          | API Endpoint                            | BQ Column (bronze)            | BQ Column (gold)     | Notes                      |
| ----- | --------------------------- | ----------------------- | --------------------------------------- | ----------------------------- | -------------------- | -------------------------- |
| 1     | Subscription ID             | `subscription_id`       | `/subscription/list`                    | `subscription_id`             | `subscription_id`    | PK                         |
| 2     | Summary                     | --                      | --                                      | `summary`                     | --                   | Dropped in gold            |
| 3     | Create date                 | `create_date`           | `/subscription/list`                    | `create_date`                 | --                   | Used to derive report_date |
| 4     | Start date                  | `start_date`            | `/subscription/list`                    | `start_date`                  | `start_date`         |                            |
| 5     | End date                    | `end_date`              | `/subscription/list`                    | `end_date`                    | `end_date`           |                            |
| 6     | Days subscribed             | computed                | --                                      | `days_subscribed`             | `days_subscribed`    |                            |
| 7     | Subscription status         | `status`                | `/subscription/list`                    | `subscription_status`         | `status`             |                            |
| 8     | Upgrade status              | --                      | `/export/create/termChangeReportExport` | `upgrade_status`              | `is_upgraded`        | Cast to boolean            |
| 9     | Trial status                | `is_in_trial`           | `/subscription/list`                    | `trial_status`                | `is_trial`           | Cast to boolean            |
| 10    | Trial period end date       | --                      | `/subscription/list`                    | `trial_period_end_date`       | --                   |                            |
| 11    | Trial price                 | --                      | `/term/get`                             | `trial_price`                 | --                   |                            |
| 12    | Regular price               | `payment_first_price`   | `/term/get`                             | `regular_price`               | `regular_price`      |                            |
| 13    | Auto-renew                  | `auto_renew`            | `/subscription/list`                    | `auto_renew`                  | `is_auto_renew`      |                            |
| 14    | Auto-renew disablement date | --                      | `/subscription/list`                    | `auto_renew_disablement_date` | --                   |                            |
| 15    | Last active date            | `last_visit`            | `/user/get`                             | `last_active_date`            | --                   |                            |
| 16    | Logins last 30 days         | --                      | **NOT IN API**                          | `logins_last_30_days`         | `logins_last_30d`    | Export-only                |
| 17    | Sessions last 30 days       | --                      | **NOT IN API**                          | `sessions_last_30_days`       | `sessions_last_30d`  | Export-only                |
| 18    | Pageviews last 30 days      | --                      | **NOT IN API**                          | `pageviews_last_30_days`      | `pageviews_last_30d` | Export-only                |
| 19    | Billing period              | --                      | `/term/get`                             | `billing_period`              | `billing_period`     |                            |
| 20    | Total charged               | --                      | computed from payments                  | `total_charged`               | `total_charged`      |                            |
| 21    | Charge count                | `charge_count`          | `/subscription/list`                    | `charge_count`                | `charge_count`       |                            |
| 22    | Total refunded              | --                      | `/payment/get`                          | `total_refunded`              | `total_refunded`     |                            |
| 23    | Payment source              | `payment_method`        | `/payment/get`                          | `payment_source`              | `payment_source`     |                            |
| 24    | Last billing date           | --                      | `/subscription/list`                    | `last_billing_date`           | --                   |                            |
| 25    | Next billing date           | `next_bill_date`        | `/subscription/list`                    | `next_billing_date`           | --                   |                            |
| 26    | Renewed                     | --                      | computed                                | `renewed`                     | `is_renewed`         | Cast to boolean            |
| 27    | Currently in grace period   | --                      | `/subscription/list`                    | `currently_in_grace_period`   | `is_in_grace_period` |                            |
| 28    | Grace period start date     | --                      | `/subscription/list`                    | `grace_period_start_date`     | --                   |                            |
| 29    | Grace period extended to    | --                      | `/subscription/list`                    | `grace_period_extended_to`    | --                   |                            |
| 30    | User ID (UID)               | `uid`                   | `/user/get`                             | `user_id_uid`                 | `uid`                |                            |
| 31    | User email                  | `email`                 | `/user/get`                             | `user_email`                  | --                   | PII, excluded from gold    |
| 32    | First name                  | `first_name`            | `/user/get`                             | `first_name`                  | --                   | PII, excluded from gold    |
| 33    | Last name                   | `last_name`             | `/user/get`                             | `last_name`                   | --                   | PII, excluded from gold    |
| 34    | Access expiration date      | `expire_date`           | `/user/access/list`                     | `access_expiration_date`      | --                   |                            |
| 35    | Resource ID (RID)           | `rid`                   | `/resource/get`                         | `resource_id_rid`             | `resource_id`        | FK to dim                  |
| 36    | Resource name               | `name`                  | `/resource/get`                         | `resource_name`               | `resource_name`      |                            |
| 37    | Term ID                     | `term_id`               | `/term/get`                             | `term_id`                     | `term_id`            | FK to dim                  |
| 38    | Term name                   | `name`                  | `/term/get`                             | `term_name`                   | `term_name`          |                            |
| 39    | Term type                   | `type`                  | `/term/get`                             | `term_type`                   | `term_type`          |                            |
| 40    | Template ID                 | `template_id`           | `/conversion/data/get`                  | `template_id`                 | --                   | Resolve via dim            |
| 41    | Template name               | --                      | **export only**                         | `template_name`               | `template_name`      |                            |
| 42    | Offer ID                    | `offer_id`              | `/conversion/data/get`                  | `offer_id`                    | --                   | Resolve via dim            |
| 43    | Offer name                  | --                      | **export only**                         | `offer_name`                  | `offer_name`         |                            |
| 44    | Experience ID               | --                      | **NOT IN API**                          | `experience_id`               | --                   | Piano Composer internal    |
| 45    | Experience name             | --                      | **NOT IN API**                          | `experience_name`             | --                   | Piano Composer internal    |
| 46    | Campaign codes              | `campaigns`             | `/conversion/data/get`                  | `campaign_codes`              | `campaign_code`      |                            |
| 47    | Promo code                  | `promo_code.code`       | `/conversion/list`                      | `promo_code`                  | `promo_code`         |                            |
| 48    | Conversion city             | `user_city`             | `/conversion/data/get`                  | `conversion_city`             | --                   |                            |
| 49    | Conversion state            | `user_region`           | `/conversion/data/get`                  | `conversion_state`            | `region_state`       |                            |
| 50    | Conversion country          | `user_country`          | `/conversion/data/get`                  | `conversion_country`          | `country`            |                            |
| 51    | Device                      | `platform`              | `/conversion/data/get`                  | `device`                      | `device_type`        |                            |
| 52    | URL                         | `url`                   | `/conversion/data/get`                  | `url`                         | `page_url`           |                            |
| 53    | Cleaned URL                 | derived                 | --                                      | `cleaned_url`                 | --                   |                            |
| 54    | UTM parameters              | derived                 | --                                      | `utm_parameters`              | --                   |                            |
| 55-70 | Address / Tax fields        | `/subscription/address` | various                                 | kept in bronze                | --                   | PII, excluded from gold    |
| 71-78 | Migration / Period fields   | --                      | various                                 | kept in bronze                | partially            |                            |


---

## Conversion Export CSVs -> API -> BQ

### conversion-device.csv


| CSV Column       | API Equivalent                | API Endpoint           | BQ Column (bronze) | BQ Column (gold)    |
| ---------------- | ----------------------------- | ---------------------- | ------------------ | ------------------- |
| Device           | `platform`                    | `/conversion/data/get` | `device`           | `dimension_value_1` |
| Operating System | `operating_system`            | `/conversion/data/get` | `operating_system` | `dimension_value_2` |
| Browser          | `browser`                     | `/conversion/data/get` | `browser`          | `dimension_value_3` |
| Exposures        | **NOT IN API**                | **export only**        | `exposures`        | `exposures`         |
| Conversions      | count from `/conversion/list` | `/conversion/list`     | `conversions`      | `conversions`       |
| Conversion rate  | computed                      | --                     | `conversion_rate`  | `conversion_rate`   |
| Currency         | `payment_currency`            | `/term/get`            | `currency`         | `currency`          |


### conversion-page.csv


| CSV Column      | API Equivalent | API Endpoint           | BQ Column (bronze) | BQ Column (gold)    |
| --------------- | -------------- | ---------------------- | ------------------ | ------------------- |
| URL             | `url`          | `/conversion/data/get` | `url`              | `dimension_value_1` |
| Exposures       | **NOT IN API** | **export only**        | `exposures`        | `exposures`         |
| Conversions     | count          | `/conversion/list`     | `conversions`      | `conversions`       |
| Conversion rate | computed       | --                     | `conversion_rate`  | `conversion_rate`   |


### conversion-location.csv


| CSV Column      | API Equivalent     | API Endpoint           | BQ Column (bronze) | BQ Column (gold)    |
| --------------- | ------------------ | ---------------------- | ------------------ | ------------------- |
| Country         | `user_country`     | `/conversion/data/get` | `country`          | `dimension_value_1` |
| Location        | `user_region`      | `/conversion/data/get` | `location`         | `dimension_value_2` |
| Exposures       | **NOT IN API**     | **export only**        | `exposures`        | `exposures`         |
| Conversions     | count              | `/conversion/list`     | `conversions`      | `conversions`       |
| Conversion rate | computed           | --                     | `conversion_rate`  | `conversion_rate`   |
| Currency        | `payment_currency` | `/term/get`            | `currency`         | `currency`          |


### conversion-tag.csv


| CSV Column      | API Equivalent | API Endpoint           | BQ Column (bronze) | BQ Column (gold)    |
| --------------- | -------------- | ---------------------- | ------------------ | ------------------- |
| Tag             | `tags`         | `/conversion/data/get` | `tag`              | `dimension_value_1` |
| Exposures       | **NOT IN API** | **export only**        | `exposures`        | `exposures`         |
| Conversions     | count          | `/conversion/list`     | `conversions`      | `conversions`       |
| Conversion rate | computed       | --                     | `conversion_rate`  | `conversion_rate`   |


### conversion-template.csv


| CSV Column            | API Equivalent                | API Endpoint           | BQ Column (bronze)      | BQ Column (gold)    |
| --------------------- | ----------------------------- | ---------------------- | ----------------------- | ------------------- |
| Template name         | `template_id` (ID only)       | `/conversion/data/get` | `template_name`         | `dimension_value_1` |
| Template variant name | `offer_template_id` (ID only) | `/conversion/data/get` | `template_variant_name` | `dimension_value_2` |
| Exposures             | **NOT IN API**                | **export only**        | `exposures`             | `exposures`         |
| Conversions           | count                         | `/conversion/list`     | `conversions`           | `conversions`       |
| Conversion rate       | computed                      | --                     | `conversion_rate`       | `conversion_rate`   |
| Currency              | `payment_currency`            | `/term/get`            | `currency`              | `currency`          |


### conversion-term.csv


| CSV Column      | API Equivalent     | API Endpoint           | BQ Column (bronze) | BQ Column (gold)    |
| --------------- | ------------------ | ---------------------- | ------------------ | ------------------- |
| Template        | `template_id`      | `/conversion/data/get` | `template`         | `dimension_value_1` |
| Offer           | `offer_id`         | `/conversion/data/get` | `offer`            | `dimension_value_2` |
| Term            | `term.name`        | `/conversion/list`     | `term`             | `dimension_value_3` |
| Exposures       | **NOT IN API**     | **export only**        | `exposures`        | `exposures`         |
| Conversions     | count              | `/conversion/list`     | `conversions`      | `conversions`       |
| Conversion rate | computed           | --                     | `conversion_rate`  | `conversion_rate`   |
| Currency        | `payment_currency` | `/term/get`            | `currency`         | `currency`          |


### conversion-termType.csv


| CSV Column      | API Equivalent     | API Endpoint       | BQ Column (bronze) | BQ Column (gold)    |
| --------------- | ------------------ | ------------------ | ------------------ | ------------------- |
| Term type       | `type`             | `/conversion/list` | `term_type`        | `dimension_value_1` |
| Term name       | `term.name`        | `/conversion/list` | `term_name`        | `dimension_value_2` |
| Exposures       | **NOT IN API**     | **export only**    | `exposures`        | `exposures`         |
| Conversions     | count              | `/conversion/list` | `conversions`      | `conversions`       |
| Conversion rate | computed           | --                 | `conversion_rate`  | `conversion_rate`   |
| Currency        | `payment_currency` | `/term/get`        | `currency`         | `currency`          |


### conversion-promotion.csv


| CSV Column  | API Equivalent             | API Endpoint                           | BQ Column (bronze) | BQ Column (gold)    |
| ----------- | -------------------------- | -------------------------------------- | ------------------ | ------------------- |
| Promotion   | `promo_code.code` / `name` | `/conversion/list` + `/promotion/list` | `promotion`        | `dimension_value_1` |
| Conversions | count                      | `/conversion/list`                     | `conversions`      | `conversions`       |


### conversion-campaignCode.csv


| CSV Column                 | API Equivalent | API Endpoint           | BQ Column (bronze)                   | BQ Column (gold)    |
| -------------------------- | -------------- | ---------------------- | ------------------------------------ | ------------------- |
| Campaign code              | `campaigns`    | `/conversion/data/get` | `campaign_code`                      | `dimension_value_1` |
| Attributed conversions     | count          | `/conversion/list`     | `attributed_conversions`             | `conversions`       |
| Average time to conversion | computed       | --                     | `average_time_to_conversion_seconds` | --                  |


