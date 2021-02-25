# ConceptMap

## Introduction

The OCL FHIR service converts OCL's Source into FHIR's ConceptMap resource and provides ability to interact with OCL resources in FHIR format. 
The ConceptMap can be retrieved using two type of identifiers:
1. Using Global Namespace (canonical url)
2. Using Owner Namespace (Id)

Links:
* [FHIR ConceptMap spec](https://www.hl7.org/fhir/conceptmap.html#resource)
* [FHIR ConceptMap $translate spec](https://www.hl7.org/fhir/conceptmap-operation-translate.html)


## Get a single ConceptMap

The version-less request for the conceptmap returns `most recent released version`. If none of the version is released then empty response will be returned.

**NOTE**
- The ConceptMap.group.element.target.equivalence value is returned as a extension of target with extension url 
  `http://fhir.openconceptlab.org/ConceptMap/equivalence` .

#### Request url

`GET /fhir/ConceptMap/?url=:url`

`GET /orgs/:org/ConceptMap/:id`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
| url | The canonical url of the conceptmap |
| org | The id of the OCL organization      |
| id  | The id of the conceptmap            |

#### Example Request

`GET /fhir/ConceptMap/?url=https://datim.org/ConceptMap/MER`

`GET /orgs/PEPFAR-Test10/ConceptMap/MER`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "60b6b267-531e-4d9d-b857-77c43055c2f4",
    "meta": {
        "lastUpdated": "2021-02-25T18:28:04.514+00:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/"
        },
        {
            "relation": "prev",
            "url": "null"
        },
        {
            "relation": "next",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/?page=2"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER",
                "url": "https://datim.org/ConceptMap/MER",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER/version/v1.0/"
                },
                "version": "v1.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release",
                "group": [
                    {
                        "source": "/orgs/PEPFAR-Test10/sources/MER/",
                        "target": "/orgs/PEPFAR-Test10/sources/MER/",
                        "element": [
                            {
                                "code": "025M3T2Hsh2",
                                "target": [
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Has Option"
                                            }
                                        ],
                                        "code": "GNrMxECWqDp"
                                    },
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Has Option"
                                            }
                                        ],
                                        "code": "XqbMOMJhdoo"
                                    }
                                ]
                            },
                            {
                                "code": "0W99fNHD5Dz",
                                "target": [
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Derived From"
                                            }
                                        ],
                                        "code": "q4YD3KInPoX"
                                    },
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Derived From"
                                            }
                                        ],
                                        "code": "WPkzUQaVO7k"
                                    }
                                ]
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

By default, first `100` mappings are returned for a conceptmap. If user wants to get more mappings, OCL FHIR service provides pagination support for a resource. The default page value is `page=1` and this number can be incremented to retrieve more mappings.


## Get a ConceptMap version
#### Request url

`GET /fhir/ConceptMap/?url=:url&version=:version`

`GET /orgs/:org/ConceptMap/:id/version/:version`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the conceptmap|
|org | The id of the OCL organization|
|id | The id of the conceptmap |
|version | The version of the conceptmap|

#### Example Request

`GET /fhir/ConceptMap/?url=https://datim.org/ConceptMap/MER&version=v2.0`

`GET /orgs/:org/ConceptMap/MER/version/v2.0`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "60b6b267-531e-4d9d-b857-77c43055c2f4",
    "meta": {
        "lastUpdated": "2021-02-25T18:28:04.514+00:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/"
        },
        {
            "relation": "prev",
            "url": "null"
        },
        {
            "relation": "next",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/?page=2"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER",
                "url": "https://datim.org/ConceptMap/MER",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER/version/v1.0/"
                },
                "version": "v2.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release",
                "group": [
                    {
                        "source": "/orgs/PEPFAR-Test10/sources/MER/",
                        "target": "/orgs/PEPFAR-Test10/sources/MER/",
                        "element": [
                            {
                                "code": "025M3T2Hsh2",
                                "target": [
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Has Option"
                                            }
                                        ],
                                        "code": "GNrMxECWqDp"
                                    },
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Has Option"
                                            }
                                        ],
                                        "code": "XqbMOMJhdoo"
                                    }
                                ]
                            },
                            {
                                "code": "0W99fNHD5Dz",
                                "target": [
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Derived From"
                                            }
                                        ],
                                        "code": "q4YD3KInPoX"
                                    },
                                    {
                                        "extension": [
                                            {
                                                "url": "http://fhir.openconceptlab.org/ConceptMap/equivalence",
                                                "valueString": "Derived From"
                                            }
                                        ],
                                        "code": "WPkzUQaVO7k"
                                    }
                                ]
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

## Get list of ConceptMap versions

This request returns all `released` versions for a given conceptmap. Note that this request only returns conceptmap definitions and does not populate mappings.
#### Request url

`GET /fhir/ConceptMap/?url=:url&version=*`

`GET /orgs/:org/ConceptMap/:id/version`

`GET /orgs/:org/ConceptMap/:id/version/*`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the conceptmap |
|org | The id of the OCL organization |
|id | The id of the conceptmap |
|version | '*' value indicates all versions |

#### Example Request

`GET /fhir/ConceptMap/?url=https://datim.org/ConceptMap/MER&version=*`

`GET /orgs/PEPFAR-Test10/ConceptMap/MER/version`

`GET /orgs/PEPFAR-Test10/ConceptMap/MER/version/*`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "33e60b00-93c8-407a-bd35-2ce93924abb9",
    "meta": {
        "lastUpdated": "2021-02-25T18:40:40.704+00:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/version/"
        },
        {
            "relation": "prev",
            "url": "null"
        },
        {
            "relation": "next",
            "url": "null"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER",
                "url": "https://datim.org/ConceptMap/MER",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER/version/v1.0/"
                },
                "version": "v2.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release"
            }
        },
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER",
                "url": "https://datim.org/ConceptMap/MER",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER/version/v2.0/"
                },
                "version": "v2.0",
                "name": "MER Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release"
            }
        }
    ]
}
```

</details>
<br />


## Get a list of conceptmaps

This request returns list of most recent released versions of all conceptmaps.

#### Request url

`GET /fhir/ConceptMap/`

`GET /orgs/:org/ConceptMap/`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|org | The id of the OCL organization |

#### Example Request

`GET /fhir/ConceptMap/`

`GET /orgs/PEPFAR-Test10/ConceptMap/`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "33e60b00-93c8-407a-bd35-2ce93924abb9",
    "meta": {
        "lastUpdated": "2021-02-25T18:40:40.704+00:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://fhir.qa.aws.openconceptlab.org/orgs/PEPFAR-Test10/ConceptMap/MER/version/"
        },
        {
            "relation": "prev",
            "url": "null"
        },
        {
            "relation": "next",
            "url": "null"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER1",
                "url": "https://datim.org/ConceptMap/MER1",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER1/version/v1.0/"
                },
                "version": "v2.0",
                "name": "MER1 Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release"
            }
        },
        {
            "resource": {
                "resourceType": "ConceptMap",
                "id": "MER2",
                "url": "https://datim.org/ConceptMap/MER2",
                "identifier": {
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
                    "value": "/orgs/PEPFAR-Test10/ConceptMap/MER2/version/v2.0/"
                },
                "version": "v2.0",
                "name": "MER2 Source",
                "title": "DATIM Monitoring, Evaluation & Results Metadata",
                "status": "active",
                "description": "Auto-generated release"
            }
        }
    ]
}
```
</details>
<br />


## FHIR Operations

As per mSVCM profile, following FHIR operations are supported for a conceptmap:
1. $translate

### $translate

#### Request url

`GET /fhir/ConceptMap/$translate/?url=:url&system=:systam&code=:code`

`GET /orgs/:org/ConceptMap/$translate/?url=:url&system=:systam&code=:code`

`POST /fhir/ConceptMap/$translate`

`POST /orgs/:org/ConceptMap/$translate`

#### Request parameters (GET)

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url| (M) The canonical url of the conceptmap|
|system | (M) The canonical url of the codesystem|
|code | (M) The concept code that needs to be translated|
|conceptMapVersion| (O) The version of the conceptmap|
|version | (O) The version of the codesystem|
|targetSystem | (O) The canonical url of target codesystem|
|org | The id of OCL organization|

#### Request body (POST)

```
{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"url",
            "valueUri":"<conceptmap_url>"
        },
        {
            "name":"system",
            "valueUri":"<codesystem_url>"
        },
        {
            "name":"code",
            "valueCode":"<concept_code>"
        },
        {
            "name":"conceptMapVersion",
            "valueString":"<conceptmap_version>"
        },
        {
            "name":"version",
            "valueString":"<codesystem_version>"
        },
        {
            "name":"targetSystem",
            "valueCode":"<target_codesystem_url>"
        }
    ]
}

# Alternayively, user can include `coding` parameter to provide codesystem_url, concept_code and codesystem_version.

{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"url",
            "valueUri":"<conceptmap_url>"
        },
        {
            "name":"coding",
            "valueCoding": {
                "system" : "<codesystem_url>",
                "code" : "<concept_code>",
                "version": "<codesystem_version>"
            }
        }
    ]
}

```

**NOTE:**
- If coding is provided then codesystem_url, concept_code and codesystem_version values are overridden with the values of
  coding.system, coding.code and coding.version respectively.


#### Example Request

`GET /fhir/ConceptMap/$translate/?url=https://datim.org/CodeSystem/MER&system=/orgs/PEPFAR-Test10/sources/MER/&code=025M3T2Hsh2`

`GET /orgs/PEPFAR-Test10/ConceptMap/$translate/?url=https://datim.org/CodeSystem/MER&system=/orgs/PEPFAR-Test10/sources/MER/&code=025M3T2Hsh2`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Parameters",
    "parameter": [
        {
            "name": "result",
            "valueBoolean": true
        },
        {
            "name": "message",
            "valueString": "Matches found!"
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "EsEgz70ex5M"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "mN07ApGjAKh"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "aReRE4UUoKW"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "tb2OliToe2g"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "JqROtRoCBHP"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "GcAEOo6pgjG"
                    }
                }
            ]
        },
        {
            "name": "match",
            "part": [
                {
                    "name": "equivalence",
                    "valueString": "Has Option"
                },
                {
                    "name": "concept",
                    "valueCoding": {
                        "system": "/orgs/PEPFAR-Test10/sources/MER/",
                        "code": "BiJwnz9vw41"
                    }
                }
            ]
        }
    ]
}
```
</details>
<br />
<br />








