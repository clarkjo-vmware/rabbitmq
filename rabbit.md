#Login to registry to populate docker creds
docker login registry.tanzu.vmware.com

#Pull packages from registry to tar.  
imgpkg copy -b registry.tanzu.vmware.com/p-rabbitmq-for-kubernetes/tanzu-rabbitmq-package-repo:1.2.0 --to-tar rabbit-1.2.tar
imgpkg copy -b registry.tanzu.vmware.com/p-rabbitmq-for-kubernetes/tanzu-rabbitmq-package-repo:1.4.2 --to-tar rabbit-1.4.2.tar


#Populate internal registry with image bundle (Ensure path matches default tanzu registry)
imgpkg copy --tar rabbit-1.2.tar --to-repo harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo
imgpkg copy --tar rabbit-1.4.2.tar --to-repo harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo

#Create a PackageRepository object yaml file with the following contents. Replace BUNDLE_VERSION below with the VMware RabbitMQ release version.

apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
  name: tanzu-rabbitmq-repo
spec:
  fetch:
    imgpkgBundle:
      image: harbor.h2o-2-10553.h2o.vmware.com/rabbitmq/tanzu-rabbitmq-package-repo:1.2 # Replace BUNDLE_VERSION with the release version and replace registry.tanzu.vmware.com with your own registry url if the package is placed in another location

kapp deploy -a tanzu-rabbitmq-repo -f <file> -y

#Install Cert Manager
tanzu package repository add tanzu-standard --url harbor.h2o-2-10553.h2o.vmware.com/tanzu/packages/standard/repo:v2.1.1 --namespace tkg-system


#Install Rabbit Package
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

kapp deploy -a tanzu-rabbitmq -f packageinstall.yml -y

#Deploy the PackageRepository object by running the following command. Replace <PackageRepository_object_filename> with the name of the PackageRepository yaml file that you used when you created the object.
kapp deploy -a tanzu-rabbitmq-repo -f <PackageRepository_object_filename>.yml -y

#Once this installation is complete, you can now create your RabbitMQ objects with the Carvel tooling. Since the RabbitMQ container image will also require authentication, you will need to provide the same #imagePullSecrets as earlier.

apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: my-tanzu-rabbit
  namespace: rabbitmq-system
spec:
  replicas: 1

kubectl apply -f <filename>
