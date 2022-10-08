# Helm Utils

You can see which repositories are configured using __helm repo list__
```bash
$ helm repo list
NAME                	URL                                               
prometheus-community	https://prometheus-community.github.io/helm-charts
grafana             	https://grafana.github.io/helm-charts             
gitlab              	https://charts.gitlab.io                          
bitnami             	https://charts.bitnami.com/bitnami                
tyk-helm            	https://helm.tyk.io/public/helm/charts            
```

New repositories can be added using:
```bash
helm repo add dev https://example.com/dev-charts
```


Output values from a specific helm/chart to a __values.yaml__ file. This file can be modified and then applied together with the chart. It will overwrite the chart values.

```bash
helm show values grafana/grafana > values.yaml
```

Once the values.yaml is applied, you can review all customized values applied in the chart.
```
$ helm upgrade -f panda.yaml happy-panda bitnami/wordpress

$ helm get values happy-panda

mariadb:
  auth:
    username: user1
```

## Creating your own chart
```
helm create my-new-chart
```

Once the command is executed, a new folder called __./my-new-chart__ is created, the chart will be build from that folder.

Use __helm lint__ to validate chart formatting and linting.

Use __helm package__ to package your chart for distribution.

```
helm package my-new-chart
my-new-chart-0.1.0.tgz
```

Then chart can be installed with __helm install my-new-chart ./my-new-chart-0.1.0.tgz__


## Template

Quote strings.
```
name: {{ .Values.MyName | quote }}
```

Don't quote integers.
```
port: {{ .Values.Port }}
```

__tpl__ function

```
# values
template: "{{ .Values.name }}"
name: "Tom"

# template
{{ tpl .Values.template . }}

# output
Tom
```

__Image Pull Secrets__

Images are a combination of _registry_, _username_, _password_. Most of times it is used __base64__ to encrypt the values. The values are usually stored in __values.yaml__ file.

```
imageCredentials:
  registry: quay.io
  username: bla
  password: ble
  email: bli@blo.blu
```

The helper template would be something like this:
```tpl
{{- define "imagePullSecret" }}
{{- with .Values.imageCredentials }}
{{- printf "{\"auths\":{\"%s\":{\"username\":\"%s\",\"password\":\"%s\",\"email\":\"%s\",\"auth\":\"%s\"}}}" .registry .username .password .email (printf "%s:%s" .username .password | b64enc) | b64enc }}
{{- end }}
{{- end }}
```

Then, use the helper template in a larger template to create the Secret Manifest.
```
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "imagePullSecret" . }}
```







