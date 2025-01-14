# Four Keys + CDEvents for Kubernetes

Based on https://github.com/GoogleCloudPlatform/fourkeys and https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance

This project was designed to consume Cloud Events and allow you to track the Four Keys from the DORA report all based on Kubernetes making it portable accross cloud providers.

## Components

- `CloudEvents Raw`: endpoint to send all CloudEvents, these CloudEvents will be stored in the SQL database to the `cloudevents-raw` table. 

- `CDEvents tranformers`: These functions will read from the `cloudevents-raw` table and tranform the CloudEvents to CDEvents only when apply based on the function's mapping. The results needs to be stored into the cdevents-raw table for further processing.

- 

## Installation

This project was created to consume any CloudEvent available and store it into a SQL database for further processing. Once the CloudEvents are into the system a function based approach can be used to translate to CDEvents which will be used to calculate the "four keys".

We will install the following components in an existing Kubernetes Cluster: 
- Knative Serving: https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/  
- PostgreSQL: 
  - `helm install postgresql bitnami/postgresql`
  - `kubectl port-forward --namespace default svc/postgresql 5432:5432`
  - `export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)`
  - To connect from outside the cluster: `PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432`
  - Create Tables (on default database `postgres`): 
    
    - `CREATE TABLE IF NOT EXISTS cloudevents_raw ( event_id serial NOT NULL PRIMARY KEY, content json NOT NULL, event_timestamp TIMESTAMP NOT NULL);`
    - 
- Sockeye: `kubectl apply -f https://github.com/n3wscott/sockeye/releases/download/v0.7.0/release.yaml`

Cloud Event Sources: 

- Tekton: https://github.com/cdfoundation/sig-events/tree/main/poc/tekton
  - Tekton dashboard: `k port-forward svc/tekton-dashboard 9097:9097 -n tekton-pipelines`
  - Cloud Events Controller: `kubectl apply -f https://storage.cloud.google.com/tekton-releases-nightly/cloudevents/latest/release.yaml`
  - ConfigMap: `config-defaults` for SINK URL
- Github Source: https://github.com/knative/docs/tree/main/code-samples/eventing/github-source


- Kubernetes API Server Source: https://knative.dev/docs/eventing/sources/apiserversource/getting-started/#create-an-apiserversource-object


