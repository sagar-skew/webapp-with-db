# webapp-with-db
Here we are going to deploy the shortURL application using Kubernetics , with postgre SQL database.


Reference site:
https://medium.com/@xcoulon/deploying-your-first-web-app-on-minikube-6e98d2884b3a

############################################################
1. start minikube 
2. deploy postgre database on pod.
3. deploy web-application 
4. expose webapp to host.
5. play with shortanURL webapp.

##############################################################


kubectl api-resources | grep deployment
deployments                       deploy       apps/v1                                true         Deployment


minikube start --kubernetes-version v1.17.0	

kubectl get pod/postgres-5b4c76c777-vr5t2  -o go-template={{.metadata.labels}}
kubectl exec -it postgres-5b4c76c777-vr5t2 bash

root@postgres-5b4c76c777-vr5t2:/# psql -U user -d url_shortener_db
psql (9.6.5)
Type "help" for help.

sagar@workstation:~/sagar/webapp_with_DB$mm postgres
Name:              postgres
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=postgres
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.111.89.207
IPs:               10.111.89.207
Port:              <unset>  5432/TCP
TargetPort:        5432/TCP
Endpoints:         172.17.0.5:5432
Session Affinity:  None
Events:            <none>

kubectl get all -l app=webapp
NAME                        READY   STATUS    RESTARTS   AGE
pod/webapp-c7cd94d8-6kc49   1/1     Running   0          8m51s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-c7cd94d8   1         1         1       8m51s

sagar@workstation:~/sagar/webapp_with_DB$ kubectl logs webapp-c7cd94d8-6kc49
time="2022-02-25T15:18:01Z" level=info msg="Connecting to Postgres database using: host=`postgres:5432` dbname=`url_shortener_db` username=`user`\n"
time="2022-02-25T15:18:01Z" level=info msg="Adding the 'uuid-ossp' extension..."

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v3.2.1
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:8080


sagar@workstation:~/sagar/webapp_with_DB$ kubectl apply -f app_service.yaml
service/webapp created
sagar@workstation:~/sagar/webapp_with_DB$ kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          13d
postgres     ClusterIP   10.111.89.207   <none>        5432/TCP         21m
webapp       NodePort    10.108.62.154   <none>        8080:31317/TCP   6s
sagar@workstation:~/sagar/webapp_with_DB$ 



sagar@workstation:~/sagar/webapp_with_DB$ curl -v http://$(minikube ip):31317/ping
*   Trying 192.168.59.100:31317...
* TCP_NODELAY set
* Connected to 192.168.59.100 (192.168.59.100) port 31317 (#0)
> GET /ping HTTP/1.1
> Host: 192.168.59.100:31317
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Content-Type: text/plain; charset=UTF-8
< Date: Fri, 25 Feb 2022 15:25:51 GMT
< Content-Length: 5
< 
* Connection #0 to host 192.168.59.100 left intact
pong


sagar@workstation:~/sagar/webapp_with_DB$ curl -X POST http://$(minikube ip):31317/ -d ^C
sagar@workstation:~/sagar/webapp_with_DB$ 

sagar@workstation:~/sagar/webapp_with_DB$ curl -X POST http://$(minikube ip):31317/ -d "full_url=https://redhat.com"
2RqbV7Z

curl -X POST http://$(minikube ip):31317/ -d "full_url=https://google.com"

sagar@workstation:~/sagar/webapp_with_DB$ curl -X GET http://$(minikube ip):31317/2RqbV7Z -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.59.100:31317...
* TCP_NODELAY set
* Connected to 192.168.59.100 (192.168.59.100) port 31317 (#0)
> GET /2RqbV7Z HTTP/1.1
> Host: 192.168.59.100:31317
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 307 Temporary Redirect
< Location: https://redhat.com
< Date: Fri, 25 Feb 2022 15:29:34 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host 192.168.59.100 left intact


sagar@workstation:~/sagar/webapp_with_DB$ minikube service list --namespace default
|-----------|------------|--------------|-----------------------------|
| NAMESPACE |    NAME    | TARGET PORT  |             URL             |
|-----------|------------|--------------|-----------------------------|
| default   | kubernetes | No node port |
| default   | postgres   | No node port |
| default   | webapp     |         8080 | http://192.168.59.100:31317 |
|-----------|------------|--------------|-----------------------------|

