# Data integrity checks
## Overview

MongoDB does not guard references. One is able to delete a resource even though it may be referenced by another resource. It should not happen in normal operation, but may happen due to bugs in code or manual manipulation of data in MongoDB. To discover such issues as fast as possible a check has been added.

### Find Broken References

In order to find broken references send a POST request to `/manage/brokenreferences/` (superuser privileges required). You will trigger an asynchronous task and get back a json with uuid in the task field.

You can check, if the task has completed by sending a GET request to `/manage/brokenreferences/?task=task-uuid`. In response you will either get a state of the task or results.

Some broken references can be deleted automatically. It is considered safe to delete a broken reference if it is:
1. An element of a list e.g. ConceptVersion.source_version_ids
2. Not referenced by any other resource

Broken references, which can be deleted automatically are indicated in the returned result.

Before deletion always make sure to save aside the returned results in order to be able to revert mistakenly deleted resources. The full representation of a resource is always included under `items.broken_references.item`.

You can trigger deletion by sending a DELETE request to `/manage/brokenreferences/?task=task-uuid`.
