# <div align="center">$${\color{purple}Kubernetes  \space Basic \space Testings}$$
# <div align="center">$${\color{purple}with \space Agnhost  \space Image}$$

<br>

 Agnhost (a play on "host agnostic") is a multi-platform CLI tool and container image maintained by Kubernetes (registry.k8s.io/e2e-test-images/agnhost) used primarily for end-to-end (e2e) testing and debugging across Linux and Windows environments. The examples below are based on 
 [Kubernets Instruction](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/README.md).
 
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
<br>
                             
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

- **inclusterclient:** Periodically poll the Kubernetes "/healthz" endpoint  
  - Create and run test-agnhost pod:  
  ```bash
  kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  ```
  - The subcommand will periodically poll the Kubernetes /healthz endpoint using the in-cluster config. Because of this, the subcommand is meant to be run inside of a Kubernetes pod. It can also be used to validate token rotation.
  ```python
  $ kubectl exec test-agnhost -- /agnhost inclusterclient [--poll-interval <poll-interval>]
  $ kubectl exec test-agnhost -- /agnhost inclusterclient --poll-interval 30
  I1011 14:52:44.151095      16 main.go:61] started
  I1011 14:53:14.240245      16 main.go:79] calling /healthz
  I1011 14:53:14.240773      16 main.go:96] authz_header=4mZ0nhGkqCIKnpdHTg4rDaReuMiy-46KhMQNoVo2AI0
  I1011 14:53:44.240319      16 main.go:79] calling /healthz
  I1011 14:53:44.240628      16 main.go:96] authz_header=4mZ0nhGkqCIKnpdHTg4rDaReuMiy-46KhMQNoVo2AI0
  I1011 14:54:14.240142      16 main.go:79] calling /healthz
  I1011 14:54:14.240599      16 main.go:96] authz_header=4mZ0nhGkqCIKnpdHTg4rDaReuMiy-46KhMQNoVo2AI0
  ```
<br>

liveness                              Starts a server that is alive for 10 seconds
- **liveness:** Starts a server that is alive for 10 seconds    
  - Create a manifest file liveness.yaml to test liveness:  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      test: liveness
    name: liveness-http
  spec:
    containers:
    - name: liveness
      image: registry.k8s.io/e2e-test-images/agnhost:2.40
      args:
      - liveness
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
        initialDelaySeconds: 3
        periodSeconds: 3
  ```
  - Apply to create the pod:
  ```python
  kubectl apply -f liveness.yaml
  ```
  - Checking:
    The Agnhost liveness will starts a simple server that is alive for 10 seconds, then reports unhealthy for the rest of its (hopefully) short existence.
    - For the first 10 seconds, /healthz returns 200 OK ("ok").
    - 10 seconds, /healthz starts returning 500 Internal Server Error.
    - httpGet: Sends a request to path /healthz on port 8080.
    - initialDelaySeconds: 3: Waits 3 seconds after the container starts before running the first check.
    - periodSeconds: 3: Repeats the check every 3 seconds.
    
    **Result:** Once the container runs past 10 seconds, the liveness check fails with a 500 status code, and Kubernetes automatically restarts the container.
  ```python
  $ kubectl get pods liveness-http -w
  NAME            READY   STATUS    RESTARTS   AGE
  liveness-http   1/1     Running   0          3s
  liveness-http   1/1     Running   1 (2s ago)   20s
  liveness-http   1/1     Running   2 (1s ago)   37s
  liveness-http   1/1     Running   3 (1s ago)   55s
  liveness-http   0/1     CrashLoopBackOff   3 (1s ago)   73s
  liveness-http   1/1     Running            4 (23s ago)   95s
  liveness-http   0/1     CrashLoopBackOff   4 (1s ago)    112s
  liveness-http   1/1     Running            5 (49s ago)   2m40s
  liveness-http   0/1     CrashLoopBackOff   5 (2s ago)    2m59s
  liveness-http   1/1     Running            6 (90s ago)   4m27s
  ```
<br>

- **logs-generator:** Outputs lines of logs to stdout uniformly  
  The logs-generator subcommand in the agnhost CLI tool is designed to simulate synthetic API log workloads for Kubernetes end-to-end (e2e) testing and cluster logging verification.
  
  It generates predictable, structured log streams to test how well a cluster handles log ingestion, rotation, aggregation, and throughput management without requiring a live, complex application workload.  
  - Test in Kubernetes:  
    Deploy it as a single-run Pod to test your cluster log collectors (such as Fluentd, Fluent Bit, or Promtail):  
  ```bash
  kubectl run test-logs-gen \
  --image=registry.k8s.io/e2e-test-images/agnhost:2.40 \
  --restart=Never \
  -- logs-generator --log-lines-total 100 --run-duration 10s
  ```
    Where:  
    • --log-lines-total (or -t): Total number of log lines to generate.  
    • --run-duration (or -d): How long the generation should last, specified in a Go duration format (e.g., 1s, 10s, 1m).  
    Each generated line is roughly 100 bytes long and follows this standardized format:  
    2000-12-31T12:59:59Z <id> <method> /api/v1/namespaces/<namespace>/endpoints/<random_string> <random_number>
  
  - Verify the Logs:  
    Once the Pod is running or completed, check the generated output using kubectl logs:
  ```python
  $ kubectl logs test-logs-gen
  I1011 23:17:21.444342       1 logs_generator.go:76] 0 POST /api/v1/namespaces/default/pods/fv72 222
  I1011 23:17:22.444585       1 logs_generator.go:76] 1 GET /api/v1/namespaces/kube-system/pods/qgvc 464
  I1011 23:17:23.445000       1 logs_generator.go:76] 2 GET /api/v1/namespaces/default/pods/pqpz 236
  I1011 23:17:24.444856       1 logs_generator.go:76] 3 GET /api/v1/namespaces/default/pods/27h4 451
  I1011 23:17:25.444927       1 logs_generator.go:76] 4 PUT /api/v1/namespaces/ns/pods/hfbr 225
  I1011 23:17:26.444924       1 logs_generator.go:76] 5 GET /api/v1/namespaces/default/pods/mks9 559
  I1011 23:17:27.444808       1 logs_generator.go:76] 6 POST /api/v1/namespaces/kube-system/pods/ldfk 489
  I1011 23:17:28.444868       1 logs_generator.go:76] 7 GET /api/v1/namespaces/kube-system/pods/t44c 287
  I1011 23:17:29.444880       1 logs_generator.go:76] 8 GET /api/v1/namespaces/default/pods/n76 485
  I1011 23:17:30.444865       1 logs_generator.go:76] 9 GET /api/v1/namespaces/ns/pods/4xcs 225
  ```
<br>

- **mounttest:** Creates files with given permissions and outputs FS type, owner, mode, permissions, contents of files    
  The mounttest subcommand can be used to create files with various permissions, read files, and output file system type, mode, owner, and permissions for any given file.

  The subcommand can accept the following flags:  
  • fs_type: Path to print the FS type for.  
  • file_mode: Path to print the mode bits of.  
  • file_perm: Path to print the perms of.  
  • file_owner: Path to print the owning UID and GID of.  
  • new_file_0644: Path to write to and read from with perm 0644.  
  • new_file_0666: Path to write to and read from with perm 0666.  
  • new_file_0660: Path to write to and read from with perm 0660.  
  • new_file_0777: Path to write to and read from with perm 0777.  
  • file_content: Path to read the file content from.  
  • file_content_in_loop: Path to read the file content in loop from.  
  • retry_time (default: 180): Retry time during the loop.  
  • break_on_expected_content (default: true): Break out of loop on expected content (use with --file_content_in_loop flag only).  

  - Usage:
  ```bash
  kubectl exec test-agnhost -- /agnhost mounttest \
        [--fs_type <path>] [--file_mode <path>] [--file_perm <path>] [--file_owner <path>] \
        [--new_file_0644 <path>] [--new_file_0666 <path>] [--new_file_0660 <path>] [--new_file_0777 <path>] \
        [--file_content <path>] [--file_content_in_loop <path>] \
        [--retry_time <seconds>] [--break_on_expected_content <true_or_false>]
  ```
  - Quick Test:
  ```yaml
  $ kubectl run test-mount --image=registry.k8s.io/e2e-test-images/agnhost:2.58 --restart=Never -- mounttest \
      --new_file_0644=/tmp/test.txt --file_mode=/tmp/test.txt
  $ kubectl logs test-mount
  mode of file "/tmp/test.txt": -rw-r--r--

  $ kubectl run test-mount --image=registry.k8s.io/e2e-test-images/agnhost:2.58 --restart=Never -- mounttest \
     --fs_type="/mnt" --file_mode="/mnt/test.txt" --file_perm="/mnt/test.txt" \
     --file_owner="/mnt/test.txt" --new_file_0777="/mnt/test.txt" --file_content="/mnt/test.txt" \
     --file_content_in_loop="/mnt/test.txt"
  $ kubectl logs test-mount
  mount type of "/mnt": 2035054128
  content of file "/mnt/test.txt": mount-tester new file

  mode of file "/mnt/test.txt": -rwxrwxrwx
  perms of file "/mnt/test.txt": -rwxrwxrwx
  owner UID of "/mnt/test.txt": 0
  owner GID of "/mnt/test.txt": 0
  content of file "/mnt/test.txt": mount-tester new file

  content of file "/mnt/test.txt": mount-tester new file
  ```
<br>

- **net:** Creates webserver or runner for various networking tests  
  The goal of this Go project is to consolidate all low-level network testing "daemons" into one place. In network testing we frequently have need of simple daemons (common/Runner) that perform some "trivial" set of actions on a socket.  
  
  - Create and run test-agnhost pod:  
  ```bash
  $ kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53

  $ kubectl get all -o wide
  NAME               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
  pod/test-agnhost   1/1     Running   0          19s   10.244.0.107   minikube   <none>           <none>
  ```
  - Runners can be executed into two different ways, the command-line or via an HTTP request:
  
  • Start the HTTP background server:  
  ```bash
  $ kubectl exec test-agnhost -- /agnhost net --serve 10.244.0.110:8889
  I1012 04:06:06.056502      16 log.go:245] main.go:119: Running server on 10.244.0.107:8889
  I1012 04:06:17.304524      16 log.go:245] main.go:153: handleRunRequest {POST /run/nat-closewait-server HTTP/1.1 1 1 map[Accept:[*/*] Content-Length:[30] Content-Type:[application/x-www-form-urlencoded] User-Agent:[curl/8.9.0]] 0xc000051780 <nil> 30 [] false 10.244.0.107:8889 map[] map[] <nil> map[] 10.244.0.107:47362 /run/nat-closewait-server <nil> <nil> <nil> 0xc0004006e0 0xc000177380 [] map[]}
  I1012 04:06:17.304606      16 log.go:245] main.go:140: Options: &{LocalAddr:127.0.0.1:9999 PostFinTimeoutSeconds:0}
  ```
  • Trigger a runner using a POST request:
  ```bash
  $ kubectl exec test-agnhost -- curl -v -X POST 10.244.0.107:8889/run/nat-closewait-server   -d '{"LocalAddr":"127.0.0.1:9999"}'
  Note: Unnecessary use of -X or --request, POST is already inferred.
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 10.244.0.107:8889...
  * Connected to 10.244.0.107 (10.244.0.107) port 8889
  > POST /run/nat-closewait-server HTTP/1.1
  > Host: 10.244.0.107:8889
  > User-Agent: curl/8.9.0
  > Accept: */*
  > Content-Length: 30
  > Content-Type: application/x-www-form-urlencoded
  > 
  } [30 bytes data]
  * upload completely sent off: 30 bytes
  100    30    0     0  100    30      0      5  0:00:06  0:00:05  0:00:01     5ok
  output:

  # closewait.go:75: Run &{127.0.0.1:9999 0}
  # closewait.go:88: Server listening on 127.0.0.1:9999
  # closewait.go:96: Client connected
  # closewait.go:105: Server sent FIN, waiting 0 seconds
  # closewait.go:110: Done
  < HTTP/1.1 200 OK
  < Date: Mon, 12 Oct 2026 03:56:43 GMT
  < Content-Length: 225
  < Content-Type: text/plain; charset=utf-8
  < 
  { [225 bytes data]
  100   255  100   225  100    30     44      5  0:00:06  0:00:05  0:00:01    55
  * Connection #0 to host 10.244.0.107 left intact
  ```
  • NAT Close-Wait Client Test:    
  ```bash
  $ kubectl exec test-agnhost -- /agnhost net --runner nat-closewait-client --options '{"RemoteAddr":"127.0.0.1:9999"}'
  I1012 03:56:43.714083     208 log.go:245] main.go:140: Options: &{RemoteAddr:127.0.0.1:9999 TimeoutSeconds:0 PostFinTimeoutSeconds:0 LeakConnection:false}
  output:

  # closewait.go:151: Run &{127.0.0.1:9999 0 0 false}
  # closewait.go:166: Connected to server
  # closewait.go:184: Server has half-closed the connection, waiting 0 seconds
  # closewait.go:195: Done
  ```
<br>

- **netexec:** Creates HTTP(S), UDP, and (optionally) SCTP servers with various endpoints    
  - Create and run test-agnhost pod and client-pod pod:  
  ```python
  $ kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  $ kubectl run client-pod --image=registry.k8s.io/e2e-test-images/agnhost:2.53

  $ kubectl get all -o wide
  NAME               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
  pod/client-pod     1/1     Running   0          8s    10.244.0.116   minikube   <none>           <none>
  pod/test-agnhost   1/1     Running   0          79m   10.244.0.114   minikube   <none>           <none>
  ```
  - Start the netexec Server: listening on port 8080  
  ```bash
  kubectl exec test-agnhost  -- /agnhost netexec --http-port=8080 --udp-port=8080
  ```
  - Test Using Core Endpoints:  
    Once the server is up, you can test connectivity from another pod, a service, or via kubectl exec using its built-in endpoints
    
  • Test Base Connectivity: Returns the request's timestamp.  
  ```python
  $ kubectl exec test-agnhost -- curl http://10.244.0.114:8080/
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  100    60  100    60    0     0  63829      0 --:--:-- --:--:-- --:--:-- 60000
  NOW: 2026-10-12 13:40:53.139295775 +0000 UTC m=+82.214093959
  ```
  • Test Client IP Tracking: Returns the IP address of the client making the request (useful for testing source IP preservation or load balancer behavior).
  ```python
  $ kubectl exec client-pod -- curl http://10.244.0.114:8080/clientip
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
  100    18  100    18    0     0   9278      0 --:--:-- --:--:-- --:--:-- 18000
  10.244.0.116:60506
  ```
  • Test Header Forwarding: Returns the full request header or a specific key (useful for validating Ingress controller proxy headers).
  ```python
  $ kubectl exec client-pod -- curl http://10.244.0.114:8080/header?key=X-Forwarded-For
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  ```
  • Outbound Test (The /dial Endpoint):
  The /dial endpoint allows you to command the agnhost pod to attempt connection to an external or internal host and port. It returns a JSON package detailing successes or errors.
  ```python
  $ kubectl exec client-pod -- curl "http://10.244.0.114:8080/dial?host=kubernetes.default.svc&port=443&protocol=tcp&request=GET"
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                   Dload  Upload   Total   Spent    Left  Speed
    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0unsupported protocol. tcp
  100    26  100    26    0     0  26859      0 --:--:-- --:--:-- --:--:-- 26000
  ```
  • Other testings:
  ```python
  # Return hostname
  $ kubectl exec client-pod -- curl -s "http://10.244.0.114:8080/hostname"
  test-agnhost

  #Execute a shell command
  $ kubectl exec client-pod -- curl -s  "http://10.244.0.114:8080/shell?cmd=whoami"
  {"output":"root\n"}
  ```
<br>

- **nettest:**  Starts a tiny web server for checking networking connectivity
  A tiny web server for checking networking connectivity.   
  - Create and run test-agnhost pod and client-pod pod:  
  ```python
  $ kubectl run test-agnhost --image=registry.k8s.io/e2e-test-images/agnhost:2.53
  $ kubectl run client-pod --image=registry.k8s.io/e2e-test-images/agnhost:2.53

  $ kubectl get all -o wide
  NAME               READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
  pod/client-pod     1/1     Running   0          4s      10.244.0.118   minikube   <none>           <none>
  pod/test-agnhost   1/1     Running   0          4m40s   10.244.0.117   minikube   <none>           <none>
  ```
  - Start the nettest Server: listening on port 8888
  ```bash
  kubectl exec test-agnhost -- /agnhost nettest --port 8888
  ```
  - Testing connectivity:
  ```python
  $ kubectl exec client-pod -- curl -sG "http://10.244.0.117:8888/read"
  {
          "Hostname": "test-agnhost",
          "Sent": null,
          "Received": null,
          "Errors": [
                  "EOF",
                  "test-agnhost: Got request with no source"
          ],
          "Log": [
                  "default/nettest has 0 endpoints ([]), which is less than 8 as expected. Waiting for all endpoints to come up.",
                  "default/nettest has 0 endpoints ([]), which is less than 8 as expected. Waiting for all endpoints to come up.",
                  "default/nettest has 0 endpoints ([]), which is less than 8 as expected. Waiting for all endpoints to come up."
          ],
          "StillContactingPeers": true


  $ kubectl exec test-agnhost -- curl -s http://localhost:8888/status
  running


  $ kubectl exec client-pod -- curl -sG "http://10.244.0.117:8080/write"
  {"Hostname":"test-agnhost"}
  ```
<br>

- **no-snat-test:** Creates the /checknosnat and /whoami endpoints    
  - Create and run test-agnhost pod and client-pod pod:  
  ```python
  $ kubectl run test-agnhost \
      --image=registry.k8s.io/e2e-test-images/agnhost:2.40 \
      --restart=Never \
      --env "POD_IP=10.244.0.121" \
      --env "NODE_IP=192.168.49.2" \
      -- no-snat-test --port 2222
  $ kubectl run client-pod --image=registry.k8s.io/e2e-test-images/agnhost:2.53

  $ kubectl get all -o wide
  NAME               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
  pod/client-pod     1/1     Running   0          7s    10.244.0.122   minikube   <none>           <none>
  pod/test-agnhost   1/1     Running   0          17s   10.244.0.121   minikube   <none>           <none>

  $ minikube ip
  192.168.49.2
  ```
  - Testing
  ```python
  #/whoami: returns the request's IP address.
  $ kubectl exec client-pod -- curl -sG "http://10.244.0.121:2222/whoami"
  10.244.0.122:59792

  #/checknosnat: Verify no-SNAT behavior across target IPs: 
  $ kubectl exec -it client-pod -- curl -s "http://10.244.0.121:2222/checknosnat?ips=10.244.0.121:2222"
  $
  ```
<br>

- **no-snat-test-proxy:** Creates a proxy for the /checknosnat endpoint    
  - Create and run test-agnhost pod and client-pod pod:  
  ```python
  $ kubectl run test-agnhost \
      --image=registry.k8s.io/e2e-test-images/agnhost:2.40 \
      --restart=Never \
      --env "POD_IP=10.244.0.121" \
      --env "NODE_IP=192.168.49.2" \
      -- no-snat-test --port 2222
  $ kubectl run client-pod --image=registry.k8s.io/e2e-test-images/agnhost:2.53
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





  
