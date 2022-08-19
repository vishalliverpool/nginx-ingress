# nginx-ingress

#Step 1

Creating and Connecting to a Kubernetes Cluster
Create a Kubernetes cluster through the Linode Kubernetes Engine (LKE) using either the Cloud Manager, the Linode API, or Terraform:

Setup your local environment by installing Helm 3 and kubectl on your computer (or whichever system you intend to use to manage your Kubernetes Cluster).

Configure kubectl to use the new Kubernetes cluster by downloading the kubeconfig YAML file and adding it to kubectl. See the instructions within the Download Your kubeconfig guide.

Creating a Sample Application
In order to be able to confirm that the NGINX Ingress you create is working as expected in later steps, deploy a sample application which will confirm the connection to your backend Services. Our application will be built from an official NGINX Docker image, though this application can be replaced with any you prefer.

#Step 2

Configure and Create the Deployment
Wherever you’ve installed kubectl, create two yaml manifest files using a text editor of your choice. These manifests will be responsible for creating our Deployments their associated Services. The Deployments will be called hello-one and hello-two respectively, and will be replicated three times each:

Using a text editor, create a new file named hello-one.yaml with the contents of the example file.

Create a second Service and Deployment manifest file named hello-two.yaml with the contents of the example file.
Use kubectl to create the Services and Deployments for your example applications.

#Step 3
kubectl create -f hello-one.yaml
kubectl create -f hello-two.yaml
You should see a similar output:

service/hello-one created
deployment.apps/hello-one created
service/hello-two created
deployment.apps/hello-two created
Verify that the Services are running.

kubectl get svc
You should see a similar output:

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hello-one    ClusterIP   10.128.94.166    <none>        80/TCP    6s
hello-two    ClusterIP   10.128.102.187   <none>        80/TCP    6s
kubernetes   ClusterIP   10.128.0.1       <none>        443/TCP   18m

#Step 4

Install the NGINX Ingress Controller
In this section you will use Helm to install the NGINX Ingress Controller on your Kubernetes Cluster. Installing the NGINX Ingress Controller will create Linode NodeBalancers that your cluster can make use of to load balance traffic to your example application.

Add the following Helm ingress-nginx repository to your Helm repos.

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
Update your Helm repositories.

helm repo update
Install the NGINX Ingress Controller. This installation will result in a Linode NodeBalancer being created.

helm install ingress-nginx ingress-nginx/ingress-nginx
You will see a similar output:

NAME: ingress-nginx
LAST DEPLOYED: Fri Apr  9 21:29:47 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w ingress-nginx-controller'
...
Create a Subdomain DNS Entries for your Example Applications
Now that Linode NodeBalancers have been created by the NGINX Ingress Controller, you can point a subdomain DNS entries to the NodeBalancer’s public IPv4 address. Since this guide uses two example applications, it will require two subdomain entries.

Access your NodeBalancer’s assigned external IP address.

kubectl --namespace default get services -o wide -w ingress-nginx-controller
The command will return a similar output:

NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                      AGE     SELECTOR
my-ingress-nginx-controller   LoadBalancer   10.128.169.60   192.0.2.0   80:32401/TCP,443:30830/TCP   7h51m   app.kubernetes.io/instance=cingress-nginx,app.kubernetes.io/name=ingress-nginx
Copy the IP address of the EXTERNAL IP field and navigate to Linode’s DNS manager and add two “A” records for the blog and shop subdomains. Ensure you point each record to the NodeBalancer’s IPv4 address you retrieved in the previous step.

#Step 5

Configuring the Ingress Controller
Once your Ingress Controller is installed and DNS records have been created pointing to your NodeBalancer, you need to create a manifest file to create a new Ingress resource. This resource will define how traffic coming from the LoadBalancer service we deployed earlier is handled. In this case, NGINX will accept these connections over port 80, diverting traffic to both of our services via their hostname or domain names:
Create an Ingress resource manifest file named my-new-ingress.yaml.
kubectl create -f my-new-ingress.yaml

Once the Ingress has been created, try accessing your subdomains from a browser. As a reminder, this example uses blog.example.com and shop.example.com. Each time you navigate to the page, you’ll see one of three different instances of the replicated server as the active node is rotated. While the application has been deployed to the same cluster, at no point will blog.example.com display the same three hostnames as shop.example.com, as all requests are being routed correctly.
