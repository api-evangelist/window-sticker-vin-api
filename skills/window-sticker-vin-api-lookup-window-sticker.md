---
generated: '2026-09-20'
method: generated
name: Look up a vehicle's window sticker by VIN
description: >-
  Decode a 17-character US VIN, check whether an original factory window sticker (Monroney label)
  is available, and fetch the manufacturer PDF when it is.
api: openapi/window-sticker-vin-api-openapi.yml
operations: [getVin, getSticker]
source: >-
  Grounded in openapi/window-sticker-vin-api-openapi.yml; operationIds getVin and getSticker
  verified verbatim. Cross-cutting rules cite ../conventions/, ../errors/, ../rate-limits/.
---

# Look up a vehicle's window sticker by VIN

Given a VIN, return the decoded vehicle specs and the original factory window sticker PDF when one exists.

## Auth
- None. Keyless, public, CORS-enabled. See `authentication/window-sticker-vin-api-authentication.yml`.

## Fair use
- No API key and no hard quota, but do not enumerate VINs at speed. There are no rate-limit
  headers to read; throttle voluntarily. See `rate-limits/window-sticker-vin-api-rate-limits.yml`.

## Steps
1. **Validate the VIN locally** — must match `^[A-HJ-NPR-Z0-9]{17}$` (17 chars, never I/O/Q). A bad
   VIN returns `400` from `getVin`.
2. **Decode and check availability** — `getVin` (`GET /api/v1/vin/{vin}`). Read `vehicle` for the
   NHTSA vPIC decode and `windowSticker.supported` / `windowSticker.available` to decide whether a
   label exists. Add `?sticker=false` when you only need the decode (faster). A nullable `warning`
   string signals a partial decode; it is not an error.
3. **Fetch the PDF** — only when `windowSticker.available` is true, call `getSticker`
   (`GET /api/sticker/{vin}`). It returns the unmodified manufacturer `application/pdf`. Add
   `?download=1` for a Content-Disposition attachment.

## Errors
- `400` from `getVin` — VIN failed the pattern; fix the input.
- `404` from `getSticker` — no label published for this VIN (common on older vehicles, or makes
  with no public label service). Expected, not a failure — the decode from step 2 is still valid.
  See `errors/window-sticker-vin-api-problem-types.yml`.

## Notes
- Coverage is per VIN, not per make: a supported make still returns "no label" for roughly a
  quarter of lookups. Always gate the PDF call on `windowSticker.available`.
- Results are cached because a VIN's window sticker never changes.
