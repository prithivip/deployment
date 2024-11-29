Create two nodes in GCP as kmaster and knode1 and install kubernetes cluster.

Deploy application using ArgoCD 

root@proxy:~# k get nodes
NAME      STATUS   ROLES           AGE   VERSION
kmaster   Ready    control-plane   40h   v1.29.11
knode1    Ready    <none>          40h   v1.29.11

root@proxy:~# k get namespace -ALL
NAME              STATUS   AGE    L
default           Active   40h    
kube-flannel      Active   40h    
kube-node-lease   Active   40h    
kube-public       Active   40h    
kube-system       Active   40h    
my-namespace      Active   52m


kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get -n argocd all

root@proxy:~# k get namespace -ALL
NAME              STATUS   AGE    L
argocd            Active   135m   
default           Active   40h    
kube-flannel      Active   40h    
kube-node-lease   Active   40h    
kube-public       Active   40h    
kube-system       Active   40h    
my-namespace      Active   52m

To access the argocd from UI we need to change the service of argocd-server to Nodeport/Loadbalancer. We will change to Loadbalancer.
Copy External IP address from GCP from kmaster node

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer", "externalIPs":["34.56.64.29"]}}'

root@proxy:~# k get -n argocd all
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                     1/1     Running   0          135m
pod/argocd-applicationset-controller-5ccbf46fd8-22hr7   1/1     Running   0          135m
pod/argocd-dex-server-7b6b9b5cb7-9ng99                  1/1     Running   0          135m
pod/argocd-notifications-controller-5b9566d4c7-d5rzf    1/1     Running   0          135m
pod/argocd-redis-665dd47d9b-6jt4f                       1/1     Running   0          135m
pod/argocd-repo-server-fc97bfb4d-ftgfs                  1/1     Running   0          135m
pod/argocd-server-69754c4fb-zrwch                       1/1     Running   0          135m


NAME                                              TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP      10.101.143.230   <none>        7000/TCP,8080/TCP            135m
service/argocd-dex-server                         ClusterIP      10.99.235.179    <none>        5556/TCP,5557/TCP,5558/TCP   135m
service/argocd-metrics                            ClusterIP      10.102.203.44    <none>        8082/TCP                     135m
service/argocd-notifications-controller-metrics   ClusterIP      10.111.208.122   <none>        9001/TCP                     135m
service/argocd-redis                              ClusterIP      10.106.84.185    <none>        6379/TCP                     135m
service/argocd-repo-server                        ClusterIP      10.111.12.37     <none>        8081/TCP,8084/TCP            135m
service/argocd-server                             LoadBalancer   10.110.169.24    34.56.64.29   80:30268/TCP,443:31929/TCP   135m
service/argocd-server-metrics                     ClusterIP      10.105.82.113    <none>        8083/TCP                     135m

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           135m
deployment.apps/argocd-dex-server                  1/1     1            1           135m
deployment.apps/argocd-notifications-controller    1/1     1            1           135m
deployment.apps/argocd-redis                       1/1     1            1           135m
deployment.apps/argocd-repo-server                 1/1     1            1           135m
deployment.apps/argocd-server                      1/1     1            1           135m

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-5ccbf46fd8   1         1         1       135m
replicaset.apps/argocd-dex-server-7b6b9b5cb7                  1         1         1       135m
replicaset.apps/argocd-notifications-controller-5b9566d4c7    1         1         1       135m
replicaset.apps/argocd-redis-665dd47d9b                       1         1         1       135m
replicaset.apps/argocd-repo-server-fc97bfb4d                  1         1         1       135m
replicaset.apps/argocd-server-69754c4fb                       1         1         1       135m

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     135m

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
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   10/10   10           10          73m
root@proxy:~# k get pods -n my-namespace
NAME                      READY   STATUS    RESTARTS   AGE
my-app-7d7dbb4d95-94kxf   1/1     Running   0          65m
my-app-7d7dbb4d95-bxbqd   1/1     Running   0          74m
my-app-7d7dbb4d95-cszrz   1/1     Running   0          65m
my-app-7d7dbb4d95-jbdqd   1/1     Running   0          65m
my-app-7d7dbb4d95-jl2rn   1/1     Running   0          74m
my-app-7d7dbb4d95-mqvvv   1/1     Running   0          65m
my-app-7d7dbb4d95-qkgcj   1/1     Running   0          65m
my-app-7d7dbb4d95-r2f7q   1/1     Running   0          65m
my-app-7d7dbb4d95-rjpbd   1/1     Running   0          74m
my-app-7d7dbb4d95-rtvh2   1/1     Running   0          65m

root@proxy:~# k get rs -n my-namespace
NAME                DESIRED   CURRENT   READY   AGE
my-app-7d7dbb4d95   10        10        10      74m


