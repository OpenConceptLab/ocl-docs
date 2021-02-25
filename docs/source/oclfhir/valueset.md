# ValueSet

## Introduction

The OCL FHIR service converts OCL's Collection into FHIR's ValueSet resource and provides ability to interact with OCL resources in FHIR format. 
The ValueSet can be retrieved using two type of identifiers:
1. canonical url
2. Id

Links:
* [FHIR ValueSet spec](https://www.hl7.org/fhir/valueset.html#resource)
* [FHIR ValueSet $validate-code spec](https://www.hl7.org/fhir/valueset-operation-validate-code.html)
* [FHIR ValueSet $expand spec](https://www.hl7.org/fhir/valueset-operation-expand.html)

## Get a single ValueSet

The version-less request for the valueset returns `most recent released version`. If none of the version is released then empty response will be returned.

#### Request url

`GET /fhir/ValueSet/?url=:url`

`GET /orgs/:org/ValueSet/:id`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the valueset|
|org | The id of OCL organization|
|id | The id of OCL Collection|

#### Example Request

`GET /fhir/ValueSet/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19`

`GET /orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "62cde6c4-41e6-4d8a-9603-b320cde57940",
    "meta": {
        "lastUpdated": "2020-12-15T13:48:25.048-05:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/ValueSet/?_format=json&page=1&url=https%3A%2F%2Fwww.state.gov%2Fpepfar%2Fmer_reference_indicators_fy19"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY19",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy19",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY19",
                "title": "MER_REFERENCE_INDICATORS_FY19",
                "status": "active",
                "date": "2020-12-02T00:00:00-05:00",
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
                "immutable": true,
                "purpose": "Test value set",
                "copyright": "This is the test collection and copyright protected.",
                "compose": {
                    "include": [
                        {
                            "system": "https://www.state.gov/pepfar",
                            "version": "HEAD",
                            "concept": [
                                {
                                    "code": "AGYW_PREV",
                                    "display": "AGYW_PREV"
                                },
                                {
                                    "code": "CXCA_SCRN",
                                    "display": "CXCA_SCRN"
                                },
                                {
                                    "code": "CXCA_TX",
                                    "display": "CXCA_TX"
                                },
                                {
                                    "code": "DIAGNOSED_NAT",
                                    "display": "DIAGNOSED_NAT"
                                },
                                {
                                    "code": "EMR_SITE",
                                    "display": "EMR_SITE"
                                },
                                {
                                    "code": "FPINT_SITE",
                                    "display": "FPINT_SITE"
                                },
                                {
                                    "code": "GEND_GBV",
                                    "display": "GEND_GBV"
                                },
                                {
                                    "code": "HRH_CURR",
                                    "display": "HRH_CURR"
                                },
                                {
                                    "code": "HRH_PRE",
                                    "display": "HRH_PRE"
                                },
                                {
                                    "code": "HRH_STAFF_NAT",
                                    "display": "HRH_STAFF_NAT"
                                },
                                {
                                    "code": "HTS_INDEX",
                                    "display": "HTS_INDEX"
                                },
                                {
                                    "code": "HTS_RECENT",
                                    "display": "HTS_RECENT"
                                },
                                {
                                    "code": "HTS_SELF",
                                    "display": "HTS_SELF"
                                },
                                {
                                    "code": "HTS_TST",
                                    "display": "HTS_TST"
                                },
                                {
                                    "code": "KP_MAT",
                                    "display": "KP_MAT"
                                },
                                {
                                    "code": "KP_MAT_NAT",
                                    "display": "KP_MAT_NAT"
                                },
                                {
                                    "code": "KP_PREV",
                                    "display": "KP_PREV"
                                },
                                {
                                    "code": "LAB_PTCQI",
                                    "display": "LAB_PTCQI"
                                },
                                {
                                    "code": "OVC_HIVSTAT",
                                    "display": "OVC_HIVSTAT"
                                },
                                {
                                    "code": "OVC_SERV",
                                    "display": "OVC_SERV"
                                },
                                {
                                    "code": "PMTCT_ART",
                                    "display": "PMTCT_ART"
                                },
                                {
                                    "code": "PMTCT_ART_NAT",
                                    "display": "PMTCT_ART_NAT"
                                },
                                {
                                    "code": "PMTCT_EID",
                                    "display": "PMTCT_EID"
                                },
                                {
                                    "code": "PMTCT_FO",
                                    "display": "PMTCT_FO"
                                },
                                {
                                    "code": "PMTCT_HEI_POS",
                                    "display": "PMTCT_HEI_POS"
                                },
                                {
                                    "code": "PMTCT_STAT",
                                    "display": "PMTCT_STAT"
                                },
                                {
                                    "code": "PMTCT_STAT_NAT",
                                    "display": "PMTCT_STAT_NAT"
                                },
                                {
                                    "code": "PP_PREV",
                                    "display": "PP_PREV"
                                },
                                {
                                    "code": "PREP_NEW",
                                    "display": "PrEP_NEW"
                                },
                                {
                                    "code": "PrEP_CURR",
                                    "display": "PrEP_CURR"
                                },
                                {
                                    "code": "SC_STOCK",
                                    "display": "SC_STOCK"
                                },
                                {
                                    "code": "TB_ART",
                                    "display": "TB_ART"
                                },
                                {
                                    "code": "TB_PREV",
                                    "display": "TB_PREV"
                                },
                                {
                                    "code": "TB_STAT",
                                    "display": "TB_STAT"
                                },
                                {
                                    "code": "TX_CURR",
                                    "display": "TX_CURR"
                                },
                                {
                                    "code": "TX_CURR_NAT",
                                    "display": "TX_CURR_NAT"
                                },
                                {
                                    "code": "TX_ML",
                                    "display": "TX_ML"
                                },
                                {
                                    "code": "TX_NEW",
                                    "display": "TX_NEW"
                                },
                                {
                                    "code": "TX_PVLS",
                                    "display": "TX_PVLS"
                                },
                                {
                                    "code": "TX_TB",
                                    "display": "TX_TB"
                                },
                                {
                                    "code": "VL_SUPPRESSION_NAT",
                                    "display": "VL_SUPPRESSION_NAT"
                                },
                                {
                                    "code": "VMMC_CIRC",
                                    "display": "VMMC_CIRC"
                                },
                                {
                                    "code": "VMMC_CIRC_NAT",
                                    "display": "VMMC_CIRC_NAT"
                                },
                                {
                                    "code": "VMMC_TOTALCIRC_NAT",
                                    "display": "VMMC_TOTALCIRC_NAT"
                                }
                            ]
                        }
                    ]
                }
            }
        }
    ]
}
```
</details>
<br />

By default, first `100` concepts are returned for a value set. If user wants to get more concepts, OCL FHIR service provides pagination support for a resource. The default page value is `page=1` and this number can be incremented to retrieve more concepts.

## Get a ValueSet version

#### Request url

`GET /fhir/ValueSet/?url=:url&version=:version`

`GET /orgs/:org/ValueSet/:id/version/:version`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the valueset|
|org | The id of OCL organization|
|id | The id of OCL Collection|
|version | The version of valueset|

#### Example Request

`GET /fhir/ValueSet/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19&version=v11.0`

`GET /orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v11.0`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "010272a2-fdff-4b10-9152-3315f27d2eba",
    "meta": {
        "lastUpdated": "2020-12-15T15:17:43.245-05:00"
    },
    "type": "searchset",
    "total": 1,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/ValueSet/?_format=json&url=https%3A%2F%2Fwww.state.gov%2Fpepfar%2Fmer_reference_indicators_fy19&version=v11.0"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY19",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy19",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v11.0/"
                    }
                ],
                "version": "v11.0",
                "name": "MER_REFERENCE_INDICATORS_FY19",
                "title": "MER_REFERENCE_INDICATORS_FY19",
                "status": "active",
                "immutable": false,
                "compose": {
                    "include": [
                        {
                            "system": "https://www.state.gov/pepfar",
                            "version": "HEAD",
                            "concept": [
                                {
                                    "code": "AGYW_PREV",
                                    "display": "AGYW_PREV"
                                },
                                {
                                    "code": "CXCA_SCRN",
                                    "display": "CXCA_SCRN"
                                },
                                {
                                    "code": "CXCA_TX",
                                    "display": "CXCA_TX"
                                },
                                {
                                    "code": "DIAGNOSED_NAT",
                                    "display": "DIAGNOSED_NAT"
                                },
                                {
                                    "code": "EMR_SITE",
                                    "display": "EMR_SITE"
                                },
                                {
                                    "code": "FPINT_SITE",
                                    "display": "FPINT_SITE"
                                },
                                {
                                    "code": "GEND_GBV",
                                    "display": "GEND_GBV"
                                },
                                {
                                    "code": "HRH_CURR",
                                    "display": "HRH_CURR"
                                },
                                {
                                    "code": "HRH_PRE",
                                    "display": "HRH_PRE"
                                },
                                {
                                    "code": "HRH_STAFF_NAT",
                                    "display": "HRH_STAFF_NAT"
                                },
                                {
                                    "code": "HTS_INDEX",
                                    "display": "HTS_INDEX"
                                },
                                {
                                    "code": "HTS_RECENT",
                                    "display": "HTS_RECENT"
                                },
                                {
                                    "code": "HTS_SELF",
                                    "display": "HTS_SELF"
                                },
                                {
                                    "code": "HTS_TST",
                                    "display": "HTS_TST"
                                },
                                {
                                    "code": "KP_MAT",
                                    "display": "KP_MAT"
                                },
                                {
                                    "code": "KP_MAT_NAT",
                                    "display": "KP_MAT_NAT"
                                },
                                {
                                    "code": "KP_PREV",
                                    "display": "KP_PREV"
                                },
                                {
                                    "code": "LAB_PTCQI",
                                    "display": "LAB_PTCQI"
                                },
                                {
                                    "code": "OVC_HIVSTAT",
                                    "display": "OVC_HIVSTAT"
                                },
                                {
                                    "code": "OVC_SERV",
                                    "display": "OVC_SERV"
                                },
                                {
                                    "code": "PMTCT_ART",
                                    "display": "PMTCT_ART"
                                },
                                {
                                    "code": "PMTCT_ART_NAT",
                                    "display": "PMTCT_ART_NAT"
                                },
                                {
                                    "code": "PMTCT_EID",
                                    "display": "PMTCT_EID"
                                },
                                {
                                    "code": "PMTCT_FO",
                                    "display": "PMTCT_FO"
                                },
                                {
                                    "code": "PMTCT_HEI_POS",
                                    "display": "PMTCT_HEI_POS"
                                },
                                {
                                    "code": "PMTCT_STAT",
                                    "display": "PMTCT_STAT"
                                },
                                {
                                    "code": "PMTCT_STAT_NAT",
                                    "display": "PMTCT_STAT_NAT"
                                },
                                {
                                    "code": "PP_PREV",
                                    "display": "PP_PREV"
                                },
                                {
                                    "code": "PREP_NEW",
                                    "display": "PrEP_NEW"
                                },
                                {
                                    "code": "PrEP_CURR",
                                    "display": "PrEP_CURR"
                                },
                                {
                                    "code": "SC_STOCK",
                                    "display": "SC_STOCK"
                                },
                                {
                                    "code": "TB_ART",
                                    "display": "TB_ART"
                                },
                                {
                                    "code": "TB_PREV",
                                    "display": "TB_PREV"
                                },
                                {
                                    "code": "TB_STAT",
                                    "display": "TB_STAT"
                                },
                                {
                                    "code": "TX_CURR",
                                    "display": "TX_CURR"
                                },
                                {
                                    "code": "TX_CURR_NAT",
                                    "display": "TX_CURR_NAT"
                                },
                                {
                                    "code": "TX_ML",
                                    "display": "TX_ML"
                                },
                                {
                                    "code": "TX_NEW",
                                    "display": "TX_NEW"
                                },
                                {
                                    "code": "TX_PVLS",
                                    "display": "TX_PVLS"
                                },
                                {
                                    "code": "TX_TB",
                                    "display": "TX_TB"
                                },
                                {
                                    "code": "VL_SUPPRESSION_NAT",
                                    "display": "VL_SUPPRESSION_NAT"
                                },
                                {
                                    "code": "VMMC_CIRC",
                                    "display": "VMMC_CIRC"
                                },
                                {
                                    "code": "VMMC_CIRC_NAT",
                                    "display": "VMMC_CIRC_NAT"
                                },
                                {
                                    "code": "VMMC_TOTALCIRC_NAT",
                                    "display": "VMMC_TOTALCIRC_NAT"
                                }
                            ]
                        }
                    ]
                }
            }
        }
    ]
}
```
</details>
<br />


## Get list of ValueSet versions

This request returns all `released` versions for a given valueset. Note that this request only returns valueset definitions and does not populate concepts.

#### Request url

`GET /fhir/ValueSet/?url=:url&version=*`

`GET /orgs/:org/ValueSet/:id/version`

`GET /orgs/:org/ValueSet/:id/version/*`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | The canonical url of the valueset|
|org | The id of OCL organization|
|id | The id of OCL Collection|
|version | '*' value indicates all versions|

#### Example Request

`GET /fhir/ValueSet/?url=https://www.state.gov/pepfar&version=*`

`GET /orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version`

`GET /orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/*`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "f38f18ce-d24c-49fe-8184-7e27e9a2fec8",
    "meta": {
        "lastUpdated": "2020-12-15T15:24:08.502-05:00"
    },
    "type": "searchset",
    "total": 2,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/ValueSet/?_format=json&url=https%3A%2F%2Fwww.state.gov%2Fpepfar%2Fmer_reference_indicators_fy19&version=*"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY19",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy19",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v11.0/"
                    }
                ],
                "version": "v11.0",
                "name": "MER_REFERENCE_INDICATORS_FY19",
                "title": "MER_REFERENCE_INDICATORS_FY19",
                "status": "active",
                "immutable": false
            }
        },
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY19",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy19",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY19",
                "title": "MER_REFERENCE_INDICATORS_FY19",
                "status": "active",
                "date": "2020-12-02T00:00:00-05:00",
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
                "immutable": true,
                "purpose": "Test value set",
                "copyright": "This is the test collection and copyright protected."
            }
        }
    ]
}
```

</details>
<br />

## Get a list of valuesets

This request returns most recent released versions of all valuesets.

#### Request url

`GET /fhir/ValueSet/`

`GET /orgs/:org/ValueSet/`

#### Request Parameters

|  Parameter   |            Description     |
|-----|-------------------------------------|
|org | The id of OCL organization|

#### Example Request

`GET /fhir/ValueSet/`

`GET /orgs/PEPFAR-Test8/ValueSet`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "Bundle",
    "id": "a856c93f-f0f3-4384-ac18-45fc30ee8fd5",
    "meta": {
        "lastUpdated": "2020-12-15T15:29:25.095-05:00"
    },
    "type": "searchset",
    "total": 4,
    "link": [
        {
            "relation": "self",
            "url": "http://localhost:8080/fhir/ValueSet/?_format=json"
        }
    ],
    "entry": [
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY17",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy17",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY17/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY17",
                "title": "MER_REFERENCE_INDICATORS_FY17",
                "status": "active",
                "immutable": false
            }
        },
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY18",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy18",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY18/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY18",
                "title": "MER_REFERENCE_INDICATORS_FY18",
                "status": "active",
                "immutable": false
            }
        },
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY19",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy19",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY19/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY19",
                "title": "MER_REFERENCE_INDICATORS_FY19",
                "status": "active",
                "date": "2020-12-02T00:00:00-05:00",
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
                "immutable": true,
                "purpose": "Test value set",
                "copyright": "This is the test collection and copyright protected."
            }
        },
        {
            "resource": {
                "resourceType": "ValueSet",
                "id": "MER_REFERENCE_INDICATORS_FY20",
                "url": "https://www.state.gov/pepfar/mer_reference_indicators_fy20",
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
                        "value": "/orgs/PEPFAR-Test8/ValueSet/MER_REFERENCE_INDICATORS_FY20/version/v12.0/"
                    }
                ],
                "version": "v12.0",
                "name": "MER_REFERENCE_INDICATORS_FY20",
                "title": "MER_REFERENCE_INDICATORS_FY20",
                "status": "active",
                "immutable": false
            }
        }
    ]
}
```

</details>
<br />

## Create ValueSet

The ValueSet can be created in two ways either using global namespace or owner namespace. The server returns HTTP `201 Created` on succussful operation. 

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
1. The `ValueSet.url` is mandatory field.
2. If version is not provided either in `accession identifier` or in `version` field, then ValueSet of `default version 0.1` will be created.
3. The version value in `accession identifier` takes precedence in case version is provided in both `accession identifier` and `version` field.
4. If `ValueSet.language` is empty then `en` languages is assumed.
5. If `ValueSet.status` is empty then `draft` status is assumed.
6. In Global namespace, the ValueSet.identifier (accession) is required and the ValueSet.Id is ignored.
7. In Owner namespace, either ValueSet.identifier (accession) or ValueSet.Id is required. Both can not be empty.
8. The ValueSet.compose.include.system is mandatory field.
9. All of the CodeSystems included in ValueSet.compose.include.system should be canonical url of CodeSystem and known to OCL.
10. The concept codes provided in ValueSet.compose.include.concept.code should exist in respective CodeSystem and only valid codes will be added in ValueSet.

#### Using global namespace

#### Request url

`POST /fhir/ValueSet/`

<details>
<summary><b>Example request</summary>

```json
{
    "resourceType": "ValueSet",
    "id": "Test1",
    "url": "https://ocl.org/ValueSet/test1",
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
            "value": "/users/testuser/ValueSet/Test1/"
        }
    ],
    "version": "v5.0",
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
    "compose": {
        "include": [
            {
                "system": "https://ocl.org/CodeSystem/test1",
                "version": "v2.0",
                "concept": [
                    {
                        "code": "AGYW_PREV"
                    },
                    {
                        "code": "CXCA_SCRN"
                    }
                ]
            }
        ]
    }
}

```
</details>
<br />

#### Using owner namespace

#### Request url

`POST /orgs/:org/ValueSet/`

`POST /users/:user/ValueSet/`

<details>
<summary><b>Example request</summary>

```json
{
    "resourceType": "ValueSet",
    "id": "Test1",
    "url": "https://ocl.org/ValueSet/test1",
    "version": "v5.0",
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
    "compose": {
        "include": [
            {
                "system": "https://ocl.org/CodeSystem/test1",
                "version": "v2.0",
                "concept": [
                    {
                        "code": "AGYW_PREV"
                    },
                    {
                        "code": "CXCA_SCRN"
                    }
                ]
            }
        ]
    }
}

```
</details>
<br />

## Delete ValueSet
The ValueSet can only be deleted using Gloabl Namespace. The server returns HTTP `204 No Content` on succussful operation.

#### Request url

`DELETE /orgs/:org/ValueSet/:id/version/:version`

`DELETE /users/:user/ValueSet/:id/version/:version`

## FHIR Operations

As per mSVCM profile, following FHIR operations are supported for a valueset:
1. $validate-code
2. $expand

### $validate-code

### Request url

`GET /fhir/ValueSet/$validate-code/?url=:url&system=:system&code=:code`

`GET /orgs/:org/ValueSet/$validate-code/?url=:url&system=:system&code=:code`

`POST /fhir/ValueSet/$validate-code`

`POST /orgs/:org/ValueSet/$validate-code`

#### Request Parameters (GET)

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | (M) The canonical url of the valueset|
|code | (M) The concept code, the code that is to be validated|
|system | (M) The system canonical url for the code that is to be validated|
|valueSetVersion | (O) The version of the valueset|
|systemVersion | (O) The version of the codesystem|
|display | (O) The display associated with the code|
|displayLanguage | (O) Specifies the language to be used for description when validating the display property|
|coding | (O) A coding to validate (Alternate way to provide system, code, systemVersion and display) (only valid in POST request)|
|org | The id of OCL organization|


#### Request body (POST)

```
{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"url",
            "valueUri":"<valueset_url>"
        },
        {
            "name":"code",
            "valueCode":"<code>"
        },
        {
            "name":"system",
            "valueUri":"<system_url>"
        },
        {
            "name":"valueSetVersion",
            "valueString":"<valueset_version>"
        },
        {
            "name":"systemVersion",
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
            "name":"url",
            "valueUri":"<valueset_url>"
        },
        {
            "name":"valueSetVersion",
            "valueString":"<valueset_version>"
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
2. If coding is provided then system_url, code, system_version and display values are overridden with the values of
  coding.system, coding.code, coding.version and coding.display respectively.


### Example Request

`GET /fhir/ValueSet/$validate-code/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19&system=https://www.state.gov/pepfar&code=DIAGNOSED_NAT `

`GET /orgs/:org/ValueSet/$validate-code/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19&system=https://www.state.gov/pepfar&code=DIAGNOSED_NAT `


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

### $expand

### Request url

`GET /fhir/ValueSet/$expand/?url=:url`

`GET /orgs/:org/ValueSet/$expand/?url=:url`

`POST /fhir/ValueSet/$expand`

`POST /orgs/:org/ValueSet/$expand`

#### Request Parameters (GET)

|  Parameter   |            Description     |
|-----|-------------------------------------|
|url | (M) The canonical url of the valueset|
|valueSetVersion | (O) The version of the valueset|
|offset | (O) Starting point if subset is desired (default 0)|
|count | (O) Desired number of codes to be returned (default 100)|
|includeDesignations | (O) Controls whether concept designations are to be included or excluded in value set expansions (default true)|
|includeDefinition | (O) Controls whether the value set definition is included or excluded in value set expansions (default false)|
|activeOnly | (O) Controls whether inactive(retired) concepts are included or excluded in value set expansions (default true)|
|displayLanguage | (O) The language to be used for ValueSet.expansion.contains.display|
|exclude-system | (O) Code system, or a particular version of a code system to be excluded from the value set expansion, example - http://loinc.org\|2.56 |
|system-version | (O) Specifies a version to use for a system, if the value set does not specify which one to use, example - http://loinc.org\|2.56 |
|filter | (O) The <b>case-sensitive</b> code filter to be used to control codes included in valueSet expansion. If multiple filters are needed then each code filter should be separated by <b>double underscore "\_\_"</b>, for example - <b>EMR\_\_HRP\_\_KP (EMR or HRP or KP)</b>. If the filter itself includes "\_", then the filter should be surrounded in double quotes. For example, if user wants to filter on "HRH_" then the multi filter string should be <b>EMR\_\_"HRH_"\_\_KP (EMR or HRH_ or KP) </b>. |
|org | The id of OCL organization |


#### Request body (POST)

```
{
    "resourceType":"Parameters",
    "parameter": [
        {
            "name":"url",
            "valueUri":"<valueset_url>"
        },
        {
            "name":"valueSetVersion",
            "valueString":"<valueset_version>"
        },
        {
            "name":"offset",
            "valueInteger":0
        },
        {
            "name":"count",
            "valueInteger":100
        },
        {
            "name":"includeDesignations",
            "valueBoolean": false
        },
        {
            "name":"includeDefinition",
            "valueBoolean": false
        },
        {
            "name":"activeOnly",
            "valueBoolean": false
        },
        {
            "name":"displayLanguage",
            "valueCode":"<display_language>"
        },
        {
            "name":"exclude-system",
            "valueCanonical":"<system_canonical_url>"
        },
        {
            "name":"system-version",
            "valueCanonical":"<system_canonical_url>"
        }
    ]
}
```

#### Example Request

`GET /fhir/ValueSet/$expand/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19`

`GET /orgs/PEPFAR-Test8/ValueSet/$expand/?url=https://www.state.gov/pepfar/mer_reference_indicators_fy19`

<details>
<summary><b>Example response</summary>

```json
{
    "resourceType": "ValueSet",
    "status": "active",
    "compose": {
        "include": [
            {
                "valueSet": [
                    "https://www.state.gov/pepfar/mer_reference_indicators_fy19|v12.0"
                ]
            }
        ]
    },
    "expansion": {
        "identifier": "834041db-d0a9-47ef-9cd2-bf92dcc18992",
        "timestamp": "2020-12-15T17:15:08-05:00",
        "total": 44,
        "offset": 0,
        "parameter": [
            {
                "name": "url",
                "valueUri": "https://www.state.gov/pepfar/mer_reference_indicators_fy19"
            },
            {
                "name": "valueSetVersion",
                "valueString": "v12.0"
            },
            {
                "name": "offset",
                "valueInteger": 0
            },
            {
                "name": "count",
                "valueInteger": 100
            },
            {
                "name": "includeDesignations",
                "valueBoolean": true
            },
            {
                "name": "includeDefinition",
                "valueBoolean": false
            },
            {
                "name": "activeOnly",
                "valueBoolean": true
            }
        ],
        "contains": [
            {
                "system": "https://www.state.gov/pepfar",
                "inactive": false,
                "version": "HEAD",
                "code": "AGYW_PREV",
                "display": "AGYW_PREV",
                "designation": [
                    {
                        "language": "en",
                        "use": {
                            "code": "Fully Specified"
                        },
                        "value": "AGYW_PREV"
                    }
                ]
            },
            {
                "system": "https://www.state.gov/pepfar",
                "inactive": false,
                "version": "HEAD",
                "code": "CXCA_SCRN",
                "display": "CXCA_SCRN",
                "designation": [
                    {
                        "language": "en",
                        "use": {
                            "code": "Fully Specified"
                        },
                        "value": "CXCA_SCRN"
                    }
                ]
            }
        ]
    }
}
```
</details>
<br />
<br />




