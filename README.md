## Tanzu RabbitMQ

This is a guide on how to set up Tanzu RabbitMQ using Carvel tooling.

### Prerequisites

Before you begin, make sure you have the following:

- Access to your Tanzu registry
- Docker credentials
- Kubernetes cluster
- kubectl CLI
- Carvel tooling

### Procedure

1. Log in to the registry to populate Docker credentials:

```
docker login registry.tanzu.vmware.com
```

2. Pull packages from the registry to a tar file:

```
imgpkg copy -b registry.tanzu.vmware.com/p-rabbitmq-for-kubernetes/tanzu-rabbitmq-package-repo:1.2.0 --to-tar rabbit-1.2.tar
imgpkg copy -b registry.tanzu.vmware.com/p-rabbitmq-for-kubernetes/tanzu-rabbitmq-package-repo:1.4.2 --to-tar rabbit-1.4.2.tar
```

3. Populate the internal registry with the image bundle:

```
imgpkg copy --tar rabbit-1.2.tar --to-repo harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo
imgpkg copy --tar rabbit-1.4.2.tar --to-repo harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo
```

4. Create a PackageRepository object YAML file with the following contents:

```
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  name: tanzu-rabbitmq-repo
spec:
  fetch:
    imgpkgBundle:
      image: harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo:1.2 # Replace BUNDLE_VERSION with the release version and replace registry.tanzu.vmware.com with your own registry url if the package is placed in another location
```

5. Deploy the PackageRepository object:

```
kapp deploy -a tanzu-rabbitmq-repo -f <PackageRepository_object_filename>.yml -y
```

6. Install the Cert Manager Tanzu package repository:

```
tanzu package repository add tanzu-standard --url harbor.h2o-2-10553.h2o.vmware.com/tanzu/packages/standard/repo:v2.1.1 --namespace tkg-system
```

7. Install the RabbitMQ package:

```
apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageInstall
metadata:
  name: tanzu-rabbitmq
spec:
  serviceAccountName: default # Replace with service account name
  packageRef:
    refName: rabbitmq.tanzu.vmware.com
    versionSelection:
      constraints: 1.2.0 # Replace with release version
```

8. Deploy the RabbitMQ package:

```
kapp deploy -a tanzu-rabbitmq -f packageinstall.yml -y
```

9. Once the installation is complete, you can create your RabbitMQ objects with Carvel tooling. Since the RabbitMQ container image will also require authentication, provide the same `imagePullSecrets` as before.

```
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: my-tanzu-rabbit
  namespace: rabbitmq-system
spec:
  replicas: 1
```

```
kubectl apply -f <rabbitmq_object_filename>.yml
```




