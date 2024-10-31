# PASS Data Model

The PASS data model is represented using [JSON API](https://jsonapi.org/).

## Model Objects

* [Deposit](./model/Deposit.md): An attempt to push a publication to a repository.
* [File](./model/File.md): File being sent to a repository.
* [Funder](./model/Funder.md): The sponsor of a grant.
* [Grant](./model/Grant.md): Associates an award at the insitution with funders and principal investigators.
* [Journal](./model/Journal.md): The journal of a publication
* [Policy](./model/Policy.md): The institutional requirements to publish to certain repositories.
* [Publication](./model/Publication.md): The publication being sent to a repository
* [Repository](./model/Repository.md): The destination of a deposit
* [RepositoryCopy](./model/RepositoryCopy.md): A publication in a repository.
* [Submission](./model/Submission.md): The submission of a publication to a set of repositories.
* [SubmissionEvent](./model/SubmissionEvent.md): An event performed by a user on a submission.
* [User](./model/User.md): A PASS user.

## Model Diagram

<figure>
    <img src="/.gitbook/assets/pass_data_model.png" alt="Data Model Diagram">
    <figcaption>
        <p>Data Model Diagram</p>
    </figcaption>
</figure>

## Notes

### Identifiers

An object is uniquely identified by a tuple consisting of its id attribute and its type.

### DateTime attributes

DateTime attributes are strings formatted as per the Java DateTimeFormatter with pattern `yyyy-MM-dd'T'HH:mm:ss.SSSX`.
