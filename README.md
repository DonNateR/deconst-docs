[![Build Status](https://build.deconst.horse/deconst/deconst-docs/badge?branch=master)](https://build.deconst.horse/deconst/deconst-docs/)

# Deconst documentation

Documentation for the *deconst* project: a continuous delivery pipeline for heterogenous documentation. It's also used to develop deconst itself, like some kind of documentation ouroboros. You can read the documentation itself at [deconst.horse](https://deconst.horse/).

## Building

To build this documentation standalone, use the [Deconst client](https://github.com/deconst/client). Use the Sphinx preparer and a clone of `https://github.com/deconst/deconst-docs-control` as the control repository.

### DNS and TLS

DNS entries for `deconst.horse`, `build.deconst.horse`, `staging.deconst.horse`, and `content.staging.deconst.horse` are managed by Cloud DNS entries in the "drgsites" account. They should be pointed to the appropriate load balancers.

TLS certificates are currently retrieved from Let's Encrypt by a manual, downtime-inducing process. To reissue them:

* Run `script/reissue` in your [deconst/deploy](https://github.com/deconst/deploy) clone to reissue and download the new certificates to `le_certificates/`.
* Copy and paste the new certificate as the `ssl_key:` entry in `credentials.yml`.
* Use `script/encrypt` to encrypt the modified credentials file and commit and push it to nexus-credentials.
* Invoke Ansible to roll it out:

```bash
$ script/deploy \
   -e 'credentials_update=true' \
   -e 'service_pod_restart=true' \
   -e 'staging_restart=true' \
   -e 'strider_restart=true'
```

# Deconst Dev Env in Kubernetes with Minikube

These instructions will prepare and submit the content and assets for this deconst documentation in a dev env in Kubernetes with Minikube.

1. Run through [Deconst Dev Env in Kubernetes with Minikube](https://github.com/deconst/presenter#deconst-dev-env-in-kubernetes-with-minikube)

1. Prepare the content

    ```bash
    wget https://raw.githubusercontent.com/deconst/preparer-sphinx/master/deconst-preparer-sphinx.sh
    chmod u+x deconst-preparer-sphinx.sh
    ./deconst-preparer-sphinx.sh
    ```

1. Submit the content

    The `CONTENT_SERVICE_APIKEY` must match the `ADMIN_APIKEY` set when [adding the content service to the dev env](https://github.com/deconst/content-service#deconst-dev-env-in-kubernetes-with-minikube).

    ```bash
    export CONTENT_SERVICE_URL=$(minikube service --url --namespace deconst content)
    export CONTENT_SERVICE_APIKEY=$ADMIN_APIKEY

    wget https://raw.githubusercontent.com/deconst/submitter/master/deconst-submitter.sh
    chmod u+x deconst-submitter.sh
    ./deconst-submitter.sh
    ```
