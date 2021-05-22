https://istio.io/latest/about/service-mesh/

## Download Istio

    wget https://github.com/istio/istio/releases/download/1.5.2/istio-1.5.2-linux.tar.gz
    cd istio-1.5.2
    tar -zxvf istio-1.5.2-linux.tar.gz
    export PATH=$PWD/bin:$PATH


## Checkout available Istio Profile and their differences
   
     istioctl profile list
   
     istioctl profile dump demo >demo.txt
     istioctl profile dump default >default.txt
     vim -d demo.txt default.txt

## Install istio with demo profile 
- Make sure your server has enough CPU/Mem resources.
- Demo profile makes sure that it installs grafana, prometheus Kiali and other services

        istioctl manifest apply --set profile=demo
     
## Enable Sidecar injection

Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your application later:

    kubectl label namespace default istio-injection=enabled

# Deploy the sample application 

    kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    kubectl get services
    kubectl get deploy
    
## Check the deployments 

Wait for 5 minutes and then Make sure it is working fine 

    kubectl get po
    kubectl get ep    
    kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
    
This should give you an output containing "<title>Simple Bookstore App</title>"

      kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
      kubectl get gateway
      
Change the service type from LoadBalancer to NodePort
      
      kubectl get svc istio-ingressgateway -n istio-system
      kubectl edit svc istio-ingressgateway -n istio-system
     
Then Find the nodeport corresponding to http2 
     
      kubectl get svc istio-ingressgateway -n istio-system
      kubectl get ep istio-ingressgateway -n istio-system
      kubectl describe svc istio-ingressgateway -n istio-system |grep -i http2 |grep -i nodeport
      
The command above will show the value of the nodeport assigned  

 - Open the web browser and type: 
 
 - http://public-ip:nodeport-value/productpage 
    
 - You shall see a book-info website 
 
 - Access individual tools (Prometheus, Grafana, Kiali, Jaeger) on separate browser-tabs. If required change the service type from ClusterIP to NodePort for below services and access them with random node port assigned
```sh
kubectl edit svc prometheus -n istio-system
kubectl edit svc grafana -n istio-system
kubectl edit svc Kiali -n istio-system
kubectl edit svc jaeger-query -n istio-system
```
