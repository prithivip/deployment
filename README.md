Create two nodes in GCP as kmaster and knode1 and install kubernetes cluster.

Deploy application using ArgoCD 

root@proxy:~# k get nodes

root@proxy:~# k get namespace -ALL

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get -n argocd all

To access the argocd from UI we need to change the service of argocd-server to Nodeport/Loadbalancer. We will change to Loadbalancer.
Copy External IP address from GCP from kmaster node

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer", "externalIPs":["34.56.64.29"]}}'

root@proxy:~# alias k=kubectl

root@proxy:~# k get -n argocd all


k get secret -n argocd argocd-initial-admin-secret -o yaml

echo "YVVIUDB4Um9KSm9ZTkpsRA==" | base64 -d

Using the password you can login to UI using https://34.56.64.29:30268

Now we install argocd CLI in kmaster

curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x /usr/local/bin/argocd

argocd version

export ARGOCD_SERVER=34.56.64.29:30268

export ARGO_PWD=aUHP0xRoJJoYNJlD

argocd login $ARGOCD_SERVER --username admin --password $ARGO_PWD --insecure

'admin:login' logged in successfully

Context '34.56.64.29:30268' updated

Now we tested Argo login in CLI. Lets login to UI https://34.56.64.29:30268 and change password for admin user by clicking User Info followed by UPDATE PASSWORD

Now start creating our git repo where we keep all kubernetes deployment files, for that we add our repo into ArgoCD connection.

Click Settings --> Repositories --> Connect Repo --> Choose VIA HTTPS in connection method 
Select the connection method as “Https”, Then project as “default”. We can also use different project if we have created one. Then give our repo URL https://github.com/prithivip/deployment.git . Then click on connect 

Now we start creating our application

Click Applications --> NEW APP --> Fill below details
Application Name : my-namespace
Project Name : default
SYNC POLICY : Manual
Repository URL : Select GIT URL
Revision : HEAD  and choose Branches
Path : .
Cluster URL : https://kubernetes.default.svc
Namespace : my-namespace
Click CREATE at the top 

Now app is created.

Then click on sync. App will be synced and deployed.

Login to the kmaster node

root@proxy:~# k get deploy -n my-namespace

root@proxy:~# k get pods -n my-namespace

root@proxy:~# k get rs -n my-namespace
NAME                DESIRED   CURRENT   READY   AGE
my-app-7d7dbb4d95   10        10        10      74m


