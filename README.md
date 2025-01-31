# Managing applications in Virtual Machines with Red Hat Developer Hub and Ansible Automation Platform

## Resources needed to run this demo

You need an OpenShift cluster with OpenShift Virtualization capabilities. If you're a Red Hatter or a Red Hat partner, you can request an OpenShift cluster running in GCP for this demo. You can request it [here](https://catalog.demo.redhat.com/catalog?item=babylon-catalog-prod/gcp-gpte.ocp4-on-gcp.prod&utm_source=webapp&utm_medium=share-link).

If you're requesting an OpenShift cluster in GCP, you must add a new worker. Otherwise, there won't be enough resources to run your virtual machines. To add a new node, create a new MachineSet. You can find the template in this repository, specifically in `extra-node/machine-set-extra-node.yaml` (make sure you substitute the values according to the comments on that file). 

## Deploying demo environment

This demo uses all the software officially provided by Red Hat. Therefore, you need to provide certain usernames, passwords, and tokens so the different software can access the Red Hat CDN and use official packages. You'll also need to obtain a GitHub Personal Access Token (PAT). 

This is the list of information you need to obtain:

- Red Hat Customer Portal username and password: These are used to connect a RHEL system to Red Hat repositories using subscription-manager.

- Pool ID for an Ansible Automation Platform subscription: You can find this in the [Red Hat Customer Portal](https://access.redhat.com/) by navigating to Subscriptions (top left) > Subscriptions tab > Search "Ansible Automation Platform" > Click on your subscription and locate the Pool ID. Ensure it's a valid Ansible Automation Platform subscription.

- Token for Red Hat Automation Hub. Ansible will need this token to download and use official Red Hat collections. To obtain this token, navigate to the [Red Hat Hybrid Cloud Console](https://console.redhat.com) and go to Services (on the top left) > Automation > Automation Hub. Once there, click on "Connect to Hub" on the left side navigation panel. There, you'll be able to generate a token.

- Client ID and Client Secret for a Red Hat Customer Portal Service Account are used during the JBoss installation process. To obtain these values, you can easily create a Service Account. Go to the [Red Hat Hybrid Cloud Console](https://console.redhat.com) and click on the gear icon (top right). In the drop-down menu, click on "Service Accounts." Once there, create a service account and note down the Client ID and Client Secret.

- GitHub PAT (classic): Log in to GitHub and navigate to the [tokens settings site](https://github.com/settings/tokens) to create a classic token. This is needed to import the Git repo into the GitLab that we deploy locally for the demo. Even if the repo is public, you need a PAT to interact with the GitHub API.

- Cluster domain. You need to pass the application domain of your cluster to the Helm chart to properly deploy the demo. It should be something like `apps.cluster-dqxwv.dqxwv.gcp.redhatworkshops.io`. 

Once you have retrieved all the usernames, passwords, and tokens, go to your OpenShift cluster and install the OpenShift GitOps operator. Once installed, create an Application object with all the information you just collected under the helm parameters section. 

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rhdh2vm-demo-deploy
  namespace: openshift-gitops
spec:
  destination:
    server: 'https://kubernetes.default.svc'
  project: default
  source:
    helm:
      parameters:
        - name: cluster.domain
          value: <here-goes-your-openshift-cluster-domain>
        - name: github.pat
          value: <here-goes-your-github-pat>
        - name: rhn.username
          value: <here-goes-your-redhat-customer-portal-username>
        - name: rhn.password
          value: <here-goes-your-redhat-customer-portal-password>
        - name: rhn.poolId
          value: <here-goes-your-ansible-subscription-poolId>
        - name: automationhub.apiToken
          value: <here-goes-your-redhat-automationhub-apiToken>
        - name: rhn.clientId
          value: <here-goes-your-redhat-customer-portal-clientId>
        - name: rhn.clientSecret
          value: <here-goes-your-redhat-customer-portal-clientSecret>
    path: rhdh2vm-demo-deploy
    repoURL: 'https://github.com/redhat-iberia/rhdh2vm.git'
    targetRevision: HEAD
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 10m
      limit: 10
    syncOptions:
      - CreateNamespace=true
```
You can also find this application template under the directory `/argo-application` of this repository.

Add the application using OpenShift GUI, ArgoCD GUI, or with the `oc` command line using the command `oc apply -f rhdh2vm-demo-deploy-application.yaml`. After a few minutes, all resources will be ready.

## Demo walkthrough

You can run this demo and view everything from the Red Hat Developer Hub's perspective, but we will examine all the components involved. The first step is to open the following tabs in your browser:

 - Red Hat Developer Hub: Obtain the URL using `oc get route backstage-developer-hub -n rhdh-operator -o jsonpath='{.spec.host}'`.
 - Ansible Automation Platform: Obtain the URL using `oc get route my-aap -n aap` and login with the `admin` user. Get the password by running `oc get secret my-aap-admin-password -n aap -o jsonpath='{.data.password}' | base64 -d`
 - OpenShift GitOps (ArgoCD): Obtain the password by running `oc get secret openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d`, use the `admin` user, and get the URL from `oc get route openshift-gitops-server -n openshift-gitops`
 - GitLab: Get the URL from `oc get routes -n GitLab-system | grep "GitLab-webservice-default"`, the user has been harcoded with a username `user1` and a password of `redhat1!`

Once you have all those tabs opened, you can start the demo workflow. First, log into Red Hat Developer Hub.

![Image1](images/image1.png)

Once you click on "Login," it will redirect you to the Red Hat Build of Keycloak to provide your username and password. In this case, we are going to click on "GitLab" as it's the identity provider used during this demo.

![Image2](images/image2.png)

Log in via GitLab using `user1` and `redhat1!`:

![Image3](images/image3.png)

Authorize Keycloak to log in GitLab.

![Image4](images/image4.png)

You've logged into Red Hat Developer Hub.

![Image5](images/image5.png)

Go to Catalog and see that there is only one template available. This template will create a Virtual Machine, install JBoss on it, and deploy a Hello World application on top.

![Image6](images/image6.png)

Choose the template and fill in the required data through the wizard. First, give the application a name and leave the rest as it is. 

![Image7](images/image7.png)

Choose the size of the Virtual Machine. We recommend Medium for adequate performance.

![Image8](images/image8.png)

Paste your GitLab instance URL without `https://` so that Red Hat Developer Hub can create the appropriate repositories and access the scaffolding repository.

![Image9](images/image9.png)

Review your inputs. If everything looks good, click "Create."

![Image10](images/image10.png)

Red Hat Developer Hub will create all the resources.

![Image11](images/image11.png)

Wait until all steps are shown as completed.

![Image12](images/image12.png)

Now navigate to Catalog and click on the application you just created:

![Image13](images/image13.png)

It's available for your to click on it and navigate through its options

![Image23](images/image21.png)

This application we created calls the Ansible Automation Platform to execute a job that spawns a Virtual Machine, configures it, installs JBoss 8, and deploys a hello world application on it. We do this via a Custom Resource called AnsibleJob, made available by the Ansible Automation Platform Operator. Navigate to your OpenShift ArgoCD GUI to see the Argo applications created by Red Hat Developer Hub. 

![Image14](images/image14.png)

Look for one whose name ends in 'dev-build', click on it and see how it has deployed a resource of type AnsibleJob.

![Image15](images/image15.png)

If you navigate to you Ansible Automation Platform, you'll see there is a new job being executed. That is the job that the object AnsibleJob started.

![Image16](images/image16.png)

As you can see, we have made a call to the Ansible Automation Platform via the Red Hat Developer Hub. For this, we leverage the AnsibleJob object and ArgoCD capabilities. In our case, the Ansible Automation Platform is running on OpenShift, but it could be a completely independent instance.

In Ansible Automation Platform GUI, click on the Workflow being run.

![Image17](images/image17.png)

As you can see, our workflow has different steps. Once the first one appears as completed (with a green checkmark), you can view the Virtual Machine created in your OpenShift cluster.

![Image18](images/image18.png)

You can also see the virtual machine in the Topology view of Red Hat Developer Hub.

![Image19](images/image19.png)

And in the Kubernetes tab of Red Hat Developer Hub. 

![Image20](images/image20.png)

Your virtual machine has been created by Ansible Automation Platform. Although it's running in OpenShift Virtualization and could have been created with ArgoCD, we chose Ansible to simulate scenarios similar to those of customers using other virtualization platforms like VMware, KVM, Nutanix, AWS, Azure, etc.

Once the virtual machine is created, it will take some time for Ansible to complete its tasks: installing JBoss 8 on the virtual machine and then building and deploying our newly created application. While this process is finishing, we recommend taking a look at the repositories that Red Hat Developer Hub created for us. To do this, in Red Hat Developer Hub, go to the catalog entry of your application and click on "View Source" in the "About" section. 

![Image21](images/image21.png)

That will redirect you to the GitLab repository created for your application. You can navigate through it and see that it was created based on a skeleton in our scaffolding GitLab repository.

You can also navigate to the "development" organization of the GitLab instance. There, you'll find the base repository for our application that Red Hat Developer Hub created from a skeleton we provided to the template. 

![Image22](images/image22.png)

You can also see the list of repositories in the "development" organization. There, you'll find another repository with your application name followed by "-gitops." This is the repository that Red Hat Developer Hub has created to manage the ArgoCD resources.

![Image23](images/image23.png)

Lastly, wait until the Ansible Automation Platform has finished running the entire workflow. 

![Image24](images/image24.png)

You can now navigate to the route exposing the virtual machine with our application. To do this, go to your application in the Red Hat Developer Hub catalog, access the topology view, and click on the virtual machine. A blade menu will appear on the right side with two tabs; choose the "Resources" tab and scroll down until you see routes. 

![Image25](images/image25.png)

Click on the route associated with your virtual machine. Remember, you need to allow insecure (HTTP) traffic in your browser. The route will show you the JBoss 8 web interface.

![Image26](images/image26.png)

If you add the path `/helloworld` to your route, you'll see our web application

![Image27](images/image27.png)

With this, you've completed the demo and demonstrated how Red Hat Developer Hub can manage both legacy applications running in virtual machines and modern cloud-native ones. In this demo, we used OpenShift Virtualization, but thanks to the Ansible Automation Platform, you could easily reproduce this behavior with virtual machines on any legacy virtualization platform (like VMware or Hyper-V) or public cloud provider (AWS, Azure, GCP, etc.).