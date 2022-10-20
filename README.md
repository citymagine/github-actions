# Citymagine reusable workflows

## Backend workflows:
The workflows in this repository rely on a specific Maven dependency that builds the container on package goal.
We also rely on a `package.json` file to do the semantic versioning and release the proper tag, container when pushing on master.
```xml
<dependency>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
</dependency>
```

| Action name              | Description                                                       | Phase                     | Required variables                                                                                                                                     | Result                                        |
|--------------------------|-------------------------------------------------------------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------|
| action-back-dev-build    | Triggers a maven dev build                                        | On pull request           | Secrets: **PAT**                                                                                                                                       | A docker image for dev is pushed in registry  |
| action-back-make-release | Triggers a maven release build with automatic semantic versioning | On merge on master branch | Secrets: **PAT**                                                                                                                                       | A docker image for prod is pushed in registry |
| action-back-deploy       | Triggers a deployment on environment                              | On demand                 | Inputs: **release-tag** (image tag), **appname** (used in URLs), **environment** (prod/dev)<br/>Secrets: **PAT**, **KUBECONFIG**, **KUBECONFIG_PROD** | Application deployed                          |

## Frontend workflows: 

For frontend workflow, we rely on a `Dockerfile` placed in the root folder of the app. Dev and release build uses this Dockerfile to build the container and push it to the registry.

| Action name                | Description                               | Phase                     | Required variables                                                                                                                                     | Result                                                                                             |
|----------------------------|-------------------------------------------|---------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| action-back-dev-build      | Triggers a maven dev build                | On pull request           | Secrets: **PAT**                                                                                                                                       | A docker image for dev is pushed in registry                                                       |
| action-front-release-build | Triggers a tag build                      | On tag on repository      | Secrets: **PAT**                                                                                                                                       | A docker image for prod is pushed in registry                                                      |
| action-front-make-release  | Triggers an automatic semantic versioning | On merge on master branch | Secrets: **PAT**                                                                                                                                       | A new tag is available in the repository, the package.json is tagged properly for next dev version |
| action-front-deploy        | Triggers a deployment on environment      | On demand                 | Inputs: **release-tag** (image tag), **appname** (used in URLs), **environment** (prod/dev)<br/>Secrets: **PAT**, **KUBECONFIG**, **KUBECONFIG_PROD** | Application deployed                                                                               |


### Package.json example
```json
{
  "name": "MyAPI",
  "dependencies": {
    "@conveyal/maven-semantic-release": "^5.0.0",
    "@semantic-release/changelog": "^6.0.1",
    "@semantic-release/commit-analyzer": "^9.0.2",
    "@semantic-release/git": "^10.0.1",
    "@semantic-release/release-notes-generator": "^10.0.3",
    "semantic-release": "^19.0.3"
  },
  "overrides": {
    "semantic-release": "^19.0.3"
  }
}
```

### Secret values template

```json
                "env": "dev",
                "namespace": "dev",
                "database": {
                  "host":""
                  "name":"",
                  "user":"",
                  "password":"",
                  "schema":""
                },     
                "ingress": {
                    "hosts": [{
                        "host": "${{env.url}}"
                    }],
                    "cors": [{
                        "host": "http://localhost:4200"
                    },{
                        "host": "https://${{env.citymapurl}}"
                    },{
                        "host": "https://${{env.citymapperurl}}"
                    },{
                        "host": "https://${{env.citydesignerurl}}"
                    },{
                        "host": "https://${{env.cityroadurl}}"
                    }]
                }

```