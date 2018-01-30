#Instruction

## Git repo
https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes

1. Create separate kubernetes namespace for jenkins
  
  $ kubectl create namespace jenkins
  
2. Create secret for Jenkins from creds.yaml

  $ kubectl apply -f k8s-jenkins/k8s/creds.yaml
  $ kubectl get secret -n jenkins

3. Create Jenkins pod and services:
  $ kubectl apply -f k8s-jenkins/k8s/jenkins.yaml
  $ kubectl get deployments -n jenkins
  
4. Create Jenkins services 
  $ kubectl apply -f k8s-jenkins/k8s/service_jenkins.yaml
  $ kubectl get services -n jenkins

5. Expose Jenkins service to local machine
  $ kubectl expose deploy jenkins --name=jenkins --type=LoadBalancer -n jenkins