# <div align="center">$${\color{purple}Kubernetes  \space Basic \space Testings}$$

<br>

- #### Start your Minikube cluster:
  ```
  minikube star --driver=docker
  ```
  Checking minikube cluster info.:
  ```
  minikube status
  kubectl get nodes
  kubectl cluster-info
  kubectl get pods -A
  minikube addons list

  #get all api-resources
  for i in `kubectl api-resources | awk '{print $1}'`; do echo  -e "-----------------\nkubectl get $i\n" && kubectl get $i; done
  ```

- #### Agnhost Usage:
  ```python
  $ kubectl run hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  $ kubectl exec pod/hello-node  -- /agnhost  --help
  Usage:
  app [command]

  Available Commands:
  audit-proxy                           Listens on port 8080 for incoming audit events
  completion                            Generate the autocompletion script for the specified shell
  connect                               Attempts a TCP, UDP or SCTP connection and returns useful errors
  crd-conversion-webhook                Starts HTTP server on port 443 for testing CustomResourceConversionWebhook
  dns-server-list                       Prints the host's DNS Server list
  dns-suffix                            Prints the host's DNS suffix list
  entrypoint-tester                     Prints the args it's passed and exits
  etc-hosts                             Prints the host's /etc/hosts file
  fake-gitserver                        Fakes a git server
  grpc-health-checking                  Starts a simple grpc health checking endpoint
  guestbook                             Creates a HTTP server with various endpoints representing a guestbook app
  help                                  Help about any command
  inclusterclient                       Periodically poll the Kubernetes "/healthz" endpoint
  liveness                              Starts a server that is alive for 10 seconds
  logs-generator                        Outputs lines of logs to stdout uniformly
  mounttest                             Creates files with given permissions and outputs FS type, owner, mode, permissions, contents of files
  net                                   Creates webserver or runner for various networking tests
  netexec                               Creates HTTP(S), UDP, and (optionally) SCTP servers with various endpoints
  nettest                               Starts a tiny web server for checking networking connectivity
  no-snat-test                          Creates the /checknosnat and /whoami endpoints
  no-snat-test-proxy                    Creates a proxy for the /checknosnat endpoint
  pause                                 Pauses the execution
  port-forward-tester                   Creates a TCP server that sends chunks of data
  porter                                Serves requested data on ports specified in ENV variables
  resource-consumer-controller          Starts a HTTP server that spreads requests around resource consumers
  serve-hostname                        Serves the hostname
  stress                                Lightweight compute resource stress utlity
  tcp-reset                             Serves on a tcp port and RST the connections received
  test-service-account-issuer-discovery Tests the ServiceAccountIssuerDiscovery feature
  test-webserver                        Starts a simple HTTP fileserver
  webhook                               Starts a HTTP server, useful for testing MutatingAdmissionWebhook and ValidatingAdmissionWebhook

  Flags:
  -h, --help                           help for app
      --log-flush-frequency duration   Maximum number of seconds between log flushes (default 5s)
  -v, --v Level                        number for the log level verbosity
      --version                        version for app
      --vmodule moduleSpec             comma-separated list of pattern=N settings for file-filtered logging (only works for the default text log format)

  Use "app [command] --help" for more information about a command.
  ```
<br>

- **connect:** Attempts a TCP, UDP or SCTP connection and returns useful errors  
  - Create agnhost-backend pod and service, the target server
  - Create hello-node pod, the testing server
  ```
  kubectl run agnhost-backend --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- netexec --http-port=1111
  kubectl expose  pod/agnhost-backend --port=2222 --target-port=1111 --protocol=TCP
  kubectl run hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - List all pods and serices
  ```python
  $ kubectl get all
  NAME                  READY   STATUS    RESTARTS   AGE
  pod/agnhost-backend   1/1     Running   0          14s
  pod/hello-node        1/1     Running   0          13s

  NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
  service/agnhost-backend   ClusterIP   10.111.10.160   <none>        2222/TCP   13s
  ```
  - Testing `connect` command
  ```
  kubectl exec pod/hello-node -- /agnhost connect agnhost-backend:2222 --protocol=tcp
  ```
<br>

- **audit-proxy:** Listens on port 8080 for incoming audit events  
  - Create and run the audit-proxy subcommand inside a Kubernetes Pod
  ```
  kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53 --port=8080 -- audit-proxy
  kubectl port-forward pod/test=agnhost 8080:8080
  ```
  - Inspect the proxy logs:
  ```
  kubectl logs -f test-agnhost
  ```
  - Trigger events:
  ```
  $
  ```
<br>

- **completion:** Generate the autocompletion script for the specified shell  
  - Create and run agnhost-completion pod
  ```
  kubectl run agnhost-completion --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - Create completion in bash, fish, powershell, zsh
  ```python
  kubectexec agnhost-competion -- /agnhost completion bash
  kubectexec agnhost-competion -- /agnhost completion fish
  kubectexec agnhost-competion -- /agnhost completion powershell
  kubectexec agnhost-competion -- /agnhost completion zsh
  ```
<br>

- **dns-server-list:** Prints the host's DNS Server list  
  - Create and run hello-node pod.  
  ```
  kubectl run hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - Prints the host's DNS Server list  
  ```python
  $ kubectl exec hello-node -- /agnhost dns-server-list
  10.96.0.10
  ```
<br>

- **dns-suffix:** Prints the host's DNS suffix list  
  - Create and run hello-node pod.  
  ```
  kubectl run hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - Prints the host's DNS suffix list  
  ```python
  $ kubectl exec hello-node -- /agnhost dns-suffix
  default.svc.cluster.local,svc.cluster.local,cluster.local
  ```
<br>

- **entrypoint-tester:** Prints the args it's passed and exits   
  ```python
  $ kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53 --rm -i --tty \
   -- entrypoint-tester hello world foo bar
  [/agnhost entrypoint-tester hello world foo bar]
  ```
<br>

- **etc-hosts:** Prints the host's /etc/hosts file
  - Create and run hello-node pod.  
  ```
  kubectl run hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - Prints the host's /etc/hosts file
  ```bash
  $ kubectl exec hello-node -- etc-hosts
  # Kubernetes-managed hosts file.
  127.0.0.1       localhost
  ::1     localhost ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  fe00::0 ip6-mcastprefix
  fe00::1 ip6-allnodes
  fe00::2 ip6-allrouters
  10.244.0.6      hello-node
  ```
<br>

- **fake-gitserver:** Fakes a git server    
  - Create and run test-fake-gitserver pod and service.  
  ```
  kubectl run test-fake-gitserver --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- fake-gitserver
  kubectl expose pod/test-fake-gitserver --port=8000
  ```
  - Forward pod's port 8000 to host machine port 8000  
  ```python
  kubectl port-forward service/test-fake-gitserver  8000:8000
  ```
  - Testing
  ```python
  $ curl -w "\n" http://localhost:8000
  I am a fake git server

  $ git clone http://localhost:8000
  Cloning into 'localhost'...
  warning: You appear to have cloned an empty repository.

  $ git clone http://localhost:8000  test-fake-gitserver
  Cloning into 'test-fake-gitserver'...
  warning: You appear to have cloned an empty repository.
  ```
<br>

- **grpc-health-checking:** Starts a simple grpc health checking endpoint  
  - Create and run grpc-test pod:  
  ```bash
  kubectl run grpc-test --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
   - Test and Verify Status: 
  ```
  ```python
  $ kubectl exec -it grpc-test -- /agnhost grpc-health-checking \
      --delay-unhealthy-sec 5 --service "" \
      --port 5000 --http-port 8080
  I1010 22:55:04.772001      16 log.go:245] Http server starting to listen on :8080
  I1010 22:55:04.772164      16 log.go:245] gRPC server starting to listen on :5000
  ```
  - Force an Unhealthy State: 
  ```bash
  kubectl exec grpc-test -- curl http://localhost:8080/make-not-serving
  ```
  
  - Checking  grpc-test status again 
  ```python
  $ ubectl exec -it grpc-test -- /agnhost grpc-health-checking \
      --delay-unhealthy-sec 5 --service "" \
      --port 5000 --http-port 8080
  I1010 22:55:04.772001      16 log.go:245] Http server starting to listen on :8080
  I1010 22:55:04.772164      16 log.go:245] gRPC server starting to listen on :5000
  I1010 22:55:14.349575      16 log.go:245] Mark as unhealthy
  ```

                             
- **guestbook:** Creates a HTTP server with various endpoints representing a guestbook app    
  - Create services and pods:  
  ```bash
  guestbook="https://raw.githubusercontent.com/kubernetes/kubernetes/master/test/e2e/testing-manifests/guestbook"
  sed_expr="s|{{.AgnhostImage}}|registry.k8s.io/e2e-test-images/agnhost:2.40|"

  # create the services.
  curl -sL ${guestbook}/frontend-service.yaml | kubectl create -f - 
  curl -sL ${guestbook}/agnhost-primary-service.yaml | kubectl create -f - 
  curl -sL ${guestbook}/agnhost-replica-service.yaml | kubectl create -f - 

  # create the deployments.
  curl -sL ${guestbook}/frontend-deployment.yaml.in | sed ${sed_expr} | kubectl create -f -
  curl -sL ${guestbook}/agnhost-primary-deployment.yaml.in | sed ${sed_expr} | kubectl create -f -
  curl -sL ${guestbook}/agnhost-replica-deployment.yaml.in | sed ${sed_expr} | kubectl create -f -
  ```
  - Check Pod and Service Status:  
  ```python
  $ kubectl get svc,pods -o wide
  NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
  pod/agnhost-primary-f5d5b56d7-lhbk5    1/1     Running   0          69m   10.244.0.64   minikube   <none>           <none>
  pod/agnhost-replica-5ccbf4c899-59bl2   1/1     Running   0          69m   10.244.0.65   minikube   <none>           <none>
  pod/agnhost-replica-5ccbf4c899-5x2c2   1/1     Running   0          69m   10.244.0.66   minikube   <none>           <none>
  pod/frontend-fdbc844f5-88v8h           1/1     Running   0          69m   10.244.0.63   minikube   <none>           <none>
  pod/frontend-fdbc844f5-r5r8x           1/1     Running   0          69m   10.244.0.62   minikube   <none>           <none>
  pod/frontend-fdbc844f5-x5skg           1/1     Running   0          69m   10.244.0.61   minikube   <none>           <none>

  NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE   SELECTOR
  service/agnhost-primary   ClusterIP   10.109.91.40     <none>        6379/TCP   69m   app=agnhost,role=primary,tier=backend
  service/agnhost-replica   ClusterIP   10.97.33.162     <none>        6379/TCP   69m   app=agnhost,role=replica,tier=backend
  service/frontend          ClusterIP   10.103.216.147   <none>        80/TCP     69m   app=guestbook,tier=frontend
  ```
  - Verify via Port-Forwarding:
  ```bash
  kubectl port-forward service/frontend 8088:80
  curl http://localhost:8088
  ```
  - Manual Verification:
  ```bash
  $ curl "http://agnhost-primary-svc:80/register?endpoint=http://<replica-pod-ip>:<backend-port>"

  $ curl "http://127.0.0.1:8088/register?endpoint=http://10.244.0.65:6379"
  {"data":{}}
  
  $ curl "http://127.0.0.1:8088/get?endpoint=http://10.244.0.65:6379"
  {"data":{}}
  
  $ curl "http://127.0.0.1:8088/set?endpoint=http://10.244.0.65:6379"
  cannot set with empty key.
  
  $ curl "http://127.0.0.1:8088/guestbook?endpoint=http://10.244.0.65:6379"
  unsupported cmd: ''
  ```





<br>
<br>
<br>
<br>
  
  ```
  kubectl run  hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53 --port=8080 -- netexec
  kubectl run  hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- netexec --http-port=8080
  kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  kubectl expose deployment hello-node --type=LoadBalancer --port=8080

  kubectl create deployment hiep-test --image=registry.k8s.io/e2e-test-images/agnhost:2.53 --port=8080 --replicas=2  -- /agnhost netexec --http-port=8080
  kubectl run agnhost-backend --image=registry.k8s.io/e2e-test-images/agnhost:2.53 --port=8080 -- netexec --http-port=8080
  kubectl expose  pod/agnhost-backend --port=8080 --target-port=8080
  ```
  ```python

  ```

  ```
  kubectl get all
  kubectl exec -it <pod-name> -- bash          #go inside agnhost:2.53 image os
  
  kubectl exec <pod-name> -- /agnhost --help   #usage: list all subcomand
  kubectl exec <pod-name> -- /agnhost dns-suffix
  kubectl exec <pod-name> -- /agnhost dns-server-list
  
  ```
  ```
  kubectl create deployment my-nginx --image=nginx
  kubectl expose deployment my-nginx --port=80 --target-port=8080
  kubectl exec hello-node-78b9f44d96-qszq7 -- /agnhost connect  10.244.0.25:80
  ```





  
