Kubernetes Tutorial Notes:

1.     1. Pods: Kubernetes representation of single container. Will represent a webapp, db, queue or anything other than a networking component of a system
    2. Service: For now, just a component that helps access the pod (pods run inside a Kubernetes vm on the host and cannot be accessed directly, the service has config to map the port on the host to the Kubernetes managed VM), services *select* the pods that they are opening access to via the *selectors* directive on the service end and *labels* directive on the pods end. This essentially represents the networking aspect of a system. Why we need to provide the selector and label is not understood to me yet. One pod can connect to another pod through a service (Digression - the dns for all services inside the same Kubernetes instance is available to all Pods through a shared dns server which exists in a separate namespace and this can be accessed by ‘kubectl get namespace’ and kubectl get all -ns kube-system) (answer for why it is needed - Because pods can scale, for example if we have a nodejs container and need to access it, that can be done with just using port forwarding, but when we have 2 nodejs containers on the same host, how do we forward requests to one or the other, the answer is that service is responsible to know how to forward the request to the different replicas of the same containers)
    3. Replica Sets: Provides a way to define the number of “pods” (container instances) should be available for a given compoenent (such as webapp, db, queue). How this works with systems like elasticsearch/kafka where we group the instances isn’t known yet. This automatically brings up another instance of a pod if one fails
    4. Deployments: Creates a replica-set but with the added functionality of rolling updates on upgrades to a container (in case a new version of a container is available, we will just update the image directive of the template section on a deployment and Kubernetes manages downloading, starting the new version and stopping and shutting down the old version and then transferring the service to access the new version rather than the old). 
        1. Question: How do we validate the correctness of the new version before it is deployed (where does testing fit in here?)
        2. What if there is a dependency between the old containers and the new ones (such as data stored on a container - again in case of a database container) - According to the course a dependency should not exist —> So if we launch a new version of the api backend, then the new version of the front end should not start until the new version of the api is not started.. ? How to implement this? Because the new version of the front end will be able to connect to the old API, this will work only if new versions of all microservices are compatible with last recently deployed and newly deployed versions of their dependencies (in our case if we change the python web socket server and the react app to comply with that change then the react app should be compatible with both the older and newer version of the python web socket server so it can run while the newer version of the websocket is being deployed (or another way is that the newer container doesn’t start until the dependency is deployed and running - react app somehow detects the change on python websocket and then starts only after python web socket has started))
        3. Related to above - Setting up a workflow, how does dependent deployment work ? Example, if we want our webapp to be deployed only after our cache/db/queue container? How does Kubernetes specify order of deployment
    5. VolumeMounts and Volumes: Volume mounts are on the deployment —> template (represents a pod) —> container —> VolumeMount level, they enable sharing of the container (the pod) persistent memory with the host (so that if the pod goes down the data from the pod isn’t lost and is retained on the host). The Volume is the directory on the host (when running locally) or the disk space on the cloud. For host the directive is HostPath —> path and type, where path is absolute value of the directory and type is Directory/DirectoryOrCreate/File/FileOrCreate etc.
    6. Persistence Volume Claims: Instead of providing a Volume on disk for space, we can provide a reference to the volume (the purpose is so we don’t have to change everywhere if the same volume is shared by a lot of pods). 
    7. DaemonSet: Is a Workload just like replicaSet but runs on all nodes (example is log stash/fluentd daemon that needs to run for all nodes so they can forward logs to an aggregator such as ElasticSearch)
    8. StatefulSet: Used on containers such as elastic search and other dbs where the Kubernetes names the service (used to connect to the pod) for these systems in sequence (use? Not known yet)
    9. ConfigMap: key value config that all nodes/pods can access
2. Deploying to cloud:
    1. Create an ec2-instance —> free tier works, create a key pair using ssh-keygen
    2. KOPS: 
        1. Install: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
        2. Deploy: https://kops.sigs.k8s.io/getting_started/aws/
        3. Note - Cluster update fails if Kops secret has not been provided public key (kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub) otherwise everything works as provided on link
        4. Copy/Clone the yamls (pods, services, storage) and change the services node port to LoadBalancer and the storage host paths to EBS volume provisioning and apply

