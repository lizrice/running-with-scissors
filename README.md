# Running with Scissors
Here's the source for everything I used in my [Running with Scissors keynote demos at KubeCon + CloudNativeCon](https://youtu.be/ltrV-Qmh3oY), so you can try it all out for yourself and explore how your containerized workloads in Kubernetes are, by default, running as root. See some ways this can easily be abused! 

## Kubernetes cluster
I used a single-node Kubernetes cluster running in a VM on my laptop, that I set up using Vagrant. The Vagrantfile is included in this repo, and it's based on the set-up [described here](https://medium.com/@lizrice/kubernetes-in-vagrant-with-kubeadm-21979ded6c63). 

You could use any Kubernetes cluster where you can get access to host machine / VM, but note that if you have multiple nodes in your cluster, you'll only be able to see containerized processes on the same node where the pod is actually running.

## Docker images
This demo uses a few Docker images. For the sake of ease and to avoid any network dependencies during the live demo I built and tagged them on the VM as follows: 

```
# nginx image with cash utility installed
$ docker build -f Dockerfile.capsh -t nginx:capsh .

# non-root version of the nginx image
$ docker build -f Dockerfile.user -t nginx:user .

# a very naughty container! 
$ docker build -f Dockerfile.naughty -t lizrice:naughty:0.1 .
```

The tags I've used for nginx are for convenience, and are not good practice for anything other than local demos. I haven't pushed these images to Docker Hub. 

## Terminal setup 
When I gave the talk, I split my terminal window into three panes, all of which were ssh'd into the VM. 

I used the top one to run and exec into pods, and the middle one to run commands on the host while pod(s) were running, to see what's happening from the host's perspective. The third pane is constantly watching for running pods: 

```
$ watch kubectl get pods
```

## Pod YAML
I edited the YAML live, but in this repo you'll find files for each of the different scenarios: 

* `pod.yaml` - running the nginx image as root
* `priv.yaml` - add the privileged securityContext
* `non-root.yaml` - try to run the nginx image with runAsNonRoot securityContext
* `non-root-user.yaml` - run the non-root nginx image
* `mount-root.yaml` - run nginx as root, with the host's root filesystem mounted as /host inside the container
* `naughty.yaml` - run the naughty container

Run a pod with `kubectl apply -f pod.yaml`. Stop them with `kubectl delete pod nginx`. Exec into a shell inside the container with `kubectl exec -it nginx /bin/bash`. I aliased `k` for `kubectl` to save some keypresses. 

Aside from `naughty.yaml` these YAML files all use the same name for the nginx pod, so you won't be able to run them simultaneously.

## Observing processes from the host's perspective
I often run a `sleep` command inside a container, and then find any sleep processes from the host's perspective. You might find these useful: 
```
# Find process ID's for any sleep processes
pidof sleep

# Print out user name & ID for any sleep processes
ps -C sleep -o user,uid
```

[![Running with Scissors](https://img.youtube.com/vi/ltrV-Qmh3oY/0.jpg)](https://youtu.be/ltrV-Qmh3oY)


