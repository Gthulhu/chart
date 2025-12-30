# Helm Charts for Gthulhu deployment


## Usage

The Gthulhu requires prometheus and grafana to be installed in your kubernetes cluster.

```bash
$  microk8s helm install  kube-prometheus-stack kube-prometheus-stack
```

To deploy Gthulhu using Helm charts, follow these steps:
```bash
$ microk8s helm install gthulhu gthulhu -f ./gthulhu/values-production.yaml
```

To uninstall Gthulhu, run the following command:
```bash
$ microk8s helm uninstall gthulhu
```

## Licence

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.