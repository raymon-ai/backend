# License
See [LICENSE.md](LICENSE.md)

# Deployment

## 1. Notes before deploying

First of all, the Raymon web UI needs to be accessed over a secured connection (TLS / HTTPS). The API does work over unsecured HTTP, but you should really use HTTPS too. This means you need to determine which URL you want the Raymon platform te be accessible (for example https://ui.raymon.internal.company.com and https://api.raymon.internal.company.com) and acquire a TLS certificate for it (which you might already have). When deploying to a cloud provider (AWS, GCP, Azure, ...), there should be an easy way to get managed certificates (see further below). 
If you are not deploying on a common cloud provider, you can deploy your own TLS endpoint alongside Raymon. See [TLS-ingress.md](TLS-ingress.md).

Secondly, we need to enable the URL your UI is accessed at on our backend authentication provider or you won't be able to log in. You can do this automatically after deployment with your Raymon client ID and client secret. This does **not** mean that this URL does needs to be publicly accessable, you can still only make your Raymon deployment completely air gapped.

## 2. Deploying on Kubernetes
The recommended way to deploy Raymon is to run it on Kubernetes (k8s). 

### 2.1 Google GKE
Instructions coming soon.

### 2.2 Amazon EKS
Instructions coming soon.

### 2.3 Azure AKS
Instructions coming soon.

### 2.4 Other Kubernetes
Eh-oh, uncharted territory. Please [contact us](mailto:hello@raymon.ai) and we'll help you out.

## 3. Alternative Deployment options
To get started, copy the `stack.template` file and name the copy `docker-compose.yml`. You'll have to make a few changes.

### 3.1 Run locally using Docker
You can deploy Raymon to your local machine with Docker (swarm mode) for quick testing.

1. Make sure Docker is in [Swarm mode](https://docs.docker.com/engine/swarm/), if not run `docker swarm init`
2. Replace the value of `RAYMON_MGMT_API_CLIENT_ID` in `stack-local.yml` with your Client ID
3. Create docker secrets:
    - `echo "<your client secret>" | docker secret create raymon-mgmt-api-secret -`
    - `echo "your password" | docker secret create raymondb-password -`
4. Run `docker stack deploy raymon`

### 3.2 Deploy on remote Docker
Same as 3.1, but deploy an extra Ingress controller / load balancer that serves as TLS endpoint as described in [TLS-ingress.md](TLS-ingress.md).

### 3.3 Amazon ECS

1. Make sure Docker is configured to [use ECS](https://docs.docker.com/cloud/ecs-integration/).
2. Create docker secrets:
    - `echo "<your client secret>" | docker secret create raymon-mgmt-api-secret -`
    - `echo "your password" | docker secret create raymondb-password -`
3. 
    - Replace the value of `RAYMON_MGMT_API_CLIENT_ID` in `docker-compose.yml` with your Client ID
    - Add the secret names to the secret section, as follows, but be sure to check the exact secret names with `docker secrets ls`. 
    ```yaml
    secrets:
        raymondb-password:
            external: true
            name: "arn:aws:secretsmanager:eu-west-3:249213429262:secret:raymondb-password-Ae62Xg"
        raymon-mgmt-api-secret:
            external: true
            name: "arn:aws:secretsmanager:eu-west-3:249213429262:secret:raymon-mgmt-api-secret-KVghvj"
    ```

4. Run `docker compose up`
5. After deployment is complete, we still need to set up a TLS endpoint. For this, first run `docker compose ps` and copy the URL of the stack, which can be found under PORTS. In this case, we need to copy `backe-LoadB-153PQDGDYBHBX-381d3d4604337ae2.elb.eu-central-1.amazonaws.com`
```bash
> docker compose -f stack-local.yml ps
NAME                                            SERVICE             STATUS              PORTS
task/backend/0a2ca67099e44fc9a4feeb31c94f0a11   ui                  Running             backe-LoadB-153PQDGDYBHBX-381d3d4604337ae2.elb.eu-central-1.amazonaws.com:80->80/tcp
task/backend/224a3fe345444f76a0cacce2cc7da175   db                  Running             backe-LoadB-153PQDGDYBHBX-381d3d4604337ae2.elb.eu-central-1.amazonaws.com:5432->5432/tcp
task/backend/5682356515bb41e89580d95ac506ce7d   mapper              Running
...
```
6. Forward your subdomain like `raymon.dev.company.com` to that URL, by setting a CNAME record.
7. Go to the AWS Management Console > EC2 > Load Balancers. In the list of running instances theres should be one created by deploying Raymon. Its name will have the same prefix as the URL you copied in step 5. Select it and go to Listeners
8. Add 2 Listeners, both using the TLS protocol: one listening on port 443 that forwards to the UI service, and one listening on port 5001 that forwards to the API service. The wizard will allow you to select or create a managed TLS certificate (for free) tied to your domain.

The API will now be available on `https://raymon.dev.company.com:5001` and the UI should be at `https://raymon.dev.company.com`. Be sure to activate your UI first!

## 3. Post Deployment
We still need to enable your UI deployment on our Auth backend. For now, you need to [mail us](mailto:hello@raymon.ai) the URL of your UI, and we'll enable it.

