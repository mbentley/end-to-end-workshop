Welcome to the Tanzu End to End demo!  In this session, we'll be exploring some of the various capabilitites of Tanzu.

We're going to be using Tanzu to deploy an application, deploy dependent services for that application, observe the metrics for that application and supporting infrastructure, and manage the cluster hosting that application.

To get started, you need to clone Spring Pet Clinic to you can make some changes to it as part of the demo process.  Click the icon in the upper right of the box below to open a new browser tab so that you can fork the Spring Pet Clinic repo into your Github account.
```dashboard:open-url
url: https://github.com/tanzu-end-to-end/spring-petclinic/fork
```

We'll be logging into KubeApps next.  To do that, we'll need to grab our user token to use to login.  Click the running person icon in the upper right of the box below to get your token.
```terminal:execute
command: yq r ~/.kube/config 'users(name==eduk8s).user.token'
session: 1
```

Copy the resulting token into your clipboard, then open the following link in a new tab to start the process to deploy MySQL to your namespace. In the login screen, paste your token into the text field, and click "Login".  
```dashboard:open-url
url: https://kubeapps.{{ ingress_domain }}/#/c/default/ns/{{ session_namespace }}/apps/new-from-global/bitnami/mysql/versions/6.14.11
```

Within the resulting screen, change the "Name" of the service we're going to deploy to "petclinic-db".  Replace all the content in the "YAML" section with the following content.
```workshop:copy
text: |-
  db:
    name: petclinic
    password: petclinic
    user: petclinic
  replication:
    enabled: false
  root:
    password: petclinic
```
At the bottom of the page, click the "Deploy V6.14.11" button.  The database may take a minute or two to become ready.  

Next, you have a Concourse team created for you.  Let's login with the `fly` command in the terminal.
```terminal:execute
command: fly -t concourse login -c https://concourse.{{ ingress_domain }} -u test -p test -n={{ session_namespace }}
session: 1
```
Now, set your pipeline.
```terminal:execute
command: fly -t concourse set-pipeline -c pipeline/spring-petclinic.yaml -p spring-petclinic -v harborDomain=harbor.{{ ingress_domain }} -v harborUser=admin -v harborUser=Harbor12345 -v kubeconfigBuildServer=$(yq r ~/.kube/config -j) -v kubeconfigAppServer=$(yq r ~/.kube/config -j) -v concourseHelperImage=harbor.{{ ingress_domain }}/concourse/concourse-helper -v host=petclinic-{{ session_namespace }}.{{ ingress_domain }} -v image=harbor.{{ ingress_domain }}/{{ session_namespace }}/spring-petclinic -v tbsNamespace={{ session_namespace }} -v wavefrontApplicationName=petclinic-{{ session_namespace }} -v wavefrontUri=https://vmware.wavefront.com -v wavefrontApiToken=48b7b6a9-74ef-41f5-8002-f18dec9f82e2 -v wavefrontDeployEventName=petclinic-{{ session_namespace }}-deploy -v codeRepo=https://github.com/cdelashmutt-pivotal/spring-petclinic -v configRepo=https://github.com/tanzu-end-to-end/spring-petclinic-config
session: 1
```

Now, let's open a browser window to your pipeline.  Login with user "test" and password "test"
```dashboard:open-url
url: https://concourse.{{ ingress_domain }}/teams/{{ session_namespace }}/pipelines/spring-petclinic
```

Next, login to harbor with the user "admin" and password "Harbor12345", and navigate to your project called {{ session_namespace }}
```dashboard:open-url
url: https://harbor.{{ ingress_domain }}
```

Need to finish the rest of setup, but this is an example.