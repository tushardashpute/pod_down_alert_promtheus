# prometheus-alertmanager-slack-RMQ-kubernetes

# Pre-Requisites:
    EKS-Cluster
# Install helm 
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
    cp /usr/local/bin/helm /usr/bin
    which helm
# Deploy RMQ on EKS
    helm repo add stable https://charts.helm.sh/stable
    kubectl create namespace rabbit
    helm install mu-rabbit stable/rabbitmq --namespace rabbit
    
    Now login to RMQ pod and enable rabbitmq_management plugin, which will expose RabbitMQ metrics to /metrics endpoint.
    
    rabbitmq-plugins enable rabbitmq_management
    
    kubectl exec mu-rabbit-rabbitmq-0 -n rabbit -- rabbitmq-plugins enable rabbitmq_prometheus
    
    kubectl exec mu-rabbit-rabbitmq-0 -n rabbit -- curl -v -H "Accept:text/plain" "http://localhost:15692/metrics"
    
# Deploy Node Exporter 
    kubectl apply -f node-exporter
# Deploy kube-state-metrics
    kubectl apply -f kube-state-metrics
# Deploy alertmanager

    Edit the config.yml and update the slack web hook in it with your slack web hook.
    
    "https://api.slack.com/messaging/webhooks" --> How to create Slack app and get web hook URL
    
    slack_api_url: 'https://hooks.slack.com/services/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'

    kubectl apply -f alertmanager
# Deploy prometheus
    kubectl apply -f prometheus
# Note:
  Here I kept three rules: (we can get alaram)
  3. Messages in Queue if goes beyond 10
  2. CPU Utilization High for Nodes
  3. Pod status Pending/Unknown

# Alarm1: Create a new Queue TestQueue and post more than 12 messages, will get alarm

Now login to RMQ UI and create testQueue

kubectl get svc mu-rabbit-rabbitmq -n rabbit --> Now grab the ALB url and login 

http://url:15672

User : user
echo "Password      : $(kubectl get secret --namespace rabbit mu-rabbit-rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)"
echo "ErLang Cookie : $(kubectl get secret --namespace rabbit mu-rabbit-rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)"

Create Exchange 

![image](https://user-images.githubusercontent.com/74225291/145673483-94ba9293-2484-41ac-a54c-d35f04691830.png)

Then create Queue

![image](https://user-images.githubusercontent.com/74225291/145673641-717dd050-a358-49ff-b555-17b287a23dab.png)

Now bind the Exchange with Queue.

Exchange --> TestExchange --> Add Binding 

![image](https://user-images.githubusercontent.com/74225291/145673712-ff9d27c8-135e-4012-a362-32bb16ed515f.png)

Now post 10+messages to Queue, after one minutes alert with be in Firing state from Active state.

![image](https://user-images.githubusercontent.com/74225291/145672055-71d27167-8319-4ca9-bfce-e99d08148faa.png)

Now you will see alert notification in Slack as well.

# Alaram2: Connect to node and try to increase CPU Utilization, will get Alaram
# Alaram3: Deploy nginx
  kubectl apply -f nginx-deployment.yml
  Nginx Pod status will be under pending state, So will get Alaram
