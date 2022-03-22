# issues-and-fix
- This repo contains all the issues and fix(if could get it) that I faced during my day to day SWE tasks.
- You all are most welcome to contribute in this repo, contribute by opening a PR/issue, thanks!


## issue#1
- What?   
    - `kind-control-plane` status `NotReady`
- Summary?
    - created a k8s cluster in kind using `kind create cluster`
    - then `kubectl get nodes` shows node is NotReady (it stays same for long period of time):
        - ```
            NAME                 STATUS     ROLES                  AGE    VERSION
            kind-control-plane   NotReady   control-plane,master   119s   v1.20.2
          ```
    - then tried to debug, first saw all the workloads in `kube-system` namespace by `kubectl get all -n kube-system`, it shows:
        - ```
            NAME                                             READY   STATUS             RESTARTS   AGE
            pod/coredns-74ff55c5b-2h95l                      0/1     Pending            0          113s
            pod/coredns-74ff55c5b-x4s6n                      0/1     Pending            0          113s
            pod/etcd-kind-control-plane                      1/1     Running            0          2m1s
            pod/kindnet-4cfvt                                1/1     Running            0          113s
            pod/kube-apiserver-kind-control-plane            1/1     Running            0          2m1s
            pod/kube-controller-manager-kind-control-plane   1/1     Running            0          2m1s
            pod/kube-proxy-h5bkg                             0/1     CrashLoopBackOff   3          113s
            pod/kube-scheduler-kind-control-plane            1/1     Running            0          2m1s
            
            NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
            service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   2m8s
            
            NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
            daemonset.apps/kindnet      1         1         1       1            1           <none>                   2m5s
            daemonset.apps/kube-proxy   1         1         0       1            0           kubernetes.io/os=linux   2m8s
            
            NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
            deployment.apps/coredns   0/2     2            0           2m8s
            
            NAME                                DESIRED   CURRENT   READY   AGE
            replicaset.apps/coredns-74ff55c5b   2         2         0       113s

          ```
    - then saw the `kube-proxy` pod's logs by `kubectl logs pod/kube-proxy-h5bkg -n kube-system`, it shows:
        - ```
            ......
            ......
            I0322 03:50:13.098798       1 server_others.go:185] Using iptables Proxier.
            I0322 03:50:13.100071       1 server.go:650] Version: v1.20.2
            I0322 03:50:13.104418       1 conntrack.go:100] Set sysctl 'net/netfilter/nf_conntrack_max' to 131072
            F0322 03:50:13.104476       1 server.go:495] open /proc/sys/net/netfilter/nf_conntrack_max: permission denied
          ```
    - then after some googling found that:
        > Need to manually set the parameter with `sudo sysctl net/netfilter/nf_conntrack_max=131072` before creating the Kind cluster. 

- Solution?
    - I deleted this cluster and set the parameter by `sudo sysctl net/netfilter/nf_conntrack_max=131072` and then created cluster again and it works :) 

## issue#2
- What?
    - K8s namespace stuck in `Terminating` phase when tried to delete it.
- Summary?
    - I tried to delete a namespace by `kubectl delete ns <namespace_name>`
    - It was stuck in `Terminating` phase for long period of time and the `ns` was not getting deleted
    - then run `kubectl get all -n <namespace_name>` this command to see whether anything was there in that `ns`, but found nothing
    - after some googling & talking to my colleague pulak it seems he also faced the same issue before
    - pulak tell me to run the command `kubectl get apiservice`, it shows some of the apiservice's `Availability` were `false` and those were making problem
    - then deleted those apiservices manually and the `ns` got deleted then :) 

- Solution?
    - it can be occurred because of various reasons but I'm sharing what helps me here.
    - run `kubectl get apiservice`, if you see any apiservice's Available is `false` then delete them manually
    - now your namespace should be deleted fine :) 
    - reference: [see this doc](https://cert-manager.io/v1.2-docs/installation/uninstall/kubernetes/#namespace-stuck-in-terminating-state)
    
