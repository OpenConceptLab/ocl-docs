# CodeSystem

## Introduction

The OCL FHIR service converts OCL's Source into FHIR's CodeSystem resource and provides ability to interact with OCL resources in FHIR format. 
The CodeSystem can be retrieved using two type of identifiers:
1. Using Global Namespace (canonical url)
2. Using Owner Namespace (Id)

Links:
* [FHIR CodeSystem spec](https://www.hl7.org/fhir/codesystem.html#resource)
* [FHIR CodeSystem $lookup spec](https://www.hl7.org/fhir/codesystem-operation-lookup.html)
* [FHIR CodeSystem $validate-code spec](https://www.hl7.org/fhir/codesystem-operation-validate-code.html)


## Get a single CodeSystem

The version-less request for the code system returns `most recent released version`. If none of the version is released then empty response will be returned.

#### Request url

`GET /fhir/CodeSystem/?url=:url`

`GET /orgs/:org/CodeSystem/:id`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
| url | The canonical url of the codesystem |
| org | The id of OCL organization          |
| id  | The id of OCL Source                |

#### Example Request

`GET /fhir/CodeSystem/?url=https://datim.org/CodeSystem/MER`

`GET /orgs/PEPFAR/CodeSystem/MER`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "e689d8a8-462a-426e-bb74-cb36c1be6938",
    "meta": {
        "lastUpdated": "2020-12-14T13:38:22.667-05:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/CodeSystem/?_format=json&url=https://datim.org/CodeSystem/MER"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER",
                "identifier": [
                    {
                        "use": "usual",
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER/version/v2.0/"
                    }
                ],
                "version": "v2.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "date": "2020-12-01T00:00:00-05:00",
                "publisher": "PEPFAR",
                "contact": [
                    {
                        "name": "Jon Doe 1",
                        "telecom": [
                            {
                                "system": "email",
                                "value": "jondoe1@gmail.com",
                                "use": "work",
                                "rank": 1,
                                "period": {
                                    "start": "2020-10-29T10:26:15-04:00",
                                    "end": "2025-10-29T10:26:15-04:00"
                                }
                            }
                        ]
                    }
                ],
                "jurisdiction": [
                    {
                        "coding": [
                            {
                                "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                                "code": "USA",
                                "display": "United States of America"
                            }
                        ]
                    }
                ],
                "purpose": "Test code system",
                "copyright": "This is the test source and copyright protected.",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ],
                "concept": [
                    {
                        "code": "A08J7tE7g4m",
                        "display": "TB_SCREEN (N, DSD, Age): PLHIV Screened",
                        "designation": [
                            {
                                "language": "en",
                                "use": {
                                    "code": "Short"
                                },
                                "value": "TB_SCREEN (N, DSD, Age)"
                            },
                            {
                                "language": "en",
                                "use": {
                                    "code": "Fully Specified"
                                },
                                "value": "TB_SCREEN (N, DSD, Age): PLHIV Screened"
                            },
                            {
                                "language": "en",
                                "use": {
                                    "code": "Code"
                                },
                                "value": "TB_SCREEN_N_DSD_Age"
                            }
                        ],
                        "property": [
                            {
                                "code": "conceptclass",
                                "valueString": "Data Element"
                            },
                            {
                                "code": "datatype",
                                "valueString": "Numeric"
                            },
                            {
                                "code": "inactive",
                                "valueBoolean": false
                            }
                        ]
                    }
                ]
            }
        }
    ]
}

```
</details>
<br />

By default, first `100` concepts are returned for a code system. If user wants to get more concepts, OCL FHIR service provides pagination support for a resource. The default page value is `page=1` and this number can be incremented to retrieve more concepts.


## Get a CodeSystem version
#### Request url

`GET /fhir/CodeSystem/?url=:url&version=:version`

`GET /orgs/:org/CodeSystem/:id/version/:version`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the codesystem|
|org | The id of OCL organization|
|id | The id of OCL Source|
|version | The version of code system|

#### Example Request

`GET /fhir/CodeSystem/?url=https://datim.org/CodeSystem/MER&version=v1.0`

`GET /orgs/:org/CodeSystem/MER/version/v1.0`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "e689d8a8-462a-426e-bb74-cb36c1be6938",
    "meta": {
        "lastUpdated": "2020-12-14T13:38:22.667-05:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/CodeSystem/?_format=json&url=https://datim.org/CodeSystem/MER"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER",
                "identifier": [
                    {
                        "use": "usual",
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER/version/v1.0/"
                    }
                ],
                "version": "v1.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "date": "2020-12-01T00:00:00-05:00",
                "publisher": "PEPFAR",
                "contact": [
                    {
                        "name": "Jon Doe 1",
                        "telecom": [
                            {
                                "system": "email",
                                "value": "jondoe1@gmail.com",
                                "use": "work",
                                "rank": 1,
                                "period": {
                                    "start": "2020-10-29T10:26:15-04:00",
                                    "end": "2025-10-29T10:26:15-04:00"
                                }
                            }
                        ]
                    }
                ],
                "jurisdiction": [
                    {
                        "coding": [
                            {
                                "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                                "code": "USA",
                                "display": "United States of America"
                            }
                        ]
                    }
                ],
                "purpose": "Test code system",
                "copyright": "This is the test source and copyright protected.",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ],
                "concept": [
                    {
                        "code": "A08J7tE7g4m",
                        "display": "TB_SCREEN (N, DSD, Age): PLHIV Screened",
                        "designation": [
                            {
                                "language": "en",
                                "use": {
                                    "code": "Short"
                                },
                                "value": "TB_SCREEN (N, DSD, Age)"
                            },
                            {
                                "language": "en",
                                "use": {
                                    "code": "Fully Specified"
                                },
                                "value": "TB_SCREEN (N, DSD, Age): PLHIV Screened"
                            },
                            {
                                "language": "en",
                                "use": {
                                    "code": "Code"
                                },
                                "value": "TB_SCREEN_N_DSD_Age"
                            }
                        ],
                        "property": [
                            {
                                "code": "conceptclass",
                                "valueString": "Data Element"
                            },
                            {
                                "code": "datatype",
                                "valueString": "Numeric"
                            },
                            {
                                "code": "inactive",
                                "valueBoolean": false
                            }
                        ]
                    }
                ]
            }
        }
    ]
}
```
</details>
<br />

## Get list of CodeSystem versions

This request returns all `released` versions for a given code system. Note that this request only returns code system definitions and does not populate concepts.
#### Request url

`GET /fhir/CodeSystem/?url=:url&version=*`

`GET /orgs/:org/CodeSystem/:id/version`

`GET /orgs/:org/CodeSystem/:id/version/*`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the codesystem |
|org | The id of OCL organization |
|id | The id of OCL Source |
|version | '*' value indicates all versions |

#### Example Request

`GET /fhir/CodeSystem/?url=https://datim.org/CodeSystem/MER&version=*`

`GET /orgs/PEPFAR/CodeSystem/MER/version`

`GET /orgs/PEPFAR/CodeSystem/MER/version/*`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "52939e86-ad18-4287-bf14-573a446adff9",
    "meta": {
        "lastUpdated": "2020-12-14T14:27:00.868-05:00"
    },
    "type": "searchset",
    "total": 2,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/CodeSystem/?_format=json&url=https%3A%2F%2Fwww.state.gov%2Fpepfar&version=*"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER",
                "identifier": [
                    {
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER/version/v2.0/"
                    }
                ],
                "version": "v2.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "date": "2020-12-01T00:00:00-05:00",
                "publisher": "PEPFAR",
                "contact": [
                    {
                        "name": "Jon Doe 1",
                        "telecom": [
                            {
                                "system": "email",
                                "value": "jondoe1@gmail.com",
                                "use": "work",
                                "rank": 1,
                                "period": {
                                    "start": "2020-10-29T10:26:15-04:00",
                                    "end": "2025-10-29T10:26:15-04:00"
                                }
                            }
                        ]
                    }
                ],
                "jurisdiction": [
                    {
                        "coding": [
                            {
                                "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                                "code": "USA",
                                "display": "United States of America"
                            }
                        ]
                    }
                ],
                "purpose": "Test code system",
                "copyright": "This is the test source and copyright protected.",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ]
            }
        },
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER",
                "identifier": [
                    {
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER/version/v1.0/"
                    }
                ],
                "version": "v1.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ]
            }
        }
    ]
}
```

</details>
<br />


## Get a list of codesystems

This request returns most recent released versions of all code systems.

#### Request url

`GET /fhir/CodeSystem/`

`GET /orgs/:org/CodeSystem/`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|org | The id of OCL organization |

#### Example Request

`GET /fhir/CodeSystem/`

`GET /orgs/PEPFAR/CodeSystem`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "c260abd6-4c35-4b1d-8f43-d269e1f6f543",
    "meta": {
        "lastUpdated": "2020-12-14T16:43:29.773-05:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/CodeSystem"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER1",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER1",
                "identifier": [
                    {
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER1/version/v1.0/"
                    }
                ],
                "version": "v1.0",
                "name": "MER1 Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "date": "2020-11-01T00:00:00-05:00",
                "publisher": "PEPFAR",
                "contact": [
                    {
                        "name": "Jon Doe 1",
                        "telecom": [
                            {
                                "system": "email",
                                "value": "jondoe1@gmail.com",
                                "use": "work",
                                "rank": 1,
                                "period": {
                                    "start": "2020-10-29T10:26:15-04:00",
                                    "end": "2025-10-29T10:26:15-04:00"
                                }
                            }
                        ]
                    }
                ],
                "jurisdiction": [
                    {
                        "coding": [
                            {
                                "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                                "code": "USA",
                                "display": "United States of America"
                            }
                        ]
                    }
                ],
                "purpose": "Test code system",
                "copyright": "This is the test source and copyright protected.",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ]
            }
        },
        {
            "resource": {
                "resourceType": "CodeSystem",
                "id": "MER2",
                "language": "en",
                "url": "https://datim.org/CodeSystem/MER2",
                "identifier": [
                    {
                        "type": {
                            "coding": [
                                {
                                    "system": "http://hl7.org/fhir/v2/0203",
                                    "code": "ACSN",
                                    "display": "Accession ID"
                                }
                            ],
                            "text": "Accession ID"
                        },
                        "system": "https://fhir.qa.aws.openconceptlab.org",
                        "value": "/orgs/PEPFAR-Test8/CodeSystem/MER2/version/v2.0/"
                    }
                ],
                "version": "v2.0",
                "name": "MER2 Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "date": "2020-12-01T00:00:00-05:00",
                "publisher": "PEPFAR",
                "contact": [
                    {
                        "name": "Jon Doe 2",
                        "telecom": [
                            {
                                "system": "email",
                                "value": "jondoe2@gmail.com",
                                "use": "work",
                                "rank": 1,
                                "period": {
                                    "start": "2020-10-29T10:26:15-04:00",
                                    "end": "2025-10-29T10:26:15-04:00"
                                }
                            }
                        ]
                    }
                ],
                "jurisdiction": [
                    {
                        "coding": [
                            {
                                "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                                "code": "USA",
                                "display": "United States of America"
                            }
                        ]
                    }
                ],
                "purpose": "Test code system",
                "copyright": "This is the test source and copyright protected.",
                "content": "example",
                "count": 20751,
                "property": [
                    {
                        "code": "conceptclass",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Classes/concepts",
                        "description": "Standard list of concept classes.",
                        "type": "string"
                    },
                    {
                        "code": "datatype",
                        "uri": "https://api.openconceptlab.org/orgs/OCL/sources/Datatypes/concepts",
                        "description": "Standard list of concept datatypes.",
                        "type": "string"
                    },
                    {
                        "code": "inactive",
                        "uri": "http://hl7.org/fhir/concept-properties",
                        "description": "True if the concept is not considered active.",
                        "type": "Coding"
                    }
                ]
            }
        }
    ]
}
```
</details>
<br />

## Create CodeSystem

The CodeSystem can be created in two ways either using global namespace or owner namespace. The server returns HTTP `201 Created` on succussful operation. 

<b>Create Accession identifier</b>

```
{
    "type": {
        "coding": [
            {
                "system": "http://hl7.org/fhir/v2/0203",
                "code": "ACSN",
                "display": "Accession ID"
            }
        ],
        "text": "Accession ID"
    },
    "system": "<HOSTED FHIR SERVER ADDRESS>",
    "value": "<RESOURCE URI>"
}

# Accepted values for system based on environment:
- https://fhir.aws.openconceptlab.org
- https://fhir.qa.aws.openconceptlab.org
- https://fhir.staging.aws.openconceptlab.org
- https://fhir.demo.aws.openconceptlab.org

```

**NOTE:**
1. The `CodeSystem.url` is mandatory field.
2. If version is not provided either in `accession identifier` or in `version` field, then CodeSystem of `default version 0.1` will be created.
3. The version value in `accession identifier` takes precedence in case version is provided in both `accession identifier` and `version` field.
4. If `CodeSystem.language` is empty then `en` languages is assumed.
5. If `CodeSystem.status` is empty then `draft` status is assumed.
6. In Global namespace, the CodeSystem.identifier (accession) is required and the CodeSystem.Id is ignored.
7. In Owner namespace, either CodeSystem.identifier (accession) or CodeSystem.Id is required. Both can not be empty.

#### Using global namespace

#### Request url

`POST /fhir/CodeSystem/`

<details>
<summary><b>Example request</summary>

```json
{
    "resourceType": "CodeSystem",
    "id": "Test1",
    "date": "2021-02-12",
    "language": "en",
    "url": "https://ocl.org/test1",
    "identifier": [
        {
            "use": "usual",
            "type": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/v2/0203",
                        "code": "ACSN",
                        "display": "Accession ID"
                    }
                ],
                "text": "Accession ID"
            },
            "system": "https://fhir.qa.aws.openconceptlab.org",
            "value": "/users/testuser/CodeSystem/Test1/"
        }
    ],
    "version": "2.0",
    "name": "Test Code System",
    "status": "draft",
    "contact": [
        {
            "name": "Jon Doe 1",
            "telecom": [
                {
                    "system": "email",
                    "value": "jondoe1@gmail.com",
                    "use": "work",
                    "rank": 1,
                    "period": {
                        "start": "2020-10-29T10:26:15-04:00",
                        "end": "2025-10-29T10:26:15-04:00"
                    }
                }
            ]
        }
    ],
    "jurisdiction": [
        {
            "coding": [
                {
                    "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                    "code": "USA",
                    "display": "United States of America"
                }
            ]
        }
    ],
    "content": "example",
    "concept": [
        {
            "code": "0ssVrvKblz1",
            "designation": [
                {
                    "language": "en",
                    "use": {
                        "code": "Fully Specified"
                    },
                    "value": "PMTCT_STAT (D, CS): New ANC clients"
                }
            ],
            "property": [
                {
                    "code": "conceptclass",
                    "valueString": "Data Element"
                },
                {
                    "code": "datatype",
                    "valueString": "Numeric"
                },
                {
                    "code": "inactive",
                    "valueBoolean": false
                }
            ]
        }
    ]
}

```
</details>
<br />

#### Using owner namespace

#### Request url

`POST /orgs/:org/CodeSystem/`

`POST /users/:user/CodeSystem/`

<details>
<summary><b>Example request</summary>

```json
{
    "resourceType": "CodeSystem",
    "id": "Test1",
    "date": "2021-02-12",
    "language": "en",
    "url": "https://ocl.org/test1",
    "version": "2.0",
    "name": "Test Code System",
    "status": "draft",
    "contact": [
        {
            "name": "Jon Doe 1",
            "telecom": [
                {
                    "system": "email",
                    "value": "jondoe1@gmail.com",
                    "use": "work",
                    "rank": 1,
                    "period": {
                        "start": "2020-10-29T10:26:15-04:00",
                        "end": "2025-10-29T10:26:15-04:00"
                    }
                }
            ]
        }
    ],
    "jurisdiction": [
        {
            "coding": [
                {
                    "system": "http://unstats.un.org/unsd/methods/m49/m49.htm",
                    "code": "USA",
                    "display": "United States of America"
                }
            ]
        }
    ],
    "content": "example",
    "concept": [
        {
            "code": "0ssVrvKblz1",
            "designation": [
                {
                    "language": "en",
                    "use": {
                        "code": "Fully Specified"
                    },
                    "value": "PMTCT_STAT (D, CS): New ANC clients"
                }
            ],
            "property": [
                {
                    "code": "conceptclass",
                    "valueString": "Data Element"
                },
                {
                    "code": "datatype",
                    "valueString": "Numeric"
                },
                {
                    "code": "inactive",
                    "valueBoolean": false
                }
            ]
        }
    ]
}

```
</details>
<br />

## Update CodeSystem
The CodeSystem can only be updated using Gloabl Namespace. The server returns HTTP `200 Ok` on succussful operation.

**NOTE:**
1. In order to add `new` concept in CodeSystem, user needs to populate the `CodeSystem.concept.*`. If the concept included is already exist in CodeSystem then it will be ignored. In order to update existing concept, user should delete the concept first and then add new concept with updated information.
2. The Both CodeSystem.id and CodeSystem.version is required to successfully update CodeSystem.
#### Request url

`PUT /orgs/:org/CodeSystem/:id/version/:version`

`PUT /users/:user/CodeSystem/:id/version/:version`

<details>
<summary><b>Example request</summary>

```json
{
    "resourceType": "CodeSystem",
    "language": "es",
    "name": "updated name",
    "status": "active",
    "content": "example",
    "concept": [
        {
            "code": "zuM0sUhtxTn",
            "display": "HTS_TST (N, DSD, IndexMod, Yield)",
            "designation": [
                {
                    "language": "en",
                    "use": {
                        "code": "Short"
                    },
                    "value": "HTS_TST (N, DSD, IndexMod, Yield)"
                },
                {
                    "language": "en",
                    "use": {
                        "code": "Fully Specified"
                    },
                    "value": "HTS_TST (N, DSD, IndexMod, Yield)"
                }
            ],
            "property": [
                {
                    "code": "conceptclass",
                    "valueString": "Indicator"
                },
                {
                    "code": "datatype",
                    "valueString": "Percentage"
                },
                {
                    "code": "inactive",
                    "valueBoolean": false
                }
            ]
        }
    ]
}


```
</details>
<br />

## Delete CodeSystem
The CodeSystem can only be deleted using Gloabl Namespace. The server returns HTTP `204 No Content` on succussful operation.

#### Request url

`DELETE /orgs/:org/CodeSystem/:id/version/:version`

`DELETE /users/:user/CodeSystem/:id/version/:version`

## Delete Concept from CodeSystem
The CodeSystem's concept can only be deleted using Global Namespace. The server returns HTTP `204 No Content` on succussful operation.

#### Request url

`DELETE /orgs/:org/CodeSystem/:id/version/:version/concepts/:concept-code`

`DELETE /users/:user/CodeSystem/:id/version/:version/concepts/:concept-code`

## FHIR Operations

As per mSVCM profile, following FHIR operations are supported for a code system:
1. $lookup
2. $validate-code

### $lookup

#### Request url

`GET /fhir/CodeSystem/$lookup/?system=:system&code=:code`

`GET /orgs/:org/CodeSystem/$lookup/?system=:system&code=:code`

`POST /fhir/CodeSystem/$lookup`

`POST /orgs/:org/CodeSystem/$lookup`

#### Request parameters (GET)

|  Parameter   |            Description     |
|-----|-------------------------------------|
|system | (M) The canonical url of the codesystem|
|code | (M) The concept code|
|version | (O) The version of the codesystem|
|displayLanguage | (O) The requested language for display|
|org | The id of OCL organization|

#### Request body (POST)

```
{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"system",
            "valueUri":"<system_url>"
        },
        {
            "name":"code",
            "valueCode":"<code>"
        },
        {
            "name":"version",
            "valueString":"<system_version>"
        },
        {
            "name":"displayLanguage",
            "valueCode":"<display_language>"
        }
    ]
}
```

#### Example request

`GET /fhir/CodeSystem/$lookup/?system=https://datim.org/CodeSystem/MER&code=vpvjaSZxlaA`

`GET /orgs/PEPFAR/CodeSystem/$lookup/?system=https://datim.org/CodeSystem/MER&code=vpvjaSZxlaA`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Parameters",
    "parameter": [
        {
            "name": "name",
            "valueString": "MER Source"
        },
        {
            "name": "version",
            "valueString": "v2.0"
        },
        {
            "name": "display",
            "valueString": "EA_HSS_NATIONAL_SUB_UNIT"
        },
        {
            "name": "designation",
            "part": [
                {
                    "name": "language",
                    "valueCode": "en"
                },
                {
                    "name": "use",
                    "valueCoding": {
                        "display": "Short"
                    }
                },
                {
                    "name": "value",
                    "valueString": "EA_HSS_NATIONAL_SUB_UNIT"
                }
            ]
        },
        {
            "name": "designation",
            "part": [
                {
                    "name": "language",
                    "valueCode": "en"
                },
                {
                    "name": "use",
                    "valueCoding": {
                        "display": "Code"
                    }
                },
                {
                    "name": "value",
                    "valueString": "EA_HSS_NATIONAL_SUB_UNIT"
                }
            ]
        },
        {
            "name": "designation",
            "part": [
                {
                    "name": "language",
                    "valueCode": "en"
                },
                {
                    "name": "use",
                    "valueCoding": {
                        "display": "Fully Specified"
                    }
                },
                {
                    "name": "value",
                    "valueString": "EA_HSS_NATIONAL_SUB_UNIT"
                }
            ]
        }
    ]
}
```
</details>
<br />

### $validate-code

### Request url

`GET /fhir/CodeSystem/$validate-code/?url=:url&code=:code`

`GET /orgs/:org/CodeSystem/$validate-code/?url=:url&code=:code`

`POST /fhir/CodeSystem/$validate-code`

`POST /orgs/:org/CodeSystem/$validate-code`

#### Request Parameters (GET)

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | (M) The canonical url of the codesystem|
|code | (M) The concept code|
|version | (O) The version of the codesystem|
|display | (O) The display associated with the code|
|displayLanguage | (O) Specifies the language to be used for description when validating the display property|
|coding | (O) A coding to validate (Alternate way to provide url, code, version and display) (only valid in POST request)|
|org | The id of OCL organization|

#### Request body (POST)

```
{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"url",
            "valueUri":"<system_url>"
        },
        {
            "name":"code",
            "valueCode":"<code>"
        },
        {
            "name":"version",
            "valueString":"<system_version>"
        },
        {
            "name":"display",
            "valueString":"<display>"
        },
        {
            "name":"displayLanguage",
            "valueCode":"<display_language>"
        }
    ]
}

{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"coding",
            "valueCoding": {
                "system" : "<system_url>",
                "code" : "<code>",
                "version": "<system_version>",
                "display":"<display>"
            }
        },
        {
            "name":"displayLanguage",
            "valueCode":"<display_language>"
        }
    ]
}

```

**NOTE:**
1. displayLanguage is ignored when display is not provided or empty
2. If coding is provided then system_url, code and system_version values are overridden with the values of
  coding.system, coding.code and coding.version respectively.

#### Example Request

`GET /fhir/CodeSystem/$validate-code/?url=https://datim.org/CodeSystem/MER&code=vpvjaSZxlaA`

`GET /orgs/:org/CodeSystem/$validate-code/?url=https://datim.org/CodeSystem/MER&code=vpvjaSZxlaA`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Parameters",
    "parameter": [
        {
            "name": "result",
            "valueBoolean": true
        }
    ]
}
```
</details>
<br />
<br />











