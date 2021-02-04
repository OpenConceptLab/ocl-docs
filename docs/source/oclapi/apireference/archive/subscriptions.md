# Subscriptions
## Overview
A client system may subscribe to the `concepts` and `mappings` in a source by using a combination of the [[Export API]] and the [[sources]] endpoint. The high-level subscription process is as follows:

1. **SUBSCRIPTION CLIENT CONFIGURATION:** Configure OCL subscription module to subscribe to a specific source with the correct authentication token and to optionally automatic retrieve future updates on a schedule
2. **INITIAL SYNCHRONIZATION:** Perform initial synchronization, which consists of (1) downloading and importing the full export file (containing concepts and mappings) of the "latest" released source version; and (2) fetching updates to the dictionary that were made after the export timestamp
3. **SUBSEQUENT SYNCHRONIZATIONS:** Requests additional updates (includes Creates, Updates and Deletions) to the source manually or automatically based on a schedule (e.g. weekly/monthly) that occurred after the most recent prior synchronization. Import the changes into the OpenMRS concept dictionary.


The recommended approach is to create an OCL user that is only used for subscriptions. All requests must use the API token displayed on the OCL user profile page.

An `external_id` field is supported in most resources to allow correct matching of resources between OCL and client systems. The following resources include the `external_id` field: `sources`, `concepts`, `mappings`, `concept names`, and `concept descriptions`.



## Subscription Process
_For the first synchronization, fetch the most recent export of a **released** source version_
* Fetch the latest released version of the source -- record the source version "id" field
```
GET /orgs/[:org]/sources/[:source]/latest/
```
* Retrieve the filename of full export for the latest released source version, which is returned in the response header as the `exportURL` field. Note that if the export file does not already exist, is still being processed, or is invalid, the API returns a specific status or error code and the export can be retrieved later after an appropriate interval. Refer to [[Export-API]] for more info on this request.
```
GET /orgs/[:org]/sources/[:source]/[:sourceVersion]/export/
```
* Download the export file from the `exportURL` (returns a compressed tar file of the JSON). Record the `lastUpdated` timestamp which is part of the filename. Refer to [[Export-API]] for more info on this request. Note that the `exportURL` is designed to be used only once and it expires after a set time limit -- to ensure that the `exportURL` works correctly, use it immediately and request another `exportURL` each time you want to download an export file.
* Fetch all updates to the source that are more recent than the export using the `lastUpdated` timestamp (e.g. ISO 8601 timestamp (e.g. `2011-11-16T14:26:15Z`)). There are two methods for doing this:
    * Use a combination of the `limit` and `offset` parameters to iteratively retrieve results until fewer than `limit` results are returned. Note that the standard paging response headers (num_returned, num_found, offset, page, next_url, prev_url) are not supported at the source endpoint, and paging must be managed by the client.
```
GET /orgs/[:org]/sources/[:source]/?includeConcepts=true&includeMappings=true&includeRetired=true&limit=500&offset=500&updatedSince=[:lastUpdated]
```
    * Use the `Compress: true` request header to retrieve the results as a compressed file -- this will disable paging (the `limit` and `offset` parameters will have no effect) and return all the concept/mapping results. Note that this will result in a server timeout / 404 if there are too many results. In practice, if you expect more than 1,000 results, it is not recommended to use this approach at this time.
```
GET /orgs/[:org]/sources/[:source]/?includeConcepts=true&includeMappings=true&includeRetired=true&updatedSince=[:lastUpdated]
```

_For subsequent synchronizations, fetch records that have been updated since the `lastUpdated` timestamp_
* Fetch all updates to the source that are more recent than `lastUpdated` timestamp (e.g. ISO 8601 timestamp (e.g. `2011-11-16T14:26:15Z`)) from the previous synchronization. See above for the two methods of retrieving updated records at the source endpoint. Record the current timestamp to be used in subsequent requests.
```
GET /orgs/[:org]/sources/[:source]/?includeMappings=true&includeConcepts=true&includeRetired=true&limit=0&updatedSince=[lastUpdated]
```



## Example
* A subscription client, such as the OpenMRS OCL Subscription module, should be configured with the source URL (e.g. `/orgs/CIEL/sources/CIEL/), an API token, and a synchronization schedule (e.g. weekly or monthly)
* On the first synchronization:
    * Subscription client will request details on the most recent released source version: `GET /orgs/CIEL/sources/CIEL/latest/`
    * Subscription client will request the export file URL of the most recent released source version, which will return the `exportURL` in the response header: `GET /orgs/CIEL/sources/CIEL/[:sourceVersion]/export/`
        * Note that if the export file is not ready, the subscription client will need to request the export file again after an appropriate interval
    * Subscription client will download the export file from `exportURL`, which is returned as a compressed tar of the JSON results -- the `exportURL` can be discarded, as it is re-generated for each request
    * Subscription client will decompress the file and process the results
    * Subscription client may then request any changes to the source that occurred after the `lastUpdated` timestamp of the export file: `GET /orgs/CIEL/sources/CIEL/?includeConcepts=true&includeMappings=true&includeRetired=true&updatedSince=[:lastUpdated]` -- these results should be processed in the same manner as above; the `lastUpdated` date should be stored for subsequent synchronizations
* On subsequent synchronizations:
    * Later, subscription client may request any changes to the source that occurred after the `lastUpdated` timestamp of the previous synchronization: `GET /orgs/CIEL/sources/CIEL/?includeConcepts=true&includeMappings=true&includeRetired=true&updatedSince=[:lastUpdated]` -- these results should be processed in the same manner as above; the `lastUpdated` date should be stored for subsequent synchronizations



## Subscription Client Testing Process

### Overview
This page outlines a step-by-step process for testing all key functionality of OCL from the point of view of a **Subscription Client** -- that is, any external software/module that will connect to OCL via the API to fetch exports and source updates, and perform basic content updates. Note that there may be additional functionality performed by the subscription client, such as an import or logging, that should be tested in addition to the steps outlined below.

#### A note on Source Versions
Also note about source versions:
Unfortunately source versions were not implemented to spec (meaning, they work differently than Github tags) -- so they need a bit of explanation.

* First, the most recent source version is a container of all edits that you make until you create a new source version. Originally, source versions were supposed to be a frozen snapshot in time (like Github tags), but there was something lost in translation with the developer. Meaning, that to create a snapshot that does not change, you must create a new source version (let's call it v9) and then create an additional version (v9-head) which will effectively freeze v9 and cause v9-head to contain all subsequent edits until a new source version is created.
* Second, only one source version can be "released" per source (this will be changed in the future).
* Third, source versions should always be created with previous_version ID set to the most recent source version and with released = false -- not doing so can corrupt the entire source.

### Process
<ol>

<li>Manually create user account on OCL to get an API token
<ul>
<li>From OCL website, sign up for new OCL user account</li>
<li>From your email account, verify your new account</li>
<li>Sign-in to OCL using your new account and browse to your user profile (link is at the top-right)</li>
<li>Record your user's API token from your user profile page</li>
</ul>
</li>


<li>Create a new organization and source<ul>

<li>Create new org (e.g. "MyOrg") using the "+" dropdown at top-right -- or using the API:
<pre>POST /orgs/</pre>
<pre>
{
    "id": "MyOrg",
    "full_name": "My Organization"
}
</pre></li>

<li>Create new source (e.g. "MySource") using the "Add Source" link on your organization's page -- or from API:
<pre>
POST /orgs/MyOrg/sources/
</pre>
<pre>
{
    "id": "MySource",
    "full_name": "My Source"
}
</pre>
</li>
</ul></li>


<li>Populate your new source with some starter content<ul>

<li>Add minimum of 3 concepts to test the different types of concept edits (New, Update, Delete/Retire, No Change) -- can do this on the OCL website using the "Add Concept" link on the source page, or via the API:
<pre>
POST /orgs/MyOrg/source/MySource/concepts/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/concepts/
</pre>
<pre>
{
}
</pre><pre>
POST /orgs/MyOrg/source/MySource/concepts/
</pre>
<pre>
{
}
</pre>
</li>

<li>Add minimum of 6 mappings, 1 internal and 1 external mapping per concept, to test the different types of mapping edits (Create, Update, Delete/Retire, No Change) for both internal and external mappings -- can do this on the OCL website using the "Add Mappings" link on individual concept pages, or via the API:
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
<pre>
POST /orgs/MyOrg/source/MySource/mappings/
</pre>
<pre>
{
}
</pre>
</li>
</ul></li>


<li>Create new source versions to save the current state of the source and prepare for export<ul>
<li>Fetch the automatically created source version and record ID
<pre>
GET /orgs/MyOrg/sources/MySource/versions/
</pre>
<pre>
{
    # source version output goes here
}
</pre>
</li>
<li>Create first new source version setting `previous_version` to `id` from above
<pre>
POST /orgs/CIEL/sources/CIEL/versions/
</pre>
<pre>
{
    "id":"v1.0",
    "previous_version": "5582be2550d61b5538ed694c",
    "description": "CIEL Dictionary release 2015-05-14",
    "released": false
}
</pre></li>

<li>Verify that source version `v1.0` has finished processing -- `_ocl_processing` should be `false` or null/not returned
<pre>
GET /orgs/CIEL/sources/CIEL/v1.0/
</pre>
<pre>
{
    "id":"v1.0",
    ...
    "_ocl_processing": false
}
</pre>
</li>

<li>After `v1.0` has finished processing, create a second new source version called `v1.1` setting `previous_version` to the `id` from the one you just created - this effectively freezes `v1.0` and all new changes will be saved to `v1.1`
<pre>
POST /orgs/CIEL/sources/CIEL/versions/
</pre>
<pre>
{
    "id":"v1.1",
    "previous_version": "v1.0",
    "description": "My Source v1.1 -- under development",
    "released": false
}
</pre>
</li>

<li>Now release `v1.0`
<pre>
POST /orgs/CIEL/sources/CIEL/v1.0/
</pre>
<pre>
{
    "released": true
}
</pre>
</li>
</ul></li>



<li>Cache export of the newly released export version<ul>
<li>Create and cache the export of source version `v1.0`
<pre>
POST /orgs/MyOrg/sources/MySource/v1.0/export/
</pre>
</li>
</ul></li>

<li>Make changes to concepts and mappings - these will be saved to source version `v1.1`</li>

<li>Fetch the export and changes made since the export<ul>
<li>Fetch the export - the request returns the URL to download the export file in the header attribute `exportUrl`
<pre>
GET /orgs/MyOrg/sources/MySource/v1.0/export/
</pre>
</li>
<li>Fetch changes to source since the export using the lastUpdated timestamp from the export file as an ISODate
<pre>
GET /orgs/MyOrg/sources/MySource/?includeConcepts=true&includeMappings=true&updatedSince=[:LastUpdatedTimestamp-as-ISODate]
</pre>
</li>
</ul></li>

<li>Test the subscription client configuration and initial synchronization<ul>
<li>Subscription Client Configuration</li>
<li>Initial Synchronization</li>
</ul></li>

<li>Make additional updates to concepts and mappings to test subsequent synchronization of the subscription module<ul>
<li>Make some concept changes</li>
<li>Make some mapping changes</li>
</ul></li>

<li>Test the subscription client subsequent synchronization<ul>
<li>Perform subsequent synchronization</li>
</ul></li>

</ol>
