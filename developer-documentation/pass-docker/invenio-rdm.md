# PASS Docker Testing InvenioRDM

An InvenioRDM instance that can be run locally for testing [`pass-docker`](./README.md). **Note**: This is intended for testing a local instance of `pass-docker` and is not meant for Production use.

The `pass-docker-invenio-rdm` directory was created following these instructions: [InvenioRDM Installation](https://inveniordm.docs.cern.ch/install/)


## Technologies Utilized
* Python v3.8 or greater installed.
* The `invenio-cli` python tool installed and available in your PATH: [InvenioRDM CLI Installation](https://inveniordm.docs.cern.ch/install/cli/) 


### Running the InvenioRDM Instance 

Run the following commands in order to start the InvenioRDM instance:

```console
./build.sh
./start.sh
```

The above commands starts by building the application docker image, once the docker image has been built the commands will start the application and its related services (database, Elasticsearch, Redis and RabbitMQ). The build and boot process will take time to complete. The first time the commands run the docker images will need to be downloaded, the inital downloading of docker images will result in the commands taking longer to complete.


### Accessing the InvenioRDM Instance 

* Visit https://127.0.0.1 in your browser
* Click on the `user menu button` located in the top right corner 
* Click on `Applications`
* Click `New token` in the Personal access tokens
* Enter a Name and click `Create`
* Copy the Access token that was created
* Paste the token value in the `pass-docker/.eclipse-pass.local_env` as the value for the `INVENIORDM_API_TOKEN` property

**Note**: The server is using a self-signed SSL certificate, so your browser
will issue a warning that you will have to by-pass.


### Stopping the InvenioRDM Instance

To stop the InvenioRDM instance, run the following commands:

```console
./stop.sh
```
