# Operation: $clone Documentation


## Overview


### **What is cloning?**



* The OCL API exposes a `$clone` operation to recursively copy a concept, its mappings, and its child concepts (such as answers or set members) from one source or collection into a source that you can manage.
* The $clone operation will skip any concepts already in the destination source, which is determined by checking if a concept in the destination source already has an “equivalency mapping” (e.g. “SAME-AS”) to the concept (or one of its child concepts) being cloned.
* The user performing the $clone operation must have read access to the originating source and write access to the destination source. 
* Note that cloning creates a new copy of the original concept; you are now responsible for maintaining that cloned concept. The only link to the original concept is a SAME-AS mapping from the new clone to its original concept. This means that your cloned concept will not be automatically affected by changes to the original concept and any changes you make to the cloned concept are independent of the original concept. 
    * However, $clone does "maintain the link" by creating a new mapping from each cloned concept to the original concept. And it also creates new mappings between cloned concepts and "equivalent" concepts that already exist in the same source. 
    * EquivalencyMapType – 
        * This default to “SAME-AS” (in TB only), and can be changed.
        * This is used in two ways:
            * To check if the equivalent concept with this MapType already exists in your destination Source, and,
            * To create the mapping between the cloned concept in your Source with the original one. 
        * You can specify multiple EquivalencyMapType parameters (e.g. SAME-AS,NARROWER-THAN) to find the equivalent concept, and if SAME-AS mapping doesn’t exist then it will iterate over other map-types, using the first map type in the list.
* ID and External ID Assignment
    * IDs and External IDs of cloned resources follow the configuration of the destination source.
    * If ID auto-assignment is disabled, next available ID is used – “incremental ID assignment” – this is to avoid possible ID conflicts
    * If External ID auto-assignment is disabled, then the external ID is left blank
* The $clone operation leverages OCL’s $cascade operation parameters. OCL performs a cascade on one or more concepts to determine which concepts and mappings are intended to be cloned. More information about the $cascade operation can be found here: [https://docs.openconceptlab.org/en/latest/oclapi/apireference/cascade.html](https://docs.openconceptlab.org/en/latest/oclapi/apireference/cascade.html) 
    * The $cascade operation returns the list of concepts and mappings to be cloned, with the ability to check if the concept already “exists” in the destination source. For example, if I already have a concept that is mapped as “SAME-AS” to the concept I intend to clone, OCL can recognize that my source does not want to clone that concept, so it omits the concept from the $cascade response. This ensures that my source does not get multiple copies of the same concept.
* The `$clone` operation relies on `equivalencyMapType` and `omitIfExistsIn` parameters to determine which concepts, if any, should be skipped when making a copy into the destination source.
    * Note: using these parameters, $clone can identify "equivalent" concepts that already exist in the destination source, ensuring that the same concept is not created again. 
    * $clone also creates new mappings to the “equivalent” concepts when appropriate. For example, this will ensure that an answer in the destination source can be mapped to its corresponding question 


### Cloning **Use **C**ase**s



* Example: There is a CIEL concept that I want to leverage in my concept dictionary. However, I need to change some things about it, such as its answers, name translations, etc., for it to work in my OpenMRS form. 
* Outcome: I will use the Clone feature to make a copy of the CIEL concept in my own source. I can then add it into my OCL collection, which serves as the concept dictionary for my OpenMRS implementation. I am aware that I am now fully responsible for maintaining that concept; any changes made to the original CIEL concept will not automatically be reflected in my cloned concept. Any changes I make to my new (cloned) concept will not automatically change the original CIEL concept.
* To see how OCL processes the $clone operation for this example, see this diagram: [diagram](https://mermaid.ink/img/pako:eNpdksFymzAQhl9lRxfHE-MH8KEZAqTtIW479iUTctjC2tYUJEUSbT3gd88KYwfMiZF2v3__X9uKQpckVmJv0Rxgm-YK-ItfE60KMh68ht8ERaUVlW8QRV_gsc3-S-cdSAX1EZxubEEPkHOnXNISDugAFdB7I_9iRcqHmxqNkWofcEzqi_UO_IHRZyEXzsZip_Mkj0GzeyHXQXK3-SPNfHy-1h2kd4kl9FcS3I-0J8JW7qXCaiCkPWGnLRAWh2vhMNql-ELtIGuf0YA_GnoYZst6wvO50bEbDHqeYTM-3HKlm_U8gxZr8sRSy_0SfkXxOo3is5SF5Mc6yX5uo0227eCpTTU5mHk9uxoKdRQyn0Y-TPH0mdC3SxSDmQV3WjIVFsHZhPlPevYsFc3HlJDn9xsIP2YJWJbTfs7zvaGG5n2fsbogNx27g3gcU1xVoPnBLYz2gWf-eivXL9LI51wsRE22RlnynraBmQsG1ZSLFf-WtMOm8rnI1YlLsfF6c1SFWHnb0EI0pmR6KpE3vBarHVaOTh-rT_1a?type=png) 


## Clone a Concept and its associated resources

```

POST /:ownerType/:ownerId/sources/:destinationSource/concepts/$clone/

```



* `:destinationSource` is the ID of the source that resources will be cloned into.
* Updates to a source are always made into the `HEAD` source version

**Input Parameters**

The `$clone` operation inherits all of the input parameters from the [`$cascade` operation documentation](https://docs.openconceptlab.org/en/latest/oclapi/apireference/cascade.html#get-a-list-of-resources-that-are-associated-with-a-concept-within-a-specific-source-or-collection-version)

* `expressions` (1..*) - List of one or more expressions referencing the resources to be cloned

* `parameters` (0..1) - Input parameters that control the behavior of the $clone operation


    * `mapTypes` (0..\*) - Comma-delimited list of map types used to process the cascade, e.g. `*` or `Q-AND-A,CONCEPT-SET`. If set, map types not in this list are ignored.


    * `excludeMapTypes` (0..\*) - Comma-delimited list of map types to exclude from processing the cascade. If set, all map types are cascaded except for those listed here. This parameter is ignored if `mapType` is set.


    * `returnMapTypes` (0..\*) - Comma-delimited list of map types to include in the resultset. If no value (the default), then this takes on the value of `mapTypes`. `*` returns all of a concept’s mappings; `false`/`0` will not include any mappings in the resultset.


    * `method` (0..1) - default=`sourcetoconcepts`; other option is `sourcemappings`. Controls cascade behavior via mappings, target concepts, or both. Note, to cascade everything (mappings, target concepts, and source hierarchy), use `method=sourceToConcepts` and `cascadeHierarchy=true`


      * `sourceMappings`: Cascade only mappings and not target concepts and not source hierarchy


      * `sourceToConcepts`: Cascades mappings and target concepts


    * `cascadeHierarchy` (optional; boolean) - default=true; if true (default), cascade a concept’s parent/child hierarchy relationships


    * `cascadeMappings` (optional; boolean) - default=true; if true (default), cascade a concept’s mappings


    * `cascadeLevels` (optional) - default="\*"; Set the number of levels of cascading to process from the starting concept. The number of levels of recursion for the cascade process, beginning from the requested root concept, where 0=no recursion, so only the root concept and its children/target concepts are processed. 1=one level of recursion, so the root concept, its children/targets, and their children/targets are included in the response. “\*”=infinite recursion, where the process recursively cascades until it is not possible to go further or until it has reached a hard limit (e.g. 1,000 resources, or as set by the system administrator). Note that the `$cascade` operator prevents infinite loops by not cascading a concept that has already been cascaded. Also note that each level of recursion applies the same “mapTypes” or “excludeMapTypes” filters.


    * `reverse` (optional; boolean) - default=false. By default, `$cascade` is processed from parent-to-child, from-concept-to-target-concept. Set `reverse=false` to process in the reverse direction, from child-to-parent and target-concept-to-from-concept.


    * `view` (string) - `flat` (default), `hierarchy`; Set to `“hierarchy”` to have entries returned inside each concept, beginning with the requested root concept. The default `“flat”` behavior simply returns a flat list of all concepts and mappings.


    * `omitIfExistsIn` (string) - Relative or canonical URL of a repository (or repository version) used to terminate the processing of a branch if the current concept already exists in the repository version specified in `omitIfExistsIn`. The matching concept and associated mappings will not be included in the result set, and its children (and associated mapping) will also be omitted from the result set unless they happen to be included via processing of another returned concept.


    * `equivalencyMapType` (string) - A map type (e.g. `SAME-AS`) used to terminate the processing of a branch if the current concept has an equivalent mapping in the repository version specified in `omitIfExistsIn`. This attribute has no effect if blank or if `omitIfExistsIn` is empty or does not point to a valid repository.


    * `includeMappings` DEPRECATED (optional; boolean) - default=true; if true, all mappings that are cascaded are included in the response; set this to false to exclude the mappings from the response and only include the concepts. This parameter is deprecated as it has been replaced by `returnMapTypes`, which has even more features.

**Output Parameters**
* `resourceType` - `Bundle`
* `type` - `searchset`
* `requested_url` - The relative URL of the original request
* `repo_version_url` - The relative URL of the repository version used to process the cascade request. If the repository version is specified in the original request, this will always reflect that. If the repository version was not specified
* `total` - In a flattened response, the total number of entries in the result set
* `meta` - Metadata about the request, such as `lastUpdated`
* `entry` - The concept from where the cascade operations was initiated
* `entry.type` - `Concept` or `Mapping`
* `entry.id` - ID/mnemonic of the resource
* `entry.url` - Version-less relative URL to the resource
* `entry.version_url` - Relative URL to the resource within its repository version
* `entry.retired`
* For concepts:
  * `entry.display_name`
  * `entry.terminal` - For a concept in the result set, `terminal` indicates if the cascade operation cut off the tree, if it is possible to continue cascading further given the input parameters, or if it is indeterminate:
	* `true` - The concept was cascaded, and there were no further results for the concept. i.e. The tree, as defined by the input parameters, does not go any further.
	* `false` - The concept was cascaded, and there were results that were included in the response. Note: For a hierarchical response, associated mappings and target concepts will always appear with the first occurrence of a concept.
	* `null` - The concept was not cascaded based on the provided parameters (e.g. cascadeLevels=1), so no information is available about whether the concept is terminal.
  * `entry.entries` - For a hierarchical response, includes the list of Mappings and target Concepts associated with this concept based on the input parameters.
* For mappings:
  * `entry.map_type`
  * `entry.sort_weight`
  * If `reverse=false`:
	* `entry.to_concept_code`
	* `entry.to_concept_url`
  * If `reverse=true`:
	* `entry.from_concept_code`
	* `entry.from_concept_url`
  * `entry.cascade_target_concept_code`
  * `entry.cascade_target_concept_url`
  * `entry.cascade_target_concept_name`
  * `entry.cascade_target_source_owner`
  * `entry.cascade_target_source_name`


## Example request and response



* TermBrowser Default – sets `cascadeLevels` and `returnMapTypes` to “all” (`*`), `mapTypes` to “Q-AND-A,CONCEPT-SET”, and `equivalencyMapType` to “SAME-AS”:

**Request:**


```
    POST /users/jamlung/sources/test3/concepts/$clone/
```



```
    {


      "expressions": [


    	"/orgs/CIEL/sources/CIEL/concepts/1790/"


      ],


      "parameters": {


    	"mapTypes": "Q-AND-A,CONCEPT-SET",


    	"excludeMapTypes": "",


    	"returnMapTypes": "*",


    	"cascadeLevels": "*",


    	"equivalencyMapType": "SAME-AS"


      }


    }
```

**Response:**

```

{

	"/orgs/CIEL/sources/CIEL/concepts/1790/": {

    	"status": 200,

    	"bundle": {

        	"resourceType": "Bundle",

        	"type": "searchset",

        	"total": 59,

        	"entry": [

            	{

                	"id": "1",

                	"type": "Concept",

                	"url": "/users/jamlung/sources/test4/concepts/1/",

                	"version_url": "/users/jamlung/sources/test4/concepts/1/326896/",

                	"retired": false

            	},

            	{

                	"id": "2",

                	"type": "Concept",

                	"url": "/users/jamlung/sources/test4/concepts/2/",

                	"version_url": "/users/jamlung/sources/test4/concepts/2/326898/",

                	"retired": false

            	},

            	{

                	"id": "3",

                	"type": "Concept",

                	"url": "/users/jamlung/sources/test4/concepts/3/",

                	"version_url": "/users/jamlung/sources/test4/concepts/3/326900/",

                	"retired": false

            	},

            	{

                	"id": "4",

                	"type": "Concept",

                	"url": "/users/jamlung/sources/test4/concepts/4/",

                	"version_url": "/users/jamlung/sources/test4/concepts/4/326902/",

                	"retired": false

            	},

            	…

            	{

                	"id": "48",

                	"type": "Mapping",

                	"map_type": "SAME-AS",

                	"url": "/users/jamlung/sources/test4/mappings/48/",

                	"version_url": "/users/jamlung/sources/test4/mappings/48/696446/",

                	"to_concept_code": "252109000",

                	"to_concept_url": null,

                	"cascade_target_concept_code": "252109000",

                	"cascade_target_concept_url": null,

                	"cascade_target_source_owner": "IHTSDO",

                	"cascade_target_source_name": "SNOMED-CT",

                	"cascade_target_concept_name": null,

                	"retired": false,

                	"sort_weight": null,

                	"from_concept_code": "11"

            	}

        	],

        	"requested_url": "/users/jamlung/sources/test4/concepts/$clone/",

        	"repo_version_url": "/orgs/CIEL/sources/CIEL/HEAD/"

    	}

	}

}

```



* This POST request clones one concept from CIEL, cascading through Q-AND-A and CONCEPT-SET map types through all (denoted by “*”) available levels, returning all map types. It checks the destination source to see if any concepts have a “SAME-AS” mapping to the CIEL concept to be cloned. Any concepts that would be duplicated in this way are skipped. The newly created resources are shown in the response for the POST request.

**Cloning in TermBrowser**



* OCL enables users to select one or more concepts and press the “Clone to Source” button to begin the $clone operation. When using the TermBrowser for this, it uses the following parameters:
    * **Origination Source(s)** - These parameters affect the operation with respect to the source from which the concept is being cloned
        * cascadeLevels (Default: *)
        * returnMapTypes (Default: *)
        * MapTypes (Default: “Q-AND-A,CONCEPT-SET”)
        * ExcludeMapTypes (Default: Null)
    * **Destination Source** - These parameters are populated in the TermBrowser using the source to which the concept will be cloned.
        * EquivalencyMapType (Default: “SAME-AS”)
* Note that these parameters are available for changing in the Advanced Settings menu. If unspecified, the default $cascade input parameters are used (see [here](https://docs.openconceptlab.org/en/latest/oclapi/apireference/cascade.html#get-a-list-of-resources-that-are-associated-with-a-concept-within-a-specific-source-or-collection-version)). 
* Using these parameters, users can click the “Visualize” button to view the $cascade operation outcome, providing a look at what concepts and mappings would be attempted to clone. These can also be viewed in a list view using the “Preview” button.
    * Note that the parameters defined in the Advanced Settings menu drive the Visualize and Preview features.
    * The details of the $clone operation can be viewed in the “API Details” section, showing what exactly will be submitted to the API when attempting to clone the concept into the destination source.
* After submitting the $clone operation, the number of resources cloned (including concepts and mappings) can be viewed. There is also a “Visualize” feature to see the $cascade results for the newly cloned concepts in the destination source. 
* For a demonstration of this operation in the TermBrowser, please see this YouTube video: (link TBD)

**Other notes:**



* When cloning hierarchical concepts, the hierarchy is not maintained in cloned concepts. Meaning, if you clone an ICD-10 concept with the intent of also cloning its children, you will make clones of the intended concepts, but they will not keep their parent-child relationships from ICD-10.
