# Step by step of infrastructure creation

Following tools are required to create environment:
- terraform
- kubectl
- helm & helm secrets plugin
- helmfile
- aws cli
- aws-iam-authenticator
- makefile

1. There is Dockerfile for building container. It's controller to implement all commands related to the current tasks.

```shell
$ docker build . -t controller:0.1
```
Next you can run container with the following command

```shell
$ docker run  -d --privileged \
            -v ~/.aws/:/root/.aws \
            -v ${PWD}:/project \
            -p 9090:9090 \
            -p 9093:9093 \
            -p 3000:3000 \
            -e EMAIL_NOTIFICATION='email_to_send_alerts@gmail.com' \
            -e WEBHOOK_URL='http://webhook_alerts.com/test' \
            -it controller:0.1 bash
```

To shell inside unto running container use following command
```shell
$ docker exec -it container_id bash
```

2. Next we will proceed to EKS installation.
From directory /project/terraform-eks-deploy, need to initialise terraform and then install EKS as follow:

```shell
$ terraform init
```
```shell
$ terraform plan
```
```shell
$ terraform apply
```

It takes about 15-20 minutes for completed installation.

2. From directory /project, there are make commands which help to manage applications.

```shell
$ make help
```

You will see description of each command
```
init_workspace                  - initiate workspace by importing kubeconfig, gpg key and namespace creation. -
 deploy_apps                    - deploying process of product application grafana, prometheus and postgresql database. -
 all_portforward                - run port forwarding on grafana, alertmanager and prometheus simultaneously. -
 prometheus_portforward         - prometheus port frowarding to any ip address on port 9090. -
 alertmanager_portforward       - alertmanager port frowarding to any ip address on port 9093. -
 grafana_portforward            - grafana port frowarding to any ip address on port 3000. -
 run_app_load_on_app            - run "dd if=/dev/zero of=/dev/null" to perform cpu loading. -
```

3. Now we need to initialise our workspace to be able to work with Kubernetes cluster

```shell
$ make init_workspace
```
- You will be asked to enter zone and cluster name to get kubeconfig.
- Also gpg key will be imported to decrypt secrets.
- Project and monitoring namespace will be created in k8s cluster

During script execution you will be asked to provide password for importing gpg key. 

```
secret
```

4. As k8s cluster and workcpace are ready we can deploy applications to the clsuter

Before deploying we need to reload gpg agent to be able to decrypt the secrets.

```shell
$ GPG_TTY=$(tty)
$ export GPG_TTY
```
Now deploy applications

```shell
$ make deploy_apps
```
During application deploying you will be asked to provide password for secret decryption. 

```
secret
```

5. Now it's to for monitoring
We are going to use Grafana, Prometheus and Alertmanager. There are separate comands to make portforwaring for each of them but you can do it with only one of it.

```shell
$ make all_portforward
```

During command implementation you will find Grafana credentials for UI.

```
127.0.0.1:3000 - Grafana
127.0.0.1:9090 - Prometheus
127.0.0.1:9093 - Alertmanager
```

6. Now we can initiate the load to the applications.

```shell
make run_app_load_on_app
```

In few minutes you should get alet notification to you email.

7. After that we can destroy k8s cluster

```shell
terraform destroy
```
