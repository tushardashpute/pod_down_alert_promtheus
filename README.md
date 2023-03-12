# prometheus-alertmanager-slack-kubernetes

# Pre-Requisites:
    EKS-Cluster
# Install helm 
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
    cp /usr/local/bin/helm /usr/bin
    which helm
    
# Deploy Node Exporter 
    kubectl apply -f node-exporter
# Deploy kube-state-metrics
    kubectl apply -f kube-state-metrics
# Deploy alertmanager

Steps to configure slack:

https://ivanahuckova.medium.com/step-by-step-guide-to-setting-up-prometheus-alertmanager-with-slack-pagerduty-and-gmail-75069a8905b7

    Edit the config.yml and update the slack web hook in it with your slack web hook.
    
    "https://api.slack.com/messaging/webhooks" --> How to create Slack app and get web hook URL
    
    slack_api_url: 'https://hooks.slack.com/services/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'

    kubectl apply -f alertmanager
# Deploy prometheus
    kubectl apply -f prometheus
# Note:
  Here I kept three rules: (we can get alaram)
  1. KubernetesPodNotHealthy
  2. CPU Utilization High for Nodes

# Alarm1: Create a deployment with wrong image, so that pod will goto ErrImagePull state and alert will be thrown

        kubectl apply -f  nginx-deployment.yml

        $ kubectl get pods -l app=nginx
        NAME                                READY   STATUS             RESTARTS   AGE
        nginx-deployment-698b456684-kdfqn   0/1     ImagePullBackOff   0          45m
        nginx-deployment-698b456684-sqjlp   0/1     ImagePullBackOff   0          45m
        nginx-deployment-698b456684-wvjjt   0/1     ImagePullBackOff   0          45m

Now in the prometheus alert, you can see alert is in firing mode

kubectl port-forward svc/prometheus-service   9090:9090 &

![image](https://user-images.githubusercontent.com/74225291/224524435-49a06316-f7c2-4307-bf16-d92a7a4dc30a.png)


Now you will see alert notification in Slack as well.

![image](https://user-images.githubusercontent.com/74225291/224524403-e6b55e7c-7dc6-4fee-a58f-8c886e305db1.png)

# Alaram2: Connect to node and try to increase CPU Utilization, will get Alaram
