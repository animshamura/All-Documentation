**What is BGP ?** 

BGP, or Border Gateway Protocol, is a routing protocol used to share network layer reachability information (NLRI) between different routing domains, often referred to as Autonomous Systems (ASs). Each AS is typically managed by a separate administrative organization. The current structure of the Internet is a vast network composed of numerous interconnected ASs. As an external routing protocol, BGP is extensively utilized among Internet Service Providers (ISPs). It facilitates the exchange of reachable routing information between ASs, helps establish paths across ASs, prevents routing loops, and implements routing policies between ASs. The protocol has evolved through three earlier versions: BGP-1, BGP-2, and BGP-3, with the most commonly used version today being BGP-4.

**Why do we use BGP ?**

Interior Gateway Protocols (IGPs) focus on providing routing information within a single network, while BGP was developed to handle routing between different networks, or domains. IGPs, such as RIP, OSPF, and IS-IS, optimize paths within a network but lack the extensive policy control needed for inter-domain routing. BGP, on the other hand, is designed to manage policies and scale for inter-domain routing, serving a different purpose than IGPs.
![BGP & IGP](https://github.com/animshamura/Documentation-Images/blob/main/BGPI.drawio.png?raw=true)    
**BGP Working Procedure**

BGP utilizes TCP for establishing peer relationships, requiring peers to first establish a TCP connection and negotiate parameters through Open messages. Once the peer relationship is established, BGP peers exchange routing tables, with Keepalive messages being sent to maintain connections. BGP updates routing tables incrementally via Update messages upon route changes, and in the event of errors, Notification messages are sent to report the error and terminate the BGP connection.

 **Metal-LB**

MetalLB is a tool for Kubernetes clusters without cloud provider load balancer support. It offers two main features: address allocation and external announcement.

1.  Address Allocation:
    
    -   In cloud environments, a load balancer assigns IP addresses, but MetalLB handles this for bare-metal clusters.
    -   Users provide IP address pools to MetalLB, either leased from a hosting provider or chosen from private address spaces like RFC1918.
    -   MetalLB can manage multiple address pools.
2.  External Announcement:
    
    -   MetalLB ensures that external networks recognize the IP addresses it assigns to services.
    -   In Layer 2 mode, a machine in the cluster announces the IPs using ARP or NDP protocols.
    -   In BGP mode, cluster machines establish peering sessions with nearby routers, enabling load balancing and traffic control.     
        ![MetalLB.drawio.png](https://github.com/animshamura/Documentation-Images/blob/main/MetalLB.drawio.png?raw=true)      
                       
### Installation and Configuration of MetalLB with BGP

#### Prerequisites:

-   A Kubernetes cluster (v1.13 or later)
-   Administrative access to the Kubernetes cluster
-   BGP router(s) in the network

### Step 1 : Install MetalLB

#### Apply the MetalLB Manifest

First, apply the MetalLB manifest to your cluster. This installs MetalLB’s components.

    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.3/manifests/namespace.yaml

### Step 2 : Configure VyOS 

  Configure VyOS form the VyOs CLI. VyOS must have to be installed.

    configure
    set protocols bgp <local-asn> 
    set protocols bgp <local-asn> neighbor <neighbor-ip> remote-as <neighbor-asn>
    commit
    save
    exit

### Step 3 : Configure MetalLB
Configure MetalLB using the configmap.yaml. 

    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        peers:
        - peer-address: <vyos-router-ip>
          peer-asn: <vyos-asn>
        address-pools:
        - name: default
          protocol: bgp
          addresses:
          - <start-ip>-<end-ip>

Create configmap applying kubectl command. 

    kubect apply -f configmap.yaml

### Step 4 : Check BGP peering 
Use the command below to check whether BGP peering has been established. 

    show bgp neighbor

 ### Step 5 : Test MetalLB Loadbalancing 
Deploy a Kubernetes service of type LoadBalancer and ensure that MetalLB assigns an external IP address from the configured pool. Test the load balancing functionality to verify that traffic is being properly distributed to the service endpoints.
  
      ---  
        apiVersion: apps/v1  
        kind: Deployment  
        metadata:  
        name: nginx-deployment  
        spec:  
        replicas: 3  
        selector:  
        matchLabels:  
        app: nginx  
        template:  
        metadata:  
        labels:  
        app: nginx  
        spec:  
        containers:  
        - name: nginx  
        image: nginx:latest  
        ports:  
        - containerPort: 80  
        ---  
        apiVersion: v1  
        kind: Service  
        metadata:  
        name: nginx-service  
        spec:  
        selector:  
        app: nginx  
        ports:  
        - protocol: TCP  
        port: 80  
        targetPort: 80  
        type: LoadBalancer

