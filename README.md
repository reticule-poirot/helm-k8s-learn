# helm-k8s-learn
Learn Helm and Kubernetes basics

Requires metrics-server

```commandline
% git clone https://github.com/reticule-poirot/helm-k8s-learn.git

% cd helm-k8s-learn

% helm install mynginx .
NAME: mynginx
LAST DEPLOYED: Wed Dec 18 21:31:43 2024
NAMESPACE: nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None

% kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/mynginx-helm-k8s-learn-5c8884786d-2lmk4   1/1     Running   0          9s

NAME                             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/mynginx-helm-k8s-learn   LoadBalancer   10.97.165.214   localhost     80:32740/TCP   9s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mynginx-helm-k8s-learn   1/1     1            1           9s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/mynginx-helm-k8s-learn-5c8884786d   1         1         1       9s
```
To test hpa

```commandline
% helm upgrade mynginx . --set autoscaling.enabled=true,autoscaling.targetCPUUtilizationPercentage=10
Release "mynginx" has been upgraded. Happy Helming!
NAME: mynginx
LAST DEPLOYED: Wed Dec 18 21:32:08 2024
NAMESPACE: nginx
STATUS: deployed
REVISION: 2
TEST SUITE: None

% kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/mynginx-helm-k8s-learn-5c8884786d-2lmk4   1/1     Running   0          4m20s

NAME                             TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/mynginx-helm-k8s-learn   LoadBalancer   10.97.165.214   localhost     80:32740/TCP   4m20s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mynginx-helm-k8s-learn   1/1     1            1           4m20s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/mynginx-helm-k8s-learn-5c8884786d   1         1         1       4m20s

NAME                                                         REFERENCE                           TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/mynginx-helm-k8s-learn   Deployment/mynginx-helm-k8s-learn   cpu: 1%/10%   1         3         1          3m55s
```

And make some requests
