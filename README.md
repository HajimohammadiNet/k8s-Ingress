# mergeable-k8s-Ingress

## Using Mergeable Ingress in Kubernetes

Sometimes in Kubernetes, you may need to deploy multiple Ingress resources with a common hostname but different paths. The default behavior of the ingress-controller is to prioritize the rules based on the resource version, causing one of the ingresses to be ignored. In such cases, the simplest and most basic solution is to assign a separate hostname to each ingress, which is not always logical. Another solution is to use a single ingress with different paths, but this method is not suitable for all scenarios. A more professional solution for this is to use Mergeable Ingress.

Mergeable Ingress enables you to spread the Ingress configuration for a common host across multiple Ingress resources. These resources can belong to the same or different namespaces, enabling easier management when using a large number of paths. Here's how to implement it:

## Step 1: Deploy a Master Ingress

Deploy an ingress with your desired hostname (without any rules or paths). It doesn't matter which namespace it is in. The important thing is to add the following configuration in the annotation section:

<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress-master
  annotations:
    nginx.org/mergeable-ingress-type: master
spec:
  rules:
  - host: my.host.com
</pre>


The nginx.org/mergeable-ingress-type: master annotation is used to specify that this is a master ingress. A master will process all configurations at the host level, including the TLS configuration and any annotations that will be applied for the complete host. There can only be one ingress resource on a unique host that contains the master value. Paths cannot be part of the ingress resource.

## Step 2: Deploy Minion Ingresses

Deploy as many ingresses as you want with a common hostname and different paths in different namespaces. Just add the following phrase in the annotation section:


<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress1-minion
  namespace: my-namespace
  annotations:
    nginx.org/mergeable-ingress-type: minion
spec:
  rules:
  - host: my.host.com
    http:
      paths:
      - path: /foo
        pathType: Prefix
        backend:
          service:
            name: my-service1
            port:
              name: http
</pre>         
 
<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress2-minion
  namespace: my-namespace
  annotations:
    nginx.org/mergeable-ingress-type: minion
spec:
  rules:
  - host: my.host.com
    http:
      - path: /bar
        pathType: Prefix
        backend:
          service:
            name: my-service2
            port:
              name: http
</pre>

The nginx.org/mergeable-ingress-type: minion annotation is used to specify that this is a minion ingress. A minion will be used to append different locations to an ingress resource with the master value. TLS configurations are not allowed. Multiple minions can be applied per master as long as they do not have conflicting paths. If a conflicting path is present, then the path defined on the oldest minion will be used.

### Conclusion

Mergeable Ingress is a great way to manage multiple Ingress resources with a common hostname and different paths. By deploying a master ingress and multiple minion ingresses, you can avoid conflicts and manage a large number of paths more efficiently.
