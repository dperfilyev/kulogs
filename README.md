# kulogs

Client-server tool to retrieve logs from multiple servers, both standalone and running as pods across all K8s cluster workers.

## Description

Kulogs is a client/server tool that allows you to collect logs from multiple application instances over a specified time period.
It is particularly useful in Kubernetes environments, where you can retrieve logs for an application’s pod even if that pod has already been removed from the node.

## Installation

Requires Python 3.7+

### kulogsd (server)

You may run kulogsd as stand-alone daemon, as docker container, as K8s daemonset etc.

* Standalone

  Warning: kulogsd cannot daemonize itself — handle this externally.

  ```
  mkdir kulogsd ; cd kulogsd
  sudo apt-get install python3.7-venv
  python3.7 -m venv venv
  . venv/bin/activate
  python3.7 -m pip install --upgrade pip
  pip3 install aiohttp
  # copy kulogsd to ./
  ./kulogsd
  ```
  Verify:

  ```
  curl http://localhost:8080/health
  ```

* Docker
  ```
  docker build -t kulogsd .
  docker save -o kulogsd.img kulogsd
  docker load -i kulogsd.img
  docker run -d --rm -v /var/log/pods:/var/log/pods --network=host -i -e PYTHONUNBUFFERED=1 kulogsd
  ```

* K8s daemonset

  Use docker image for daemonset.

### kulogs (client)

  Since the server is installed as a Kubernetes DaemonSet by default, the client internally executes the following command to discover the server IP addresses:

  ```
   kubectl get pods -n kulogsd -o json
  ```

  Here, -n kulogsd specifies the namespace of the DaemonSet where the kulogsd servers are running.
  If you are using a standalone or Docker installation, you must pass a JSON file containing the server details to the client's STDIN in the same format that kubectl outputs.

**Example server_list.json:** 
```
{
  "items": [
    {
      "metadata": {
        "name": "kulogsd-4l7n6"
      },
      "spec": {
        "nodeName": "worker-01.cluster.k8s"
      },
      "status": {
        "phase": "Running",
        "podIP": "10.11.12.13"
      }
    },
    {
      "metadata": {
        "name": "kulogsd-82stq"
      },
      "spec": {
        "nodeName": "worker-05.cluster.k8s"
      },
      "status": {
        "phase": "Running",
        "podIP": "10.11.15.55"
      }
    }
  ]
}
```
**Usage:** 

  Use -i ( --stdin ) flag in this case:

  ```
   cat server_list.json | kulogs -i -n ns_name -p pod_prefix -s '2026-09-30 10:00:00'  -e '2026-09-30 11:00:00'
  ```
  
