# pass-ui

PASS is an [Ember.js](https://emberjs.com/) application which provides a unified user interface that allow its users to deposit manuscripts into multiple repositories as required by applicable funding agency's public access policies

PASS communicates with an [Elide-based API](https://github.com/yahoo/elide), and [pass-core](https://github.com/eclipse-pass/pass-core) on the backend that serves json in conformance with the [JSON:API spec](https://jsonapi.org/).


## Technologies Utilized

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)


## Installation

* Clone the repository using the following command: `git clone https://github.com/eclipse-pass/pass-ui`
* Change into the directory that was cloned: `cd pass-ui`
* Run the following command: `pnpm i`


## Running `pass-ui` Inside the Docker Environment

The default environment for running `pass-ui` locally utilizes [pass-docker](https://github.com/eclipse-pass/pass-docker).

`pass-docker` can run with `pass-ui` running inside of the docker network in its own service and docker container with these [instructions](https://github.com/eclipse-pass/pass-documentation/tree/development/developer-documentation/pass-docker/README.md).

This environment is not as conducive for active development of `pass-ui`, depending on your host machine's operating system you might be able to run `pass-ui` on your host machine outside of the docker network and use these [instructions](https://github.com/eclipse-pass/pass-documentation/tree/development/developer-documentation/running-pass-ui-on-your-host-machine.md) to forward traffic from the docker network to your host machine.


### Building for the Docker Environment

GitHub automations are in place to produce production builds during a release. If you want to develop a build for local testing, you can use the `build.sh` script, specifying a `.env` file:

```sh
./build.sh ../pass-docker/.env
```

This script will remove any existing files in `dist/`, perform an Ember dev build, and create a new pass-ui Docker image with the `:latest` tag


## Running `pass-ui` Outside of the Docker Environment

It can be a better development experience to run `pass-ui` outside of `pass-docker`. To do this complete the following steps:


### Configure the Docker Environment

You will need to configure `pass-core` to load the UI from `localhost:4200`.

This can be done by seting the environment variable `PASS_CORE_APP_LOCATION` to `http://host.docker.internal:4200/app/` in `.env`. This will bypass the `pass-ui` container.

Then simply use docker compose like normal.

```
docker compose -f docker-compose.yml -f eclipse-pass.local.yml <your command here>
```

Note: You may need to investigate other ways of accessing the host machine network, [see](https://docs.docker.com/desktop/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host).

You may also consider stopping the pass-ui container.


### Run pass-ui on a Host Machine

Start ember on port 4200.

```
ember s
```


## Test Users

Refer to the LDAP service in `pass-docker` for [a list](https://github.com/eclipse-pass/pass-docker/blob/main/ldap/pass.ldif) of test users. Each has a password of `moo`.


## Configuration

The configuration for the docker environment occurs in the `pass-docker` [.env](https://github.com/eclipse-pass/pass-docker/blob/main/.env). A list of environment variables related to `pass-ui`, and other services, can be found there or in other [override .env files](https://github.com/eclipse-pass/pass-docker/blob/main/.eclipse-pass.local_env).

The application also gets "branding" configuration from a `config.json` file, with a default implementation found in the `public/` directory, which is automatically made available by default at `/app/config.json`.

`config.json`

```js
{
  "branding": {
    "homepage": "https://www.eclipse.org/org/foundation/",
    "logo": "ef/eclipse_foundation_logo_wo/EF_WHT-OR_png.png",
    "favicon": "favicon.ico",
    "stylesheet": "/app/branding.css",
    "overrides": "/app/branding-overrides.css",
    "pages": {
      "showPagesNavBar": false
    },
    "error": {
      "icon": "/app/error-icon.png"
    }
  }
}
```

The base theme styles can be found in `branding.css`. There are default fallback styles which can be overridden to customize the appearance of the UI. It is recommended to override these styles through a `branding-overrides.css` file. An example of these overrides to the base styles can be found [here](https://github.com/eclipse-pass/pass-ui/blob/main/public/branding-overrides.css).


## Testing

To run the Ember.js unit/integration/acceptance tests in this repository you can run one of the scripts in the [package.json](https://github.com/eclipse-pass/pass-ui/blob/main/package.json) or simply run the Ember development server via `ember s` and visit `http://localhost:8080/app/tests`.


## Linting

This project uses `es-lint`, `ember-template-lint` and `prettier` to enforce style decisions and code formatting. Consider installing [an integration tool](https://prettier.io/docs/en/editors.html).

This project uses (husky)[https://github.com/typicode/husky] to run a command from (lint-staged)[https://github.com/okonet/lint-staged] to run `es-lint --fix` and `prettier --write` over the staged files in a pre-commit hook. If issues arrise during a commit it might be because either one or both of these commands has failed. Check the output in the terminal for what failures have occurred.

There are also scripts defined in the [package.json](https://github.com/eclipse-pass/pass-ui/blob/main/package.json) that can manually lint check the project.


## CI

Testing and Linting utilizes the [ci.yml workflow](https://github.com/eclipse-pass/pass-ui/blob/main/.github/workflows/ci.yml) in Github when a pull request is opened and when there is a push to the `main` branch. More information regarding CI/CD can be found at the following link: [PASS Continuous Integration and Continuous Delivery](../../infrastructure-documenation/ci-cd/README.md)


## Related Documentation:

* [ember.js](https://emberjs.com/)
* [ember-cli](https://cli.emberjs.com/release/)
* Development Browser Extensions
  * [ember inspector for chrome](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi)
  * [ember inspector for firefox](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/)
* [PASS Continuous Integration and Continuous Delivery](../../infrastructure-documenation/ci-cd/README.md)
