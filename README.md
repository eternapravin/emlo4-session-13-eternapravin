# emlo4-session-13-eternapravin
Session13 - Kubernetes Introduction

### Contents
  - [Requirements]
  - [Step-by-step-Approach-in-Development]
  - [Observation]
  - [Results]

### Requirements
1. Create a Minikube Cluster Locally or with a t3a.medium Instance on EC2
2. Deploy the CatDog/Dog Breed Classifier model with FastAPI, use FastHTML for the frontend
   - Your Deployment must have 2 Pods for the Model Service
   - It should have a Service and an Ingress
3. What to Submit
    - Github Repo with Deployment YAML Files
    - Instructions to
      - Start MiniKube
      - Create the Deployment
      - Tunnel to the Ingress
      - Access the FastAPI docs page
    - Output of the following in a .md file in your repository
      - kubectl describe <your_deployment>
      - kubectl describe <your_pod>
      - kubectl describe <your_ingress>
      - kubectl top pod
      - kubectl top node
      - kubectl get all -o yaml
     
 ### Step-by-step-Approach-in-Development
  - As per the choices listed in the requirement, I had chosen to create a Minikube cluster on a t3a.medium EC2 cluster
  - As per the requirement, Cat-Dog classifiecation model was used for this assignment
  - Fastapi framework was used for serving at port 8000
 #### Local testing using Docker
   -  Built using the below command
     
     docker build -t fastapi-classifier-k8s -f ./Dockerfile . --no-cache
     
   - Test run using the below command
     
     ` docker run -it  -e PORT=8000 -p 8000:8000  fastapi-classifier-k8s bash `

 #### Deployment on minikube
   - Kubectl for Minikube is run with the command minikube kubectl -- but we can create an alias to just use kubectl by    

     ` alias kubectl="minikube kubectl --" `

   - This assignment I have used Minikube instead of EKS and so it is not a straight forward task and needs some additional configuration, for exposing minikube externally, Minikube should be started with the following flags, this tells the API server to bind to 15.216.172.50 (EC2 public IP) and also instructs Minikube to listen on all network interfaces intead of just local host(127.0.0.1) alone and allows external access for minikube API server, this is limited only to Minikube API server and does not expose the services running inside minikube cluster.
     
   ` minikube start  --apiserver-ips=15.216.172.50   --listen-address=0.0.0.0 `

   - The following changes needed in the classifier-service.yaml file, to enable Nodeport, if we dont do this step, the TYPE of the service will remain a Cluster IP, the clusterIP can be used only locally inside the Minikube cluster, but to expose this to a Public URL, the TYPE of the service need to be changed to Nodeport

     ` type= Nodeport `
     
     ` nodePort =30000 `

The modification of TYPE of the service to ` NodePort ` can be verified by running the command ` kubectl get svc -o wide `

  ![image](https://github.com/user-attachments/assets/e4d9cf1b-638d-4ce0-9539-3a3113a1cb6f)
     
     - Reference of Yaml file with modifications need is as below:

      apiVersion: v1
      kind: Service
      metadata:
          name: classifier-service
          annotations:
              nginx.ingress.kubernetes.io/rewrite-target: /$1
      spec:
          type: NodePort
          selector:
              app: classifier
          ports:
              - protocol: TCP
                port: 80
                targetPort: 8000
                nodePort: 30000

   - First configure your shell to use Minikube's internal Docker daemon instead of your system's Docker daemon, build the image inside the minikube environment and then finally unset the minikune docker variable:  
    
     ` eval $(minikube docker-env) `
     
     ` docker build -t fastapi-classifier-k8s -f ./Dockerfile . --no-cache `
     
     ` eval $(minikube docker-env -u) `
    - Deploy the configurations as per the yaml file as below:
     
     `  kubectl apply -f . `

    - Verification of running of pods, services and ingress can be done by the following command
    
     `  kubectl get all `

    - Once it is confirmed all the pods and services are running fine, modify the exposure settings of the classifier-service, Changes the service type to NodePort, which makes it accessible externally on a randomly assigned port between 30000-32767 on the Minikube node, Specifies the new port for the service inside the cluster but does not change the targetPort inside the pods.
    
    ` kubectl expose service classifier-service --type=NodePort --port=8000 ` 

   -  Open the service through a default web browser

     ` minikube service classifier-service `

   - Open one more terminal and do the tunneling through
     
      ` minikube tunnel `
     
   - To allow external clients (not just localhost) to access the forwarded port use the following command, Without this flag ` --address=0.0.0.0 ` ,the port forwarding only binds to 127.0.0.1 (localhost)
 
    ` kubectl port-forward service/classifier-service 8000:80 --address=0.0.0.0 `

### Results
- Pre-requisites for top commands:

minikube addons enable metrics-server

- kubectl describe deployment.apps/classifer-deployment
![image](https://github.com/user-attachments/assets/4712e6d0-c2ea-475a-bab3-0362019f28f6)

- kubectl describe pod/classifer-deployment-67687b7b4c-pl25j
  ![image](https://github.com/user-attachments/assets/965727c9-cb7c-4118-85ff-28823dbbf60b)
  ![image](https://github.com/user-attachments/assets/8e31d225-f2ff-42ec-ad13-52df592c859c)

- kubectl describe ingress/classifier-ingress
![image](https://github.com/user-attachments/assets/36393e5b-a8a6-459b-bd76-b6150e28166c)


- kubectl top pod
 
  ![image](https://github.com/user-attachments/assets/2833c068-661e-484d-a5e7-086197459d55)

- kubectl top node
 
  ![image](https://github.com/user-attachments/assets/4bc006e8-48f5-49a1-bf89-b66dba99b5c8)

- kubectl get all -o yaml
```yaml  
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2025-02-18T12:50:31Z"
    generateName: classifer-deployment-67687b7b4c-
    labels:
      app: classifier
      pod-template-hash: 67687b7b4c
    name: classifer-deployment-67687b7b4c-pl25j
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: classifer-deployment-67687b7b4c
      uid: d363054c-09db-404c-a361-aadb934d897b
    resourceVersion: "534"
    uid: 83312abf-9acc-4d57-b0ac-7309faa88587
  spec:
    containers:
    - image: fastapi-classifier-k8s:latest
      imagePullPolicy: Never
      name: classifier
      ports:
      - containerPort: 8000
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-lr2jd
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: minikube
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-lr2jd
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: PodReadyToStartContainers
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:31Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:31Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: docker://623337cbf5b808f709f9f372486cc8d7ad6bc757ee09f77f5fed4fb28e5ae555
      image: fastapi-classifier-k8s:latest
      imageID: docker://sha256:b71e57c8dc7da45b39f4d4499684d65952cfb15c9ca61649ea797802b86ee5c9
      lastState: {}
      name: classifier
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2025-02-18T12:50:33Z"
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-lr2jd
        readOnly: true
        recursiveReadOnly: Disabled
    hostIP: 192.168.49.2
    hostIPs:
    - ip: 192.168.49.2
    phase: Running
    podIP: 10.244.0.4
    podIPs:
    - ip: 10.244.0.4
    qosClass: BestEffort
    startTime: "2025-02-18T12:50:31Z"
- apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: "2025-02-18T12:50:31Z"
    generateName: classifer-deployment-67687b7b4c-
    labels:
      app: classifier
      pod-template-hash: 67687b7b4c
    name: classifer-deployment-67687b7b4c-tfrvt
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: ReplicaSet
      name: classifer-deployment-67687b7b4c
      uid: d363054c-09db-404c-a361-aadb934d897b
    resourceVersion: "539"
    uid: 650e4750-c653-4a29-bf0a-44a801b23dae
  spec:
    containers:
    - image: fastapi-classifier-k8s:latest
      imagePullPolicy: Never
      name: classifier
      ports:
      - containerPort: 8000
        protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-m89dc
        readOnly: true
    dnsPolicy: ClusterFirst
    enableServiceLinks: true
    nodeName: minikube
    preemptionPolicy: PreemptLowerPriority
    priority: 0
    restartPolicy: Always
    schedulerName: default-scheduler
    securityContext: {}
    serviceAccount: default
    serviceAccountName: default
    terminationGracePeriodSeconds: 30
    tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
    volumes:
    - name: kube-api-access-m89dc
      projected:
        defaultMode: 420
        sources:
        - serviceAccountToken:
            expirationSeconds: 3607
            path: token
        - configMap:
            items:
            - key: ca.crt
              path: ca.crt
            name: kube-root-ca.crt
        - downwardAPI:
            items:
            - fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
              path: namespace
  status:
    conditions:
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: PodReadyToStartContainers
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:31Z"
      status: "True"
      type: Initialized
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: Ready
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:34Z"
      status: "True"
      type: ContainersReady
    - lastProbeTime: null
      lastTransitionTime: "2025-02-18T12:50:31Z"
      status: "True"
      type: PodScheduled
    containerStatuses:
    - containerID: docker://c09a8207b80afeb6ff8ce110e1c9c6ce86b09b583ad7bd76cc8526d9c07cd25b
      image: fastapi-classifier-k8s:latest
      imageID: docker://sha256:b71e57c8dc7da45b39f4d4499684d65952cfb15c9ca61649ea797802b86ee5c9
      lastState: {}
      name: classifier
      ready: true
      restartCount: 0
      started: true
      state:
        running:
          startedAt: "2025-02-18T12:50:33Z"
      volumeMounts:
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
        name: kube-api-access-m89dc
        readOnly: true
        recursiveReadOnly: Disabled
    hostIP: 192.168.49.2
    hostIPs:
    - ip: 192.168.49.2
    phase: Running
    podIP: 10.244.0.3
    podIPs:
    - ip: 10.244.0.3
    qosClass: BestEffort
    startTime: "2025-02-18T12:50:31Z"
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"nginx.ingress.kubernetes.io/rewrite-target":"/$1"},"name":"classifier-service","namespace":"default"},"spec":{"ports":[{"nodePort":30000,"port":80,"protocol":"TCP","targetPort":8000}],"selector":{"app":"classifier"},"type":"NodePort"}}
      nginx.ingress.kubernetes.io/rewrite-target: /$1
    creationTimestamp: "2025-02-18T12:50:31Z"
    name: classifier-service
    namespace: default
    resourceVersion: "515"
    uid: 9a5b4cf4-344a-47e0-80bd-ae6759bb8e55
  spec:
    clusterIP: 10.101.201.136
    clusterIPs:
    - 10.101.201.136
    externalTrafficPolicy: Cluster
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - nodePort: 30000
      port: 80
      protocol: TCP
      targetPort: 8000
    selector:
      app: classifier
    sessionAffinity: None
    type: NodePort
  status:
    loadBalancer: {}
- apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: "2025-02-18T12:48:28Z"
    labels:
      component: apiserver
      provider: kubernetes
    name: kubernetes
    namespace: default
    resourceVersion: "227"
    uid: 888e5e31-8183-4de6-a6d4-a8f2b3e8019b
  spec:
    clusterIP: 10.96.0.1
    clusterIPs:
    - 10.96.0.1
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: 8443
    sessionAffinity: None
    type: ClusterIP
  status:
    loadBalancer: {}
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"classifier"},"name":"classifer-deployment","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"classifier"}},"template":{"metadata":{"labels":{"app":"classifier"}},"spec":{"containers":[{"image":"fastapi-classifier-k8s:latest","imagePullPolicy":"Never","name":"classifier","ports":[{"containerPort":8000}]}]}}}}
    creationTimestamp: "2025-02-18T12:50:30Z"
    generation: 1
    labels:
      app: classifier
    name: classifer-deployment
    namespace: default
    resourceVersion: "543"
    uid: 1fd94be0-35c5-4169-b7e7-af2395546d78
  spec:
    progressDeadlineSeconds: 600
    replicas: 2
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: classifier
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: classifier
      spec:
        containers:
        - image: fastapi-classifier-k8s:latest
          imagePullPolicy: Never
          name: classifier
          ports:
          - containerPort: 8000
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 2
    conditions:
    - lastTransitionTime: "2025-02-18T12:50:34Z"
      lastUpdateTime: "2025-02-18T12:50:34Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2025-02-18T12:50:30Z"
      lastUpdateTime: "2025-02-18T12:50:34Z"
      message: ReplicaSet "classifer-deployment-67687b7b4c" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    observedGeneration: 1
    readyReplicas: 2
    replicas: 2
    updatedReplicas: 2
- apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    annotations:
      deployment.kubernetes.io/desired-replicas: "2"
      deployment.kubernetes.io/max-replicas: "3"
      deployment.kubernetes.io/revision: "1"
    creationTimestamp: "2025-02-18T12:50:30Z"
    generation: 1
    labels:
      app: classifier
      pod-template-hash: 67687b7b4c
    name: classifer-deployment-67687b7b4c
    namespace: default
    ownerReferences:
    - apiVersion: apps/v1
      blockOwnerDeletion: true
      controller: true
      kind: Deployment
      name: classifer-deployment
      uid: 1fd94be0-35c5-4169-b7e7-af2395546d78
    resourceVersion: "540"
    uid: d363054c-09db-404c-a361-aadb934d897b
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: classifier
        pod-template-hash: 67687b7b4c
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: classifier
          pod-template-hash: 67687b7b4c
      spec:
        containers:
        - image: fastapi-classifier-k8s:latest
          imagePullPolicy: Never
          name: classifier
          ports:
          - containerPort: 8000
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 2
    fullyLabeledReplicas: 2
    observedGeneration: 1
    readyReplicas: 2
    replicas: 2
kind: List
metadata:
  resourceVersion: ""
```
- Inference window and sample results on EC2 instance

 ![image](https://github.com/user-attachments/assets/0523c446-0c20-45e7-bbca-220ee6678e42)

![image](https://github.com/user-attachments/assets/a82848fb-c7dc-45da-b99a-aa6c95117d2f)



 ### Observation

