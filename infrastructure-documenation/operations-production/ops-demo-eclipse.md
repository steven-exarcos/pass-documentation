# Operations/Production - Eclipse Demo Site

The Eclipse Demo website is used to demonstrate PASS to the community and potential adopters of PASS. The demo website
is hosted at [demo.eclipse-pass.org](https://demo.eclipse-pass.org). This site is hosted from our JHU AWS infrastructure.
It has a different infrastructure from our production site as it's only running a container of Pass Core and Pass UI.
Additionally, the demo site is loaded with fake data for demonstration purposes.

The demo site uses a login page as opposed to using SSO integration. To login use the following fake username and 
password: `nih-user` and `moo`. There are other types of users that can be tested in the demo environment and their 
logins are the same as the local docker setup. A list of these test accounts can be found in the Welcome Guide 
[Setup and Run PASS Locally](../../welcome-guide/setup-run-pass.md#run-pass-locally).

## Submission Workflow

1. Login with username: `nih-user` and password: `moo`.
2. Enter submission workflow by clicking on the `Start new submission` button.
3. Enter DOI `10.1039/c7an01256j` into the DOI input field.
4. PASS will retrieve data and populate the submission 'title' and 'journal' fields. You may need to wait a moment for this to occur.
5. Grants step: go to next step and select an NIH grant: Click anywhere on the row to select grant: `R07EY012124 - Regulation of Synaptic Plasticity in Visual Cortex`. 
   * When a grant is selected, it should appear in a separate “Grants added to submission” table, appearing above the “Available grants” table.
6. Policies step: accept the currently selected policies (NIH policy is the default for most cases), click Next to proceed. 
7. Repositories step: accept the selected repositories (JScholarship should be selected by default), click Next to proceed. 
8. Metadata (details) step: accept the pre-filled metadata, click Next to proceed.
   * Metadata form should have most fields pre-populated with data from the DOI with each such field being read-only
9. Files step: add any file to the submission, click Next to proceed.
10. Review step: click Submit
    * Accept the Deposit Requirements for JScholarship pop-up by selecting the checkbox and clicking the "Next" button.
    * Click "Confirm" on the next popup, confirming that _"You are about to submit your files"_.
11. Lastly, a "thank you" screen will appear, indicating that the submission was successful.
12. You can now navigate to the submissions page to view your recently submitted submission!

## Known Limitations

### Mocked APIs

#### Authentication

* Static set of users matched with demo sample PASS User entities.
* Current users (will be) used for acceptance testing, manual testing. Mix of fully faked users and custom users for the PASS team with faked credentials. These include GitHub usernames but no other real information.
* Cannot dynamically register new users or update passwords.

#### Manuscript file lookup service

Intended to lookup open access files that already exist online for a given DOI

* Fully mocked in browser to return a static dataset
* Returns a set of three files for a real DOI (`doi:10.1038/nature12373`)

#### File (upload) handling

File handling for PASS submissions. Used for CRUD operations for the manuscript and supporting files.

* Fully mocked, not very nicely resulting in some errors in the JS console.
* While the `File` entity will appear on the submission review page, none will appear on the submission's details page. For "proxy" submissions, this means that a user cannot directly submit a pre-prepared submission, but instead will have to edit the submission and (re)attach the file to the submission.

#### Metdata schema service

Returns the set of metadata requested for the set of target repositories for the in-progress submission as JSON schema to drive one or more forms.

* Fully mocked to return a common schema, meaning it will not respond to different repository requirements.

#### Repository service

Gets a set of repositories that can or must receive the submission.

* Fully mocked, always returns PubMed Central and JScholarship.

#### Policy service

Gets a set of open access policies that apply to the submission, determined after a user determines the target repositories. Technically part of the same service as the `Repository` service described above.

* Fully mocked, always returns the PMC and JHU policies.

### Other Known Issues

* Proxy submission can't be directly submitted after the preparer finalizes the submission due to the file handling.