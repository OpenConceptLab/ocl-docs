# Subscriptions

A subscription keeps a copy of an OCL source up to date in another system, such as an OpenMRS concept dictionary. A client can:

* **Follow releases:** download each new released version of the source. This suits most systems: a released version has been published by the source's owner, and it never changes.
* **Follow every change:** also pick up edits made between releases. These come from the source's working copy (HEAD), which includes unreleased work.

Every request needs an API token, sent as `Authorization: Token <token>` (see the [API overview](../../overview.md)). Use an OCL account set up just for the subscription. Match concepts and mappings to your copy by their `url` (for example `/orgs/CIEL/sources/CIEL/concepts/1234/`), which is the same in exports and in the lists below. Many sources also fill in `external_id`, but it's optional and not guaranteed unique.

## Follow releases

1. Get the latest released version and note its `id` (for example `v2026-09-24`). If you already have that version, there's nothing new.
   ```
   GET /orgs/:org/sources/:source/latest/
   ```
2. Request that version's export:
   ```
   GET /orgs/:org/sources/:source/:version/export/
   ```
   * `302 Found`: the export is ready, and the `Location` header holds a signed download URL.
   * `208 Already Reported`: the export is still being created. Try again in a few minutes.
   * `204 No Content`: there is no export. Create one with a `POST` to the same URL, then request it again once it's ready.
3. Download the zip from the signed URL; most HTTP clients follow the redirect for you. It holds `export.json`: the version's details and every concept and mapping in it, retired ones included. The signed URL expires, so request the export again each time instead of saving the URL.

Check for a new release on a schedule; once a day is plenty. Don't ask a released version for changes with `updatedSince`: a released version never changes. The [Export API](../exportapi.md) has more on exports.

## Follow every change

Start with steps 1–3 above, then fetch everything that changed after the export:

1. Take the timestamp at the end of the export's filename and convert it to ISO 8601 (UTC). For example, `orgs_CIEL_sources_CIEL_v2026-09-24_2026-09-24_030336.zip` gives `2026-09-24T03:03:36Z`. The API names the file in the download's `Content-Disposition` header.
2. Note the current time in UTC, minus a few minutes to allow for clock differences. Then ask the source's concepts and mappings for everything updated since the timestamp:
   ```
   GET /orgs/:org/sources/:source/concepts/?updatedSince=2026-09-24T03:03:36Z&includeRetired=true&verbose=true&limit=100
   GET /orgs/:org/sources/:source/mappings/?updatedSince=2026-09-24T03:03:36Z&includeRetired=true&verbose=true&limit=100
   ```
   Follow each response's `next` header to the next page, until a response has none.
3. Save each record over your copy of it. A record with `"retired": true` has been retired, which is how OCL deletes concepts and mappings.

Once every page of both lists is saved, store the time you noted: the next sync starts from it. If a sync fails partway, keep the previous time and run it again. For each later sync, repeat steps 2 and 3.

About these requests:
* `includeRetired=true` is needed to see retirements. A concept or mapping that was hard-deleted is gone entirely, so it never appears here.
* `verbose=true` adds the names, descriptions and extras you need to update your copy.
* Each record is the full current version, not a list of what changed. The same record can come back twice: `updatedSince` includes records updated at that exact time, and edits made while you page can push a record onto the next page.
* Keep `limit` at 100 or less: OCL is lowering the maximum page size to 100. These lists ignore `offset`.
* An `updatedSince` value the API can't read is ignored, and the whole list comes back. If `num_found` on the first page is unexpectedly large, check the timestamp.

Don't use the single request to the source that earlier versions of this guide described, `GET /orgs/:org/sources/:source/?includeConcepts=true&includeMappings=true&updatedSince=…`: it returns `404 Not Found` when the source's own details haven't changed since `updatedSince`, and its records have no names or descriptions.

## Testing a client

Use a source you can edit, such as one in a test organization:

1. Create the source and add a few concepts and mappings.
2. Create a released version. OCL creates its export, and the export request returns `302` once it's ready.
   ```
   POST /orgs/:org/sources/:source/versions/
   {"id": "v1.0", "description": "Test release", "released": true}
   ```
3. Run the client's first sync. Its copy should match the source.
4. In the source, edit a concept, retire one and add one; edit a mapping and retire one.
5. Sync again. A client following every change should pick up exactly those edits. A client following releases should find nothing new.
6. Create and release `v1.1` the same way, then sync again. Now both kinds of client should have the edits.
