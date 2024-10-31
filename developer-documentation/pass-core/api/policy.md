# Policy API

Contains the PASS policy service, which provides an HTTP API for determining the policies applicable to a given Submission, as well as the repositories that must be deposited into in order to comply with the applicable policies.

## Configuration

Configuration is achieved via the following environment variables:

* `PASS_POLICY_INSTITUTION`: This is the institution as it appears on User.affiliations for every user in the institution: e.g. "johnshopkins.edu"
* `PASS_POLICY_INSTITUTIONAL_POLICY_TITLE`: The value of Policy.title on the institution's Policy object
* `PASS_POLICY_INSTITUTIONAL_REPOSITORY_NAME`: The value of Repository.name on the intstitution's IR Repository object

## Policy Service

The `/policy/policies` endpoint determines the set of policies that are applicable to
a given submission.  Note:  The results may be dependent on _who_ submits the request.  For example, if
someone from JHU invokes the policies endpoint, a general "policy for JHU employees" may be included in the results.

### Policies Request

`GET /policy/policies?submission=${SUBMISSION_ID}`

### Policies Response

The response is a list of IDs to Policy resources, decorated with a `type` property. A type of `institution` indicates that this is a institutional policy, a type of `funder` indicates that the policy is describes the requirements of a funder.

```JSON
[
 {
   "id": "3",
   "type": "funder"
 },
 {
   "id": "22",
   "type": "institution"
 }
]
```

## Repositories Service

The `/policy/repositories` endpoint, for a given submission, calculates the repositories that may be
deposited into in order to satisfy any applicable policies for that submission.

### Repositories Request

GET `/policy-service/repositories?submission=${SUBMISSION_ID}`
or, with urlencoded (with encoded submission=${SUBMISSION_ID}) as the body:

### Repositories Response

The response is an application/json document that lists repositories sorted into required and optional buckets. Required repositories must be deposited to while optional repositories may be deposited to.

```JSON
{
  "required": [
    {
      "url": "1",
      "selected": false
    },
    {
      "url": "2",
      "selected": false
    },
    {
      "url": "3",
      "selected": false
    },
    {
      "url": "4",
      "selected": false
    },
    {
      "url": "5",
      "selected": false
    }
  ],
  "optional": [
    {
      "url": "6",
      "selected": true
    }
  ]
}
```

Repositories contained in the above list are JSON objects containing the following fields:

* `url`: the URL to the repository resource in Fedora
* `selected`: optional field.  Specifies if the repository should be selected by default in the UI or not.
