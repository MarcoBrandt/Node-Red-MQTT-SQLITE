# Modernized MQTT → SQLite flow (2026)

The 2020 flow is retained unchanged as `MQTT_Broker_Database_v5.json`. **This is an alternative v2 flow**, not a drop-in Dashboard 1.0 upgrade. It preserves the 16 HTTP endpoint paths while changing the database schema and selected response fields. Do **not** import both as active flows on the same Node-RED instance: their HTTP paths would conflict.

## Files
- `MQTT_SQLite_Modern_2026.json`: independent importable Node-RED flow
- `MQTT_Broker_Database_v5.json`: archived legacy flow (read-only reference)

## Prerequisites and setup
1. Back up your existing Node-RED flow and `bt_data.db` file before changes.
2. Install a currently supported Node-RED release with supported Node.js (consult https://nodered.org/docs/faq/node-versions).
3. Install `node-red-node-sqlite` from Manage Palette. Its `msg.topic` SQL node supports bound `msg.payload` arrays.
4. Import `MQTT_SQLite_Modern_2026.json` into a **separate/test Node-RED instance** first.
5. Configure the **Configure MQTT broker** node (default `localhost:1883`). Use authentication and MQTT over TLS where applicable; never commit credentials. Review topic `/esp32/#`.
6. Configure the SQLite configuration node's database path. Default is `bt_data_v2.db`. Use a writable dedicated location and verify the Node-RED service account's working directory and permissions.
7. Set the service timezone explicitly (e.g. `TZ=Europe/Helsinki`). `today` uses server-local midnight. `week` is a rolling 168-hour window.
8. Deploy and verify that the schema initializer ran; restart if the table was not initialized before the first MQTT message.
9. Keep `/data` and the Node-RED editor behind authentication and a trusted network. Node-RED `httpNodeAuth`, a properly configured reverse proxy, and TLS are appropriate controls.

## Improvements
- SQLite-bound parameters for MQTT input; no payload or topic string interpolation into SQL.
- ISO-8601 UTC timestamps plus numeric epoch milliseconds for correct time filtering, sorting and indexed queries.
- `CREATE TABLE IF NOT EXISTS` and a time index on deploy (no destructive `DROP TABLE` button).
- Input validation (nonempty, maximum 512-char ID and 256-char topic); fixed SQL allowlist and pagination capped at 1,000 records.
- Explicit JSON replies, error messages without leaking SQL details, and 16 HTTP paths retained.
- Independent of the deprecated `node-red-dashboard` and `node-red-ui-table` packages.

## Important differences
- Uses **new** `scans` table in a **new** `bt_data_v2.db` database. Old `BTdata` values are **not automatically migrated** and the old database is untouched.
- All timestamps are stored in UTC. For `/today`, the boundary uses the server timezone.
- The `unique` and `uniquedata` endpoints return grouped values with `sightings`, `first_seen`, `last_seen` rather than arbitrary grouped legacy rows.
- Endpoints returning rows accept `?limit=1..1000` (default 200). The count endpoints are unrestricted counts.
- Does not include a dashboard. To rebuild the UI, use maintained `@flowfuse/node-red-dashboard` and connect it to the new API, or build a dedicated UI. The old Angular-based dashboard is deprecated.
- This is not an authenticated API out of the box. Do not expose Node-RED or its endpoints directly to the Internet.

## Smoke tests (on a local test instance)
1. Deploy with MQTT broker and DB location configured; inspect Node-RED Debug for initialization errors.
2. Publish a string identifier to topic `/esp32/test`; verify it appears in `GET /data?limit=10`.
3. Publish an identifier containing a single quote, such as `O'Reilly`; confirm it is stored as data and cannot alter SQL.
4. Verify `GET /data/today/count`, `GET /data/week/unique/count`, and `GET /data/livefeed`.
5. Verify `GET /data?limit=1001` responds 400 and `GET /data?limit=10` responds 200.
6. Test a day/month rollover using a test clock; do not rely on the old text-based date ordering.

## Remaining work
- No live Node-RED service or MQTT broker was available for runtime verification. Test import, node package versions, TLS setup, and query behavior in your actual environment.
- Dashboard 2.0 must be created separately; old UI design was intentionally not ported.
- If existing data must be retained, build and validate an explicit `BTdata` to `scans` migration against a backup. Old day.month.year strings need careful parsing.
