# Running traefik on k8s

## before we dive in acction is important to take care about:

+ Attach a label thoes nodes where traefik act as proxy. 

```
# Add label
kubectl label nodes worker-one.192.168.66.5.xip.io traefik=true

# Verify
kubectl get nodes --show-labels | grep traefik
```

+ Ensure port 80 and 443 are open on the previously labeled nodes as traefik=true → that node handle request on and provide access to the services on the cluster.

```
# open ports
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent

firewall-cmd --reload

# from masterone-k8s
vagrant ssh masterone-k8s
sudo su
ssh root@worker-one.192.168.66.5.xip.io 'firewall-cmd --zone=public --add-port=80/tcp --permanent && firewall-cmd --reload && exit'
```


## prepare the traefik escenario with: namespaces + crd + rbac

```
kubectl apply -f 00-prepare/00-AllTheWay.yaml
```

## apply main traefik manifests: deploy + config +  dashboad + secrets + middleware + metrics

```
# important note:
## to access on traefik dasboad you need to user/pass auth.
## to achive that you need use secret, two usefull site what
## let me get the value included on 04-secret → userfile:
### https://hostingcanada.org/htpasswd-generator/
### https://www.base64encode.org/

kubectl apply -f 01-traefik/00-AllTheWay.yaml

# if you have multiple pods
kubectl -n kube-system logs -f $(kubectl -n kube-system get pods -l app=traefik  -o jsonpath='{.items[0].metadata.name}')

# if you have just one pod
kubectl -n kube-system logs -f -l app=traefik
```

## apply k8s dashboard

```
## get the last version of k8s dashboard
https://github.com/kubernetes/dashboard/releases
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc5/aio/deploy/recommended.yaml

kubectl apply -f 02-dashboard-k8s/00-AllTheWay.yaml

## get the access token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

```

### the playground for use this repo come from:
https://github.com/jakubhajek/traefik-kubernetescrd
https://github.com/GIT-VASS/kubernetesSpray-v1.16.6-glusterfs

### special thanks:
https://github.com/felix-centenera
https://github.com/jakubhajek

