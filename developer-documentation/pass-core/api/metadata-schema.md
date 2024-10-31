# Metadata Schema API

The metadata schema service is a RESTful service that provides JSON schemas intended to describe PASS submission metadata that will retrieve,
dereference, and place in the correct order all schemas relevant to a given set of pass Repositories.

## Schemas

The JSON schemas herein describe the JSON metadata payload of PASS [submission](/developer-documentation/pass-core/model/Submission.md) entities. They serve two primary
purposes

1. Validation of submission metadata
2. Generation of forms in the PASS user interface

These schemas follow a defined structure where properties in `/definitions/form/properties` are intended to be displayed by a UI, e.g.

```JSON
    {
        "title": "Example schema",
        "description": "NIHMS-specific metadata requirements",
        "$id": "https://github.com/eclipse-pass/metadata-schemas/jhu/example.json",
        "$schema": "http://json-schema.org/draft-07/schema#",
        "type": "object",
        "definitions": {
            "form": {
                "title": "Please provide the following information",
                "type": "object",
                "properties": {
                    "journal": {
                        "$ref": "global.json#/properties/journal"
                    },
                    "ISSN": {
                        "$ref": "global.json#/properties/ISSN"
                    }
                }
            },
        },
        "allOf": [
            {
                "$ref": "global.json#"
            },
            {
                "$ref": "#/definitions/form"
            }
        ]
    }
```

A PASS [repository](/developer-documentation/pass-core/model/Repository.md) entity represents a target repository where
submissions may be submitted.  Each repository may link to one or more JSON schemas that define the repository's metadata requirements.
In terms of expressing a desired user interface experience, one may observe a general pattern of pointing to a "common" schema containing ubiquitous fields, 
and additionally pointing to a "repository-specific" schema containing any additional fields that are unique to a given repository. There is also a "global"
schema which lists all the properties that may be present. All of these JSON schema properties become values in the metadata field of a submission.

As a concrete example, the NIHMS repository may point to the `common.json` schema, as well as the `nihms.json`schema. The schemas are stored as pass-core resources.

## Schema service

The schema service is an http service that accepts a list of PASS [repository](https://oa-pass.github.io/pass-data-model/documentation/Repository.html) 
entity IDs in a comma separated values String, in a GET request.  for example:

`schemaservice?entityIds=1,2,3`

For each repository, the schema service will retrieve the list of schemas relevant to the repository, place that list in the correct order (so
that schemas that provide the most dependencies are displayed first), and resolves all `$ref` references that might appear in the schema.

If a `merge` query parameter is provided (e.g., `?merge=true`), then all schemas will be merged into a single union schema. 
If the service is unable to merge schemas together, it will respond with `409 Conflict` status. 
In this case, a client can issue a request without the `merge` query parameter to get the unmerged list of schemas.

The result is an `application/json` response that contains a JSON list of schemas.

## HTTP Error Responses

The service will return the following HTTP error responses:
- 400 - Bad Request
  - This is returned when the request body is not valid list of comma separated entity Ids
- 409 - Conflict
  - This is returned when a schema is unable to be merged or when a schema fetch failed.
- 500 - Internal Server Error
    - This error is returned when an unexpected error occurs in the service.


## Usage Examples

### Retrieve schemas for a list of repositories

This command retrieves schemas for repositories with IDs 1, 2, and 3, merging them into a single schema if possible.

```shell
curl --location 'http://localhost:8080/schemaservice?entityIds=1,2,3&merge=true'
```

## Implementation details

There is a global schema. This defines the list of properties that can be present in the metadata blob. The metadata blob itself is a single JSON object that contains properties defined in that global schema. It is not divided into sections such as "common", "crossref", etc. The advantage of this approach is its simplicity - anything contributing to the metadata blob simply merges its output with the blob, and consumers of the blob simply look for the fields they need.
Individual schemas reference properties defined in the global schema, and are used for the dual purpose of display (to define the fields present in Alpaca forms displayed in the UI), and for validation.
A schema service that produces the N individual schemas drive Ember's UI and validation. These N schemas are based on the circumstances of a particular submission (who submitted it, what repositories it's going into, etc.)

Alpaca js renders forms based upon the contents of a JSON schema. That being said, a key insight to keep in mind is that form display and validation may be treated as separate concerns. This means that it is possible to write a schema that will render only a couple fields in Alpaca, but can be used to validate the required presence of many more in the metadata blob.  

### Global schema

The global schema defines all properties that can be present in the metadata blob, including their names, types, and Alpaca form options. These properties are used for both validation and display within the PASS user interface. Here's an example of a simplified global schema that defines two fields:

```JSON
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://localhost:8080/schemas/global.json",
  "title": "JHU global schema",
  "type": "object",
  "required": [
    "title",
    "journal-title"
  ],
  "additionalProperties": false,
  "properties": {
    "title": {
      "title": "Article / Manuscript Title",
      "description": "The title of the individual article or manuscript being submitted",
      "type": "string"
    },
    "journal-title": {
      "title": "Journal title",
      "description": "Title of the journal being submitted to",
      "type": "string"
    }
  },
  "options": {
    "fields": {
      "title": {
        "type": "textarea",
        "rows": 2,
        "cols": 100,
        "label": "Article / Manuscript Title",
        "placeholder": "Enter the manuscript title",
        "hidden": false
      },
      "journal-title": {
        "type": "text",
        "label": "Journal Title",
        "placeholder": "Enter the journal title",
        "hidden": false
      }
    }
  }
}
```

#### Required, additionalProperties:false

These properties are used for the purpose of schema validation.  

If the metadata blob is validated against the global schema, the required field enumerates the fields that MUST be present in the blob under all circumstances for it to be valid with respect to the global schema. If this is not present, then no fields are globally required.  

The additionalProperties: false field means that if the metadata blob is validated against the global schema, all properties in the blob must correspond to properties defined in the properties section of the global schema. Any unknown properties in the metadata blob (say, "foo" : "bar") will result in a schema validation error.   

#### Properties

The properties section defines the global set of properties allowable in the metadata blob. It is straightforward JSON schema.

#### Options

The options section of the global schema defines Alpaca options for each field. These values influence the display characteristics of each form field. Individual schemas that reference the global options will display form fields in a consistent manner.

### Individual schemas

Individual schemas list the properties to be displayed in Alpaca forms, and possibly define additional schema validation constraints.

An example hypothetical individual schema is as follows. It defines a form containing ONE field, `journal-NLMTA-ID`, but defines additional prerequisites - fields on the metadata blob that, while not input by this particular form, nevertheless must be present in order to successfully validate. In particular, this schema requires that `authors` be present.  One reason for doing this is if we wanted an NIHMS form to follow a "common" form, in which several fields have already been defined and filled out, and we (for whatever reason) don't wish to repeat the fields on the NIH-specific form.  For now, though, just focus on the parts of the schema and how they work

```JSON
{
   "title": "NIHMS schema",
   "type": "object",
   "$schema": "http://json-schema.org/draft-07/schema#",
   "definitions": {
       "form": {
           "title": "This is the title Alpaca displays in the UI",
           "type": "object",
           "required": ["journal-NLMTA-ID"],
           "properties": {
               "journal-NLMTA-ID": {"$ref": "http://localhost:8080/schemas/global.json#/properties/journal-NLMTA-ID"}
           },
	  "options": {"$ref": "http://localhost:8080/schemas/global.json#/options"}
       },
       "prerequisites": {
           "title": "prerequisites",
           "type": "object",
           "required": ["authors"],
           "properties": {
               "authors": {"$ref": "http://localhost:8080/schemas/global.json#/properties/authors"}
           }
       }
   },
   "allOf": [
       {"$ref": "http://localhost:8080/schemas/global.json#"},
       {"$ref": "#/definitions/prerequisites"},
       {"$ref": "#/definitions/form"}
   ]
}
```

The schema is divided into three sections, colored red, green, and blue.  These designate different parts of the schema that have different functions

#### /definitions/form

The "form" field of "definitions" contains the schema that will be displayed by Alpaca.  It MAY designate certain fields as required (in this example, journal-NLMTA-ID is required), and MUST contain property names that correspond to properties defined in the global schema.  Each property SHOULD reference the global schema property via a JSON reference ($ref).  All fields defined here MUST be displayed by Alpaca in the Ember UI.

This is a convention defined by PASS for the purpose of explicitly demarcating which forms are intended to be displayed.  The contents of /definition/form is a valid JSON schema as defined by the JSON schema spec.

#### /definitions/form/options

Individual schemas SHOULD define a /definitions/form/options property containing Alpaca-specific field definitions.  These contain labels for display, formatting information (e.g. text area height), and custom validation and/or behavior not specifically defined by JSON schema.  In general, for the sake of consistency, the global schema defines a global set of options, and individual schemas SHOULD reference i with a JSON reference.  That being said, it is fine for an individual form to customize the appearance of firm fields by defining their own set of options 

The location of the "options" property is a convention defined by PASS.  The contents and semantics of the "options" object are defined by Alpaca

#### /definitions/prerequisites

The "prerequisites" field of "definitions" defines a schema that is NOT displayed in screen.  Its sole purpose is for introducing validation constraints for metadata that is present in the blob, but not populated by the forms being displayed by this schema.  In the example, the "authors" field is required to be populated.  Schemas MAY have a prerequisites section if they have this additional validation need, but may omit if if not.  If validation with respect to this prerequisites is desired, the schema author should reference this section in the allOf section of the schema

The naming of this section is a convention for schema writers. The user interface will not specifically look at this section, and it is possible to name this section something else.  Only by referencing this section in the allOf section of the schema will this section be used for validation, as part of the normal JSON schema validation spec

#### /allOf

allOf is defined by JSON schema as a list of schemas (or references to schemas) that MUST be satisfied in order for the data to be considered valid with respect to this schema.  In the example shown, data must be valid with respect to three schemas, included by reference:

* Global schema
* Form schema
* Prerequisites schema

It is up to the author to decide the set of schemas to validate against.  Alpaca only validates against the form schema.  Individual schemas SHOULD validate against the global schema as well.  In any case, user interface SHOULD display a warning and MUST prevent submission (but possibly allow saving, in the case of proxy submission if incomplete metadata) of the metadata blob.  The validation MUST be against the contents of the metadata blob after all form data from a given form has been merged into it.

### Extending schemas with additional information

Additional properties may easily be added to /definitions in a schema without affecting this proposal.  For example, suppose one wishes to include mapping information from JSON fields to metadata fields in a particular format, for a particular repository.  This could be as easy as placing it in /definitions/mappings

```JSON
{
    "definitions": {
         "form": {
             // form stuff
         },
         "options" : {$ref: whatever},
         "mappings": {
             "repository": "JScholarship",
             "fields": {
	           "title": "dc.title",
                 // etc
             }
         }
    }
}
```

### Alpaca implementation notes

As mentioned earlier, forms intended to be displayed in individual schemas are present in `/definitions/form`.  This location is a convention defined by the PASS specification.  Therefore, the user interface must retrieve the contents of this section of the schema, and send it to Alpaca for rendering.  Likewise, the same must be done with Alpaca's options.  Furthermore, Alpaca must be provided the metadata blob in order to pre-populate fields according to pre-existing content This may be done as follows:

```Javascript
var blob = getMetadataBlob() // Get the pre-existing metadata blob somehow
var schema = getSchemaFromSchemaService()  // get the schema somehow

var form = {
   "data": data,
   "schema": schema.definitions.form,
   "options": schema.definitions.options
}

// Have Alpaca render the form
$("#whateverTheFormIs").alpaca(form)
```

Merging and validation.
Any operation that updates the contents of the metadata blob (including Alpaca forms) MUST merge the new contents with the existing contents of the blob.  For example, using JQuery:

```Javascript
$.extend( true, metadataBlob, alpacaFormData )
validationResults = validator.validate(metadataBlob, alpacaSchema)
```

This takes the content of the javascript object alpacaFormData, and merges it into metadataBlob.  

Schema validation should occur after relevant content has been merged in to the blob. It can occur immediately after entering in the form data, or as a final step prior to submission.  In the example above, alpacaSchema was the individual schema used to generate the form that produced alpacaFormData, and validation compared the resulting merged contents of metadataBlob with the individual schema.

Furthermore, please note that Alpaca mutates the schemas it is given, in ways that are incompatible with the JSON schema specification. Therefore, if you are validating a schema immediately after having displayed that schema in Alpaca, it is important to make a defensive copy - either of the schema given to Alpaca, or the validator.  For example:

```Javascript
var safeSchemaCopy = JSON.parse(JSON.stringify(schema))
var form = {
   "data": data,
   "schema": schema.definitions.form,
   "options": schema.definitions.options
}

// This will taint the contents of 'schema'
$("#whateverTheFormIs").alpaca(form)

// This will validate OK
validationResults = validator.validate(metadataBlob, safeSchemaCopy)
```

## Metadata blob documentation

The metadata blob is a JSON object stored as a string in the Submission metadata field.

The global JSON schema defines all the keys of the blob.


| Key               | Type   | Description                                                                                                                 | 
|-------------------|--------|-----------------------------------------------------------------------------------------------------------------------------|
| DOI               | string | Digital identifier assigned to published  article of the manuscript                                                         |
| title             | string | Title of manuscript/article                                                                                                 |
| journal-title     | string | Title of journal                                                                                                            |
| volume            | string | Journal volume                                                                                                              |
| issue             | string | Journal issue                                                                                                               |
| issns             | array  | array[issn,pubType] where issn is International Standard Serial Number and pubType is Print or Online                       |
| journal-NLMTA-ID  | string | National Library of Medicines Title Abbreviation                                                                            |
| publisher         | string | Publisher name                                                                                                              |
| publicationDate   | string | Publication date, captured in exact format entered on the form e.g. “October 2017”, “Fall 2012” “12/20/2016”                |
| abstract          | string | Publication abstract                                                                                                        |
| authors           | array  | array[author,orcid] Array of author names and corresponding orcid IDs                                                       |
| agent_information | object | Information about the browser                                                                                               |
| agreements        | array  | Array of objects which reprents ageements. Each object has a key with the repository name and a value of the agreement text |

Example of authors field:

```JSON
[{
    "author": "Bob Author",
    "orcid": "https://orcid.org/..."
  }, {
    "author": "Jane Writer",
    "orcid": "https://orcid.org/..."
  }]
```

Example of issn field:

```JSON
[{
  "issn": "0031-9023",
  "pubType": "Print"
  },
  {
  "issn": "1538-6724",
  "pubType": "Online"
  }]
```