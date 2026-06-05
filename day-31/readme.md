Okay so yesterday I learned "what is Kubernetes" in theory. Today I actually deployed things.

Created my first Kubernetes Pod using a YAML manifest. This is the mental shift: instead of docker run, you write a YAML file describing what you want, then kubectl apply -f pod.yaml and boom, it exists.



apiVersion: v1

kind: Pod

metadata:

  name: my-pod

spec:

  containers:

  - name: nginx

    image: nginx:latest

That's a Pod. Smallest unit in Kubernetes. Looks like a container but it's actually a wrapper around one or more containers.

Also learned Namespaces (basically virtual clusters within a cluster, organize your resources so teams don't step on each other), created my own, deployed Pods into it.

The workflow is elegant:

Write YAML describing desired state

kubectl apply -f file.yaml → declare it

Kubernetes makes it happen

kubectl get pods → verify it exists

Also explored contexts to work with multiple clusters and switch 

This is why Kubernetes won. Not because it's simple, but because once you understand the mental model, everything else follows from it.

31/90 done.

#90DaysOfDevOps #Kubernetes #Pods #DevOps