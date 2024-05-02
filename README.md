# Week 4 Project - Release engineering at UpCommerce

James, the Engineering Lead at UpCommerce, liked the SRE team's ideas about taking the payment system implementation out of UpCommerce's original implementation and turning it into a microservice. However, he is skeptical about the implementation details. He has asked that the SRE team use a canary deployment strategy to implement the changes to the UpCommerce app. The details of the canary deployment are as follows:

-   Deploy a canary version of the app on port 5003.
    
-   The canary deployment should have only 1 replica, since it will serve only a few customers.
    
-   In the event that the canary fails, shut it down and rollback to the previous stable condition of UpCommerce's deployment before the canary release.
    
UpCommerce's DevOps team uses Helm as the tool of choice for release management.

## Getting your development environment ready

For this week's project, you will use GitHub Codespaces as your development environment, just like you did for the projects in previous weeks.

### Steps

1.  Create a fork of the [week's repository](https://github.com/onyekaugochukwu/sre-week-four).
    
2.  When you have created a fork of the week's repository, start a codespace on the `main` branch.
    
3.  Run the command below in your codespace's terminal to create a single-node, Kubernetes cluster using Minikube:
    
    `minikube start`
    
4.  Once your Minikube cluster is running, enter the command below:
    
    `kubectl create namespace sre`
    
    This creates a namespace in your Kubernetes cluster named `sre`. It is within this namespace that you'll do all the tasks required for this project.

### Tasks

1.  Deploy the helm chart using the command below:
    
    `helm install upcommerce ./upcommerce -n sre`
    
    Your original UpCommerce deployment should be up now. You can confirm this by running the command:
    
    `kubectl get deployment -n sre`
    
2.  In the `templates/` directory, create `canary-deployment.yml` and `canary-service.yml` manifest files and fill in the required Helm templates for them. If you need some ideas, you can find Helm templates for the aforementioned manifests below:

> #### canary-deployment.yaml template
>     
>     apiVersion: apps/v1
>     kind: Deployment
>     metadata:
>       name: {{ .Release.Name }}-canary-app
>     spec:
>       replicas: {{ .Values.replicaCount }}
>       selector:
>         matchLabels:
>           app: {{ .Release.Name }}-canary-app
>       template:
>         metadata:
>           labels:
>             app: {{ .Release.Name }}-canary-app
>         spec:
>           containers:
>             - name: canary
>               image: {{ .Values.canary.image }}
>               ports:
>                 - containerPort: 5003
>               imagePullPolicy: {{ .Values.imagePullPolicy }}
>               resources:
>                 limits:
>                   cpu: {{ .Values.cpuLimit }}
>                   memory: {{ .Values.memoryLimit }}`
> 

> #### canary-service.yaml template
>     
>     apiVersion: v1
>     kind: Service
>     metadata:
>       name: {{ .Release.Name }}-canary-service
>     spec:
>       selector:
>         app: {{ .Release.Name }}-canary-app
>       ports:
>         - protocol: TCP
>           port: 5003
>           targetPort: 5003`

3.  Update the `values.yml` file with the necessary values that will be cross-referenced in the canary templates (i.e. the image for the canary deployment) using the detail below:
    
    `canary:
    image: uonyeka/canary:linux-amd64`

    If you need a hint on what your `values.yml` file should look like after the update, click on the expandable section below:
    

>  #### values.yml manifest
>     
>     replicaCount: 1
>     imagePullPolicy: Always
>     cpuLimit: "0.5"
>     memoryLimit: "2Gi"
>     upcommerce:
>       image: uonyeka/upcommerce:v3
>     canary:
>       image: uonyeka/canary:linux-amd64

4. Run the command below to upgrade your Helm installation with the latest changes:
    
    `helm upgrade upcommerce ./upcommerce -n sre`
    
5. Run the command below to show you details of all deployments in the `sre` namespace:
    
    `kubectl get deployment -n sre -o wide`
    
    You'll notice that the canary deployment is failing and Kubernetes is trying to restart it continuously.
    
6. Because the canary deployment is unstable, you have to roll back the Kubernetes deployment to the last stable state. You can run the command below to get the list of all Helm releases in your cluster and their release numbers
    
    `helm list -a -n sre`
    
    or run the command below to get a historical overview of the Upcommerce release
    
    `helm history upcommerce -n sre`
    
    When you have gotten the release numbers, rollback to the stable release using the command below (replace `<release_number>` with the actual number of the release you want to roll back to):
    
    `helm rollback upcommerce <release_number> -n sre`
    
    To confirm that things have been rolled back, please run any one of the commands below:
    
    `helm list -a -n sre`
    `helm history upcommerce -n sre`
    
7. Here are some other commands that you can use to better understand and explore your Helm deployment:

	- helm ls -n sre    # List all releases (deployments).
	- helm status upcommerce -n sre   # Get detailed information about a specific release.
	- helm uninstall upcommerce -n sre  # Uninstall a release.
	- helm history upcommerce -n sre  # View the revision history of a release.
	- helm show values ./upcommerce -n sre  # Display default values from the chart.
	- helm show readme ./upcommerce -n sre  # Show the chart’s README.
	- helm lint ./upcommerce -n sre  # Check your chart for issues.
	- helm template upcommerce ./upcommerce -n sre  # Render templates without installing.

# References in Slack

## Articles and Resources - Salim Virji
 - Jennifer Mace on “Generic Mitigations” to quickly bring a system to a known-good state, or to cut away an inoperable part of the system: [https://www.oreilly.com/content/generic-mitigations/](https://www.oreilly.com/content/generic-mitigations/)
 - The Google SRE guide to Training SRE: [https://sre.google/resources/practices-and-processes/training-site-reliability-engineers/](https://sre.google/resources/practices-and-processes/training-site-reliability-engineers/)
	 - Also have a look at the “training” section of [sre.google/resources](http://sre.google/resources). I haven’t found anything that google has published about oncall and organizational effectiveness.
 - More about testing and mocks, from the "flamingo" book (Software Engineering at Google):  
	 - [https://abseil.io/resources/swe-book/html/ch11.html](https://abseil.io/resources/swe-book/html/ch11.html)
	 - [https://abseil.io/resources/swe-book/html/ch13.html#test_doubles_at_google](https://abseil.io/resources/swe-book/html/ch13.html#test_doubles_at_google)
	 - [https://google.github.io/googletest/gmock_for_dummies.html](https://google.github.io/googletest/gmock_for_dummies.html)
	- There’s a lot of good stuff in the flamingo book … Titus (one of the editors) is giving a talk on testing next week.  I’ll find the link and post it here
- Nice book I could found this reference as a resume by Video [https://learning.acm.org/techtalks/softwaregoogle](https://learning.acm.org/techtalks/softwaregoogle)
- In today's session I mentioned Sisyphus, a release-automation framework. Here's a background read on the project: [https://sre.google/resources/practices-and-processes/community-driven-software-adoption/](https://sre.google/resources/practices-and-processes/community-driven-software-adoption/)
- Jennifer Petoff, my colleague at Google and author of the Google SRE guide to Training SRE, is giving a talk on the topic later this month: [https://www.techstrongevents.com/skilup-day-sres-and-the-rise-of-platform-engineering/beg[…]=reg-3&utm_campaign=reg-3&utm_medium=email&utm_source=hs](https://www.techstrongevents.com/skilup-day-sres-and-the-rise-of-platform-engineering/begin-registration?ref=reg-3&utm_id=reg-3&utm_campaign=reg-3&utm_medium=email&utm_source=hs)

## General Slack Notes
- [Prodverbs, Pithy Insights From Running Production Services](https://sre.google/prodverbs/) - Prodverbs, SRE, Site Reliability Sayings
- [samber.github.io Awesome Prometheus alerts](https://samber.github.io/awesome-prometheus-alerts/rules.html) - Collection of alerting rules
- If you've never used GH secrets before, this is probably a good place to start: [https://docs.github.com/en/codespaces/managing-codespaces-for-your-organization/mana[…]pment-environment-secrets-for-your-repository-or-organization](https://docs.github.com/en/codespaces/managing-codespaces-for-your-organization/managing-development-environment-secrets-for-your-repository-or-organization#adding-secrets-for-a-repository) 
- Some good examples on how to write alerting rules for the four golden signals - [https://www.metricfire.com/blog/monitoring-kubernetes-with-prometheus/](https://www.metricfire.com/blog/monitoring-kubernetes-with-prometheus/) 
- A short tutorial on alerting from the official prometheus docs:  
[https://prometheus.io/docs/tutorials/alerting_based_on_metrics/](https://prometheus.io/docs/tutorials/alerting_based_on_metrics/)

- A video that walks you through the process of creating Slack alerts and email alerts. The video is attached to this message. I have also uploaded it to the Walkthrough page attached to the project on  [uplimit.com](http://uplimit.com/).
Here's the configmap (named  `combine.yml`  in the walkthrough video) you can use to configure your Prometheus Alertmanager:

>     apiVersion: v1
>     kind: ConfigMap
>     metadata:
>       name: prometheus-alertmanager
>     data:
>       alertmanager.yml: |
>         global:
>           resolve_timeout: 1m
>     
>         receivers:
>         - name: 'notifications'
>           email_configs:
>           - to: youremail@gmail.com
>             from: youremail@gmail.com
>             smarthost: smtp.gmail.com:587
>             auth_username: youremail@gmail.com
>             auth_identity: youremail@gmail.com
>             auth_password: xxxx xxxx xxxx xxxx
>             send_resolved: true
>             headers:
>               subject: "Prometheus - Alert"
>               text: "{{ range .Alerts }} Hi, \n{{ .Annotations.summary }}\n{{ .Annotations.description }} {{end}}"
>     
>           slack_configs:
>           - channel: '#upcommerce-devs'
>             send_resolved: true
>             api_url: '[https://hooks.slack.com/services/YOUR/SLACK/webhookxxxxxxxx](https://hooks.slack.com/services/YOUR/SLACK/webhookxxxxxxxx)'
>     
>         route:
>           group_wait: 10s
>           group_interval: 2m
>           receiver: 'notifications'
>           repeat_interval: 2m

There are two ways to amend your configmap:  

1.  Using vim editor (recommended and used in the walkthrough video above)
2.  Using the command:

    `kubectl apply -f combine.yml -n sre`  

While this is obviously faster,  `kubectl` will give you a warning alert, stating that it is not advisable to amend configmaps using  `kubectl apply`  when the configmaps were not created using the  `kubectl create`  command. (edited)

- Does anyone know of a good reference for useful metrics / queries to use for SLOs in Grafana?
	- [https://github.com/grafana/slo-workshop-breakouts](https://github.com/grafana/slo-workshop-breakouts)
	- [https://github.com/grafana/slo-workshop-breakouts/blob/main/lab2/Breakout_2_SLO_Creation_in_Grafana_Cloud.md#](https://github.com/grafana/slo-workshop-breakouts/blob/main/lab2/Breakout_2_SLO_Creation_in_Grafana_Cloud.md#)
	- [https://grafana.com/docs/grafana-cloud/alerting-and-irm/slo/best-practices/](https://grafana.com/docs/grafana-cloud/alerting-and-irm/slo/best-practices/)
	- [https://grafana.com/blog/2023/11/15/set-and-scale-service-level-objectives-in-grafana-cloud-introducing-grafana-slo/](https://grafana.com/blog/2023/11/15/set-and-scale-service-level-objectives-in-grafana-cloud-introducing-grafana-slo/)
	- [https://medium.com/tblx-insider/second-part-the-slos-playbook-edf995a89f90](https://medium.com/tblx-insider/second-part-the-slos-playbook-edf995a89f90)
- great project to monitor Kubernetes pods for network issues (it does a lot more, of course). It's called kubenurse: [https://github.com/postfinance/kubenurse](https://github.com/postfinance/kubenurse)
- [https://www.cncf.io/wp-content/uploads/2020/08/The-Illustrated-Childrens-Guide-to-Kubernetes.pdf](https://www.cncf.io/wp-content/uploads/2020/08/The-Illustrated-Childrens-Guide-to-Kubernetes.pdf)
