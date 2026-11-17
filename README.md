# FX.Rates: UiPath library

A reusable UiPath library with one activity, **Get Exchange Rate**, that returns the exchange rate between two currencies on a date. It uses ECB reference rates from the public [Frankfurter API](https://frankfurter.dev). No API key is needed.

Use it in any process that works with foreign currencies: invoices, purchase orders, expense reports, reconciliation or reporting.

## Install

1. Publish the library to your Orchestrator tenant feed, or get it from the feed if it is already there.
2. In your Studio project: **Manage Packages** → tenant feed → search **FX.Rates** → **Install**.
3. The activity appears in the Activities panel under **FX → Rates → GetExchangeRate**.

## Arguments

| Argument | Direction | Type | Description |
| --- | --- | --- | --- |
| `in_BaseCurrency` | In | String | 3-letter ISO code to convert from, e.g. `USD`. Case-insensitive |
| `in_QuoteCurrency` | In | String | 3-letter ISO code to convert to, e.g. `EUR` |
| `in_RateDate` | In | DateTime | Date of the rate. Must not be in the future |
| `out_Success` | Out | Boolean | `True` if a rate was returned |
| `out_Rate` | Out | Decimal | Exchange rate, e.g. `0.89367` |
| `out_RateDate` | Out | DateTime | Date of the rate actually used (a weekend returns the previous business day) |
| `out_ErrorMessage` | Out | String | Reason for failure; empty on success |

## Behaviour

- **Same currency** (e.g. EUR → EUR): returns rate `1` without calling the API.
- **Validation:** codes must be 3 letters and the date must not be in the future; otherwise it fails with "Invalid input" and makes no API call.
- **Retries:** up to 3 attempts, 2 seconds apart, for 5xx errors and timeouts only. 4xx errors are not retried.
- **The activity never throws.** Always check `out_Success` and `out_ErrorMessage`:

| Message | Cause |
| --- | --- |
| `Invalid input: …` | Bad currency code or a future date |
| `Currency or date not supported` | The API has no rate for this currency or date |
| `Rate service unavailable, try again later` | API error, timeout, or no response after retries |
| `No rate returned for <quote>` | The response held no rate for the target currency |

- **Logging:** Info at start and on success, Warning when the rate date differs from the requested date or validation fails, Error on failure. Full URLs and responses are not logged.

## Example

Low-code: add **Get Exchange Rate**, set `in_BaseCurrency = "USD"`, `in_QuoteCurrency = "EUR"`, `in_RateDate = New DateTime(2026, 11, 3)`, then use an **If** on `out_Success`.

The process project in this repo (`ConvertExpenseClaim.cs`) shows how to call it from a C# coded workflow.

## Requirements

- UiPath Studio STS 2026.0.199 or later; cross-platform library
- Internet access to `api.frankfurter.dev` from the robot

## Version history

| Version | Changes |
| --- | --- |
| 1.0.0 | First release, exported from the ExpenseClaimConverter project |
| 1.0.2 | Removed `ConvertExpenseClaim`, which was included by the export; library now contains only `GetExchangeRate` |

## Notes

- Rates are ECB reference rates, not bank or card rates.
- Built with UiPath Autopilot (coding agent) as part of Exercise 4 and reviewed by a person.
