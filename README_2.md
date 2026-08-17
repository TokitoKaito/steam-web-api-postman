# Steam Web API — Postman Collection

Manual API testing of the public Steam Web API: 6 requests, 26 assertions, covering positive, negative, boundary and parameter-effect checks.

Companion to [steam-store-search-qa](https://github.com/TokitoKaito/steam-store-search-qa) — manual UI testing of the Steam store search. Same product, different layer.

## Contents

| File | Description |
|---|---|
| `steam-web-api.postman_collection.json` | The collection: 6 requests, 26 assertions, per-request documentation |
| `steam.postman_environment.json` | Environment with the `baseUrl` variable |

## Requests

| # | Request | Type of check | Assertions |
|---|---|---|---|
| 1 | Current players — valid appid | Positive: response contract | 5 |
| 2 | Current players — non-existent appid | Negative: unknown data | 5 |
| 3 | Current players — missing appid | Negative: required parameter omitted | 3 |
| 4 | News — count=3 | Parameter effect | 4 |
| 5 | News — count=0 | Boundary value | 4 |
| 6 | Global achievement percentages | Response contract, nested structure | 5 |

Endpoints used: `ISteamUserStats/GetNumberOfCurrentPlayers`, `ISteamNews/GetNewsForApp`, `ISteamUserStats/GetGlobalAchievementPercentagesForApp`.

## How to run

1. In Postman: **Import** → select both JSON files.
2. Select the **Steam** environment in the environment selector.
3. Open a request and press **Send**, or run the whole collection via **Run collection**.

Run the collection manually. Requests that produce HTTP 403 can lead to the client IP being temporarily blocked, so the collection is not intended to be looped in the Collection Runner.

## Why this API

The Steam Web API is publicly documented and several of its methods work without an API key, so the expected behaviour of each response is defined and can be asserted against.

Undocumented endpoints were deliberately avoided. Without a specification there is no oracle, and the expected result would have to be invented — which produces tests that verify the author's assumptions rather than the product.

## Observations

**1. Errors are returned as HTML, successes as JSON.**
A request missing a required parameter returns `400` with an HTML body, while successful responses are JSON. Clients therefore have to handle two formats. No documented requirement covers this, so it is recorded as a question rather than as a defect.

This also has a practical consequence for the tests: calling `pm.response.json()` on an HTML body throws, and the test fails because of the script rather than because of the API. Request 3 uses `pm.response.text()` for this reason.

**2. `ISteamApps/GetAppList/v2` returns 404.**
The message is `Method 'GetAppList' not found in interface 'ISteamApps'`. The official documentation marks the method as deprecated in favour of `IStoreService/GetAppList`, which requires an API key. The endpoint was replaced in this collection by `GetGlobalAchievementPercentagesForApp`.

## Notes on method

**Assertions describe invariants, not snapshots.** `player_count` changes every minute and achievement percentages shift constantly, so the tests assert type and range rather than specific values. An assertion pinned to today's number is a test that fails tomorrow for no reason.

**Type and meaning are asserted separately.** `percent` is returned as a string, not a number. One assertion records the actual contract (`to.be.a("string")`), a second records the meaning (`Number(percent)` within 0–100). If either changes, it is immediately clear which one.

**One layer is deliberately left unchecked.** Request 6 has no parameter-effect assertion: the response does not echo `gameid`, so there is nothing in the body to compare the request parameter against. Any assertion added there could never fail, and a test that cannot fail is not a test.

**Response time policy.** Every request asserts a response time below 10000 ms. This is a clearly-abnormal ceiling, not a performance requirement — no response-time SLA has been agreed for this API, so a stricter threshold would be an invented expectation. Defining a real one would require agreement with the team.

**No credentials are stored in this repository.** The environment contains only `baseUrl`. If an API key is ever added, it belongs in a `secret`-type variable and must not be committed.

## Author

Daniil, steepdan2003@gmail.com
