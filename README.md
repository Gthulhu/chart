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

## Licence

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.