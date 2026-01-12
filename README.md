# Helm Charts for Gthulhu deployment


## Usage

The Gthulhu requires prometheus and grafana to be installed in your kubernetes cluster.

```bash
$  helm install  kube-prometheus-stack kube-prometheus-stack
```

To deploy Gthulhu using Helm charts, follow these steps:
```bash
$ helm install gthulhu gthulhu -f ./gthulhu/values-production.yaml
```

To uninstall Gthulhu, run the following command:
```bash
$ helm uninstall gthulhu
```

## Testing

### API Manager

To access the Gthulhu API, you can set up port forwarding using kubectl:
```bash
$ kubectl port-forward svc/gthulhu-manager 8080:8080
```

After deploying Gthulhu, you can test the API by sending a login request using curl:
```bash
$ curl -X POST http://localhost:8080/api/v1/auth/login   -H "Content-Type: application/json"   -d '{
    "username": "admin@example.com",
    "password": "your-password-here"
  }'
{"success":true,"data":{"token":"<TOKEN>"},"timestamp":"2025-12-30T13:09:10Z"}
```

Then, you can create a new strategy by sending another curl request with the obtained token:

```bash
$ curl -X POST http://localhost:8080/api/v1/strategies \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>" \
  -d '{            
    "strategyNamespace": "default",
    "labelSelectors": [
      {"key": "app.kubernetes.io/name", "value": "prometheus"}
    ],
    "k8sNamespace": ["default"],
    "priority": 10,
    "executionTime": 20000000
  }'
```

You can also retrieve your own strategies using the following curl command:

```bash
$ curl http://localhost:8080/api/v1/strategies/self \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>"
```

### List all intents from decision maker

```bash
$ curl -X POST http://127.0.0.1:8080/api/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "public_key": "-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAny28YMC2/+yYj3T29lz6\n0uryNz8gNVrqD7lTJuHQ3DMTE6ADqnERy8VgHve0tWzhJc5ZBZ1Hduvj+z/kNqbc\nU81YGhmfOrQ3iFNYBlSAseIHdAw39HGyC6OKzTXI4HRpc8CwcF6hKExkyWlkALr5\ni+IQDfimvarjjZ6Nm368L0Rthv3KOkI5CqRZ6bsVwwBug7GcdkvFs3LiRSKlMBpH\n2tCkZ5ZZE8VyuK7VnlwV7n6EHzN5BqaHq8HVLw2KzvibSi+/5wIZV2Yx33tViLbh\nOsZqLt6qQCGGgKzNX4TGwRLGAiVV1NCpgQhimZ4YP2thqSsqbaISOuvFlYq+QGP1\nbcvcHB7UhT1ZnHSDYcbT2qiD3VoqytXVKLB1X5XCD99YLSP9B32f1lvZD4MhDtE4\nIhAuqn15MGB5ct4yj/uMldFScs9KhqnWcwS4K6Qx3IfdB+ZxT5hEOWJLEcGqe/CS\nXITNG7oS9mrSAJJvHSLz++4R/Sh1MnT2YWjyDk6qeeqAwut0w5iDKWt7qsGEcHFP\nIVVlos+xLfrPDtgHQk8upjslUcMyMDTf21Y3RdJ3k1gTR9KHEwzKeiNlLjen9ekF\nWupF8jik1aYRWL6h54ZyGxwKEyMYi9o18G2pXPzvVaPYtU+TGXdO4QwiES72TNCD\nbNaGj75Gj0sN+LfjjQ4A898CAwEAAQ==\n-----END PUBLIC KEY-----",
    "client_id": "gthulhu-scheduler",
    "expired_at": 1736899200
  }'

$ curl 127.0.0.1:8080/api/v1/scheduling/strategies -H "Content-Type: application/json" \
  -H "Authorization: Bearer <TOKEN>"
```

### View Scheduler Sidecar (decision maker) Logs

```bash
$ kubectl logs gthulhu-scheduler-hqflq -c scheduler-sidecar
```

## Licence

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.