# Helm charts service playground

> ⚠️ **This is a development repository**: A chart older than a week will be automatically deleted. To pass your service into production, please see [Create a merge request](#create-a-merge-request)

This collection of Helm Charts is tailored for playground!
For all production services please use the [Service Helm Charts](https://gitlab.mercator-ocean.fr/pub/edito-infra/service-helm-charts) repository.

This tutorial provides the guideline to follow to create your own helm chart that can host a Docker image web application. Do not hesitate to look at other charts to get inspired.

First thing first, you will need to have a Docker image hosted on a public repository. This image should run a container exposing a web service through a port.

In this tutorial, we will take [TerriaMap datalab service](https://datalab.dive.edito.eu/launcher/service-playground/terria-map-viewer) as an example. The [TerriaMap github project is available here](https://github.com/TerriaJS/TerriaMap) if you want to know more about it.

And as you can see, this project satisfy the minimal requirement for hosting a web application on the datalab, which is having a public docker image available. You can find the corresponding [Dockerfile here](https://github.com/TerriaJS/TerriaMap/blob/main/deploy/docker/Dockerfile) if you need to inspire you. Here, it is only running a node application on the port **3001**.

## Clone the repository

```sh
git clone https://gitlab.mercator-ocean.fr/pub/edito-infra/service-playground.git
```

## Create your own chart folder

You can start by copying the content `terria-map-viewer` folder inside your own folder.

```sh
cp terria-map-viewer terriamap-copy-example
```

## Update the chart configuration (minimal)

### Edit the `Chart.yaml` file

Change the following fields and **leave the others unchanged**:

- **name** (the name of your service. This name must only consist of lower case alphanumeric characters, start with an alphabetic character, and end with an alphanumeric character. Hyphens (-) are allowed, but are known to be a little trickier to work with in Helm templates. The directory that contains a chart MUST have the same name as the chart)
- **description** (a brief description of your service)
- **home** (a page to learn more about your service, generate a "Learn more" button on the service tile)
- **icon** (an image that represent the underlying service)
- **keywords** (a list of useful keywords that can be used to retrieve your service from the datalab search bar)
- **version** (the version of the chart. Starts with 1.0.0 and update later if you need some changes)
- **appVersion** (the version of the service running inside your docker container. Maybe a version of your computation is present inside the repository where your service is versioned)

**All of these attributes are mandatory**, please find an icon even a generic one to illustrate your service.


```yaml
name: terriamap-copy-example
description: Run a TerriaMap viewer server copy example.
home: https://github.com/TerriaJS/TerriaMap
icon: https://github.com/TerriaJS/TerriaMap/raw/main/terria-logo.png
keywords:
  - Viewer
version: 1.0.0
appVersion: "8.3.0"
```

### Edit the `templates/NOTES.txt` file

The content will be rendered and displayed in a pop-up window while the service is being launched.

```txt
Your TerriaMap copy example application is being deployed.

It will be available on this [link](http{{ if $.Values.ingress.tls }}s{{ end }}://{{ .Values.ingress.hostname }}).
```

As you may see, you can use Helm values in this template file. Please take a look at the official [Helm documentation](https://helm.sh/docs/chart_template_guide/notes_files/) to learn more about it.

### Edit the `values.yaml` file

Change the `service.image.version` value by the link to your public docker image as well as the service port exposed.

```yaml
...
service:
  image:
    version: "ghcr.io/terriajs/terriamap:0.0.8"
...
networking:
  ...
  service:
    port: 3001
...
```

### Edit the `values.schema.json` file

Replace the Docker image links by the one you provided above as well as the service port. Note that you can provide multiple versions in the `listEnum` field.

```json
{
    ...
    "properties": {
        "service": {
            ...
            "properties": {
                "image" : {
                    ...
                    "properties": {
                      "version": {
                        ...
                        "listEnum": [
                            "ghcr.io/terriajs/terriamap:0.0.8"
                        ],
                        ...
                        "default": "ghcr.io/terriajs/terriamap:0.0.8"
                      }
                    }
                }
            }
        }
    ...
```

## Update the chart configuration (advanced)

### Customizable ingress

It is possible to let the user choose his own ingress (URL) for accessing his service from the datalab.
To add this behavior on your service, you'll need to edit the `values.schema.json` file and remove the line that allows to hide the ingress configuration in the datalab.
That's all. A new tab will appear in the datalab to let the user change the URL of his service.

```json
        "ingress": {
            "type": "object",
            "form": true,
            "title": "Ingress Details",
            "properties": {
                  ...
                },
                "hostname": {
                      ...
                    },
                    "x-onyxia": {
                        "hidden": true,  // Just remove this line
                        "overwriteDefaultWith": "{{project.id}}-{{k8s.randomSubdomain}}-0.{{k8s.domain}}"
                    }
                }
            }
        }
```

When you push your branch, your charts will automatically be published and accessible on EDITO datalab (they may be a 5-minute refresh delay).

### Access S3 storage

It is possible to load credentials to access a personnal S3 storage directly into your service. The following configuration will automatically import the credentials from the project settings configuration.

First, to automatically load S3 credentials into your service configuration, add the following property in the `values.schema.json` file:
```json
{
  "properties": {
    "s3": {
      "description": "Configuration of temporary identity",
      "type": "object",
      "x-onyxia": {
        "overwriteSchemaWith": "ide/s3.json"
      }
    },
    ...
  }
}
```

Then, add the following lines in the end of the `values.yaml` file:
```yaml
...
s3:
  enabled: true
```

Then, you can create a `secret-s3.yaml` file with the following line inside:
```yaml
{{ include "library-chart.secretS3" . }}
```
This will create a [Kubernetes Secret](https://kubernetes.io/fr/docs/concepts/configuration/secret/) with the required AWS keys:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`
- `AWS_S3_ENDPOINT`
- `AWS_DEFAULT_REGION`

Finally, use [envFrom](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables) to define all of the secret's data as container environment variables. The key from the secret becomes the environment variable name in the Pod:
```yaml
envFrom:
{{- if .Values.s3.enabled }}
- secretRef:
    name: {{ include "library-chart.secretNameS3" . }}
{{- end }}
```

### Deploy a stateful service
Right now, you have deployed a **stateless** service.
Depending on your needs, you might want to deploy a **stateful** service.
We encourage you to check a more complete (and complex) example that is our [Jupyter Python Ocean Science](https://gitlab.mercator-ocean.fr/pub/edito-infra/service-helm-charts/-/tree/main/ocean-modelling/jupyter-python-ocean-science?ref_type=heads) helm chart.
Instead of declaring a `Deployment`, you will need to write a `StatefulSet`.
You can read more about it in the official [Kubernetes StatefulSet documentation](https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/)


### Include Copernicus Marine Service credentials
It is possible to load Copernicus Marine Service credentials as environement variables in the service. The following configuration will automatically import the credentials configured in the user's `My Account`.

First, to automatically load Copernicus Marine Service credentials into the service configuration, add the following property in the `values.schema.json` file:
```json
{
  "properties": {
    ...
    "copernicusMarine": {
      "x-onyxia": {
        "overwriteSchemaWith": "copernicusMarine.json"
      }
    },
    ...
  }
}
```

Add the following properties in the `values.yaml` files:
```yaml
copernicusMarine:
  enabled: false
  username: ""
  password: ""
```

Then create a `secret-copernicusmarine.yaml` file inside the `templates` folder with the following content:
```yaml
{{- define "library-chart.secretNameCopernicusMarine" -}}
{{- if .Values.copernicusMarine.enabled }}
{{- $name:= (printf "%s-secretcopernicusmarine" (include "library-chart.fullname" .) )  }}
{{- default $name .Values.copernicusMarine.secretName }}
{{- else }}
{{- default "default" .Values.copernicusMarine.secretName }}
{{- end }}
{{- end }}

{{- if .Values.copernicusMarine.enabled -}}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "library-chart.secretNameCopernicusMarine" . }}
  labels:
    {{- include "library-chart.labels" . | nindent 4 }}
stringData:
  COPERNICUSMARINE_SERVICE_USERNAME: "{{ .Values.copernicusMarine.username }}"
  COPERNICUSMARINE_SERVICE_PASSWORD: "{{ .Values.copernicusMarine.password }}"
{{- end }}
```

Finally, load the secret values as environment variables in the container:
```yaml
envFrom:
  {{- if .Values.copernicusMarine.enabled }}
  - secretRef:
      name: {{ include "library-chart.secretNameCopernicusMarine" . }}
  {{- end }}
```

## Create a merge request

When you push your branch, your charts will automatically be published and accessible on EDITO datalab [service playground](https://datalab.dive.edito.eu/catalog/service-playground) (there may be a 5-minute refresh delay).

Once you think your chart is ready to be published, you can:

1. Make sure the metadata are complete in the `Chart.yaml` and `README.md` files
2. Please provide somehow a point of contact for the users to reach you
3. Pick a catalog category in which your contribution fit the best
4. Create a merge request on the repository and ping @pub/edito-infra/codeowners in the description to catch our attention.

If everything is good, we will migrate your charts to the other category, and you will be granted accesses to maintain them (bug fixes, new versions, etc.).
