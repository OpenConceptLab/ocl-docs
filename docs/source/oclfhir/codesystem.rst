Code System
============

Get
----
.. http:get:: /api/oclfhir/getall

    Retrieve details of a all fhir code systems.

    **Example request**:

    .. tabs::

        .. code-tab:: bash

            $ curl -H "Authorization: Token <token>" https://oclapi.org/api/oclfhir

        .. code-tab:: python

            import requests
            URL = 'https://readthedocs.org/api/v3/projects/pip/'
            TOKEN = '<token>'
            HEADERS = {'Authorization': f'token {TOKEN}'}
            response = requests.get(URL, headers=HEADERS)
            print(response.json())

    **Example response**:

    .. sourcecode:: json

        {
            "id": 12345,
            "name": "Pip"
        }

    :query string expand: allows to add/expand some extra fields in the response.
                          Allowed values are ``active_versions``, ``active_versions.last_build`` and
                          ``active_versions.last_build.config``. Multiple fields can be passed separated by commas.

Search
-------
Create
-------
Update 
-------
Delete
-------