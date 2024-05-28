**Step 1 : Update APT**

    sudo apt update
   **Step 2 : Install Bird-BGP**
   
    sudo apt install  bird -y
   **Step 3 : Start and Enable Bird Service**
   
    sudo systemctl start bird
    sudo systemctl enable bird
   
   **Step 4 : Configure Bird-BGP**
  
    sudo vi /etc/bird/bird.conf
   Edit configuration file and set peer information such as ASN, IP address and router ID for creating BGP session. Mention **router id** which is basically **IP address of the router** and edit **protocol BGP** section like below. 

    description "My BGP Peering";  # Description of the BGP session
    local as 24342;  # Local AS number
    neighbor 172.17.17.16 as 24342;  # Neighbor IP and AS number
    import all;  # Import all routes received from the neighbor
    export all;  # Export all locally defined routes to the neighbor
**Step 5 : Restart Bird-BGP**

    sudo systemctl restart bird
**Step 6 : Check Bird Set up**
  

    sudo bird --version

**Step 7 : Check Protocols**

    sudo birdc
    show protocols

