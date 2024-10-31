# DOI API

The DOI API has two services. One service returns the corresponding PASS journal for the article identified by a DOI as well as the article's CrossRef metadata. The other service returns information about a manuscript from Unpaywall.

Both services accept the DOI of a journal article as a query parameter with a name of `doi`. The DOI should be formatted like `10.1234/ ...`. If a DOI is of a longer URL form containing the string `doi.org/`, then we truncate the DOI to take everything after this substring. If the DOI is not valid, a `400 Bad Request` status code is returned. On success a `200 OK` is returned. If there is an error the response will be a JSON object with an error key containing a message.

The services will look for an environment variable called `PASS_DOI_SERVICE_MAILTO` to specify a value on the User-Agent header on the Crossref and Unpaywall requests.

## `/doi/journal`

This service uses the Crossref API to get information about the article and its journal. We then check to see if there is a `Journal` object in PASS for this journal. If not we create one. The service then returns to the caller a JSON object containing the `journal-id` of the PASS journal, and a `crossref` object representing the data returned to the service as a result of the Crossref call. See the [Crossref docs](https://www.crossref.org/documentation/) for information on the Crossref metadata returned.

If a request is already being processed for the given DOI, `429 Too Many Requests` is returned.

### Example

Running this command:

```shell
curl -u BACKEND_USER:BACKEND_PASS  -H "X-XSRF-TOKEN:token" -H "Cookie:XSRF-TOKEN=token" localhost:8080/doi/journal?doi=DOI
```

Will return this JSON:

```JSON
{"journal-id":"72","crossref": {}}
```

## `/doi/manuscript`

This service uses Unpaywall API to get information about the corresponding locations on the web for manuscript PDFs related to the article referenced by the DOI.

Ultimately, we want a user of PASS to be informed of open access manuscripts that already exist on the web and be able to use those copies in their PASS submission, instead of having to manually upload the manuscript file(s).

The external service URLs are configured by the `XREF_BASEURI` and `UNPAYWALL_BASEURI` environment variables which default to `https://api.crossref.org/v1/works/` and `https://api.unpaywall.org/v2/` respectively.


On success a JSON object is returned with a key of manuscripts containing an array of manuscripts available to download. Each entry in the array is a JSON object.

Manuscript entry:
  * `url`: The url to download the manuscript.
  * `repositoryLabel`: The repository containing the manuscript, PubMed Central.
  * `type`: The mime type of the manuscript, application/pdf.
  * `source`: The source used to find the manuscript, Unpaywall.
  * `name`: The file name of the manuscript.
  * `isBest`: A boolean indicating whether this is the best entry to use

## Example

Running this command:

```shell
curl -u BACKEND_USER:BACKEND_PASS -H "X-XSRF-TOKEN:token" -H "Cookie:XSRF-TOKEN=token" localhost:8080/doi/manuscript?doi=DOI
```

Will return this JSON:

```JSON
  "manuscripts": [
    {
      "url": null,
      "repositoryLabel": "PubMed Central - Europe PMC",
      "type": "application/pdf",
      "source": "Unpaywall",
      "name": "test.pdf",
      "isBest": false
    },
    {
      "url": "http://pdfs.semanticscholar.org/good.pdf",
      "repositoryLabel": null,
      "type": "application/pdf",
      "source": "Unpaywall",
      "name": "good.pdf",
      "isBest": true
    }
  ]
```