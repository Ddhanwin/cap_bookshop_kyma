
# Handbook for getting started with SAP BTP Kyma

Here you can find the basic pre-requisites and guidelines to start working with SAP BTP Kyma and deploying a CAP service in Kyma


## Pre-requisites

Software to install in developer workstation:

- BTP CLI 
- Kubectl (Kubernetes)
- Visual Studio Code 
- Docker for Desktop
- kubectl plugin for Kubernetes OpenID Connect (OIDC) authentication or Kubelogin
- Container registry.  → You can use public dockerhub registry or custom registry from gitlab etc


## Setting Up & Working with Container Registries using Gitlab

For uploading docker images you need a registry this can be a gitlab registry or any other public registry.
Steps to push a image to registry
- First login to registry
- docker build the container image with tag
- Push to the image to registry
Important - For your kubernetes cluster to pull images from registry would require a deploy token from gitlab registry. This also needs to be added to your kyma namespace as a secret.




## Prepare your kyma Development Environment for windows

Before you start, please make sure you have Kyma environment enabled in your subaccount in BTP.

In your windows machine perform below steps in cmd
- Install Chocolatey (Use link - https://chocolatey.org/install#generic)
- Install Kubectl
    - choco install kubernetes-cli
    - kubectl version --client
- Download and install kubectl oidc-login
    - Install Krew - https://krew.sigs.k8s.io/docs/user-guide/setup/install/ 
    - After installing krew install  oidc-login plugin this is required to perform the authentication.   
        - kubectl krew install oidc-login
- Test the kubectl installation
    - kubectl version --client
- Download the Kyma runtime kubeconfig
    - The kubectl tool relies on a configuration file called the kubeconfig, to configure access to the cluster. The kubeconfig, can be downloaded from BTP Kyma environment directly by clicking on KubeconfigURL
- Set the KUBECONFIG environment variable
    - Open a command line prompt on your computer. In the command line screen, type in the following:
        - set KUBECONFIG="<KUBECONFIG_FILE_PATH>"
        - Test the configuration by running this command:
            - kubectl config get-contexts
## Migration steps for CAP CF to Kyma

For migration of a CAP application we can use cds tooling.
But make sure that you have @sap/cds (6.8.2) and @sap/cds-dk (6.8.2) / or any stable packages mentioned as dependencies in package.json file and should be of same version.

Delete the node_modules folder and package-lock.json file from project folder.

- Inside app/project directory, run "cds add helm"
    - This will create a chart/ repository with initial configurations
    - Essentially, chart/ will replace whatever is configured in your current mta.yaml or similar file
    - The most important part is the app/chart/values.yaml file
    - Make sure you have your imagePulLSecret set, as well as updating all image.repository parameters
    - Set the port to 4004 (as we are using a CDS App)

Important - Make sure to add the app/chart folder to you repository. After each chart change, you will need to make sure that you run "helm upgrade --install APP ./chart --namespace NAMESPACE" for the cluster to be updated.

- Next step is to add most "standard" CAP services.
    - cds add xsuaa
        - If you have an existing xs-security.json file, you need to merge this into the xsuaa.parameters part of your values.yaml file
        - You can convert JSON → YAML: yq eval -P xs-security.json -oy
        - xsappname HAS to be unique per subaccount. You will thus need to modify xsappname.
        - This WILL create new roles/scopes. You NEED to modify your (test)users to use the correct roles based on the new xsappname.

        - Important - If you a) run helm uninstall or b) manually remove the xsuaa service instance, the roles WILL be lost on BTP. This also means that you MUST later re-add them to user groups and role collections!

    - cds add db
        - Straightforward, requires HANA to be mapped to the cluster
    - cds add destinations






## Deploying a CAP Node.js service to BTP Kyma 

- Clone the repo - https://github.com/Ddhanwin/cap_bookshop_kyma.git
- Ensure to create a namespace in your kyma cluster
- Also go to app/chart/values.yaml file and add the kyma cluster domain, registry deploy token secret details and container image registry details there.

To deploy this project run

```bash
    cds build --production
    pack build <your-container-registry>/cap_bookshop_kyma-srv --path gen/srv --builder paketobuildpacks/builder:base
    docker push <your-container-registry>/cap_bookshop_kyma-srv
    pack build <your-container-registry>/cap_bookshop_kyma-hana-deployer --path gen/db --builder paketobuildpacks/builder:base
    docker push <your-container-registry>/cap_bookshop_kyma-hana-deployer
    helm upgrade --install <service-name> ./chart --namespace <your namespace>
```
