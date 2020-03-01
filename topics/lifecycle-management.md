## lifecycle management
* two types of updates (StrategyType in deployment description)
  * recreate
    * kill all the deployments and recreate them, results in app downtime
  * rolling updates
    * kill and deploy new version gradually
    * RollingUpdateStrategy tells how many pods can be down at a time (RollingUpdateStrategy: 25% max unavailable, 25% max surge)
* edit deployment on the fly
  ```
  kubectl edit deployment <deployment-name>
  ```
* example deployment file
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "4"
    creationTimestamp: "2020-02-26T07:15:11Z"
    generation: 4
    name: frontend
    namespace: default
    resourceVersion: "2513"
    selfLink: /apis/apps/v1/namespaces/default/deployments/frontend
    uid: 9e4a7177-9edc-4595-9161-3dd5a244cb02
  spec:
    minReadySeconds: 20
    progressDeadlineSeconds: 600
    replicas: 4
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        name: webapp
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          name: webapp
      spec:
        containers:
        - image: kodekloud/webapp-color:v2
          imagePullPolicy: IfNotPresent
          name: simple-webapp
          ports:
          - containerPort: 8080
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
    availableReplicas: 4
    conditions:
    - lastTransitionTime: "2020-02-26T07:15:59Z"
      lastUpdateTime: "2020-02-26T07:15:59Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2020-02-26T07:15:11Z"
      lastUpdateTime: "2020-02-26T07:34:39Z"
      message: ReplicaSet "frontend-84bb97f469" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    observedGeneration: 4
    readyReplicas: 4
    replicas: 4
    updatedReplicas: 4
  ```
### commands and agruments
* in pod definition file
  * **command** corresponds to **entrypoint** in Dockerfile (run at startup of container)
  * **args** corresponds to **cmd** in Dockerfile default param passed to the entrypoint)
* example file
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu-sleeper-2
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep"]
      args: ["5000"]  
  ```
### env variables
* types of env variables
  * key-value
  * from configMaps 
    * central place to store env variables for multiple pods
    * creation options:
      * imperative
        ```
        kubectl create configmap <config-name> --from-literal=<key>=<value> (of --from-file=app_config.properties)
        ```
      * declarative
        ```
        kubectl create -f app-config.yaml
        ```
        **app-config.yaml**
        ```
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: app-config-map
        data:
          APP_COLOR: blue
          APP_ENV: prod
        ```
        **use env variable value from configMap**
        ```
         apiVersion: v1
         kind: Pod
         metadata:
           name: ubuntu-sleeper-2
         spec:
           containers:
           - name: ubuntu
             image: ubuntu
             command: ["sleep"]
             args: ["5000"]
             envFrom:
             - configMapRef:
                 name: webapp-config-map
        ```        
  * from secrets
    * creation options:
      * imperative
        ```
        kubectl create secret generic <secret-name> --from-literal=<key>=<value> (of --from-file=app_secrets.properties)
        ```
      * declarative
        ```
        kubectl create -f app-secret.yaml
        echo -n 'password' | base64 => to encode the data
        echo -n 'SefE=ew' | base64 decode => to decode the data
        ```
        **app-secret.yaml**
        ```
        apiVersion: v1
        kind: Secret
        metadata:
          name: app-config-map
        data:
          DB_HOST: sSDsd=
          DB_PASSWORD: SefE=ew
        ```
        **use env variable value from secret**
        ```
         apiVersion: v1
         kind: Pod
         metadata:
           name: ubuntu-sleeper-2
         spec:
           containers:
           - name: ubuntu
             image: ubuntu
             envFrom:
             - secretRef:
                 name: webapp-secrets
        ```
        or 
        ```
        ...
        env: 
          - name: DB_PASSWORD
            valueFrom: 
              secretKeyRef: 
                name: webapp-secrets
                key: DB_PASSWORD
        ```
        or 
        ```
        ...
        volumes:
        - name: app-secret-volume:
            secret:
              secretName: webapp-secret
              
        ...
        cat /opt/app-secret-volumes/DB_PASSWORD => to view the secret in container
        ```
        * best pracitces is to encrypt secret data at rest => [encrypt-data](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
* example file
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu-sleeper-2
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep"]
      args: ["5000"]  
      env:
      - name: HOST_IP
        value: 172.19.234.73
  ```