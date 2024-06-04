  
# Helm Commands

## Install Helm

    sudo snap install helm --classic

## Install Helm Chart

    helm install <release-name> <chart-name> 

## Detect problems in Helm Chart
   
    helm lint <chart-name> 
    helm install <release-name> --debug --dry-run <chart-name> 

## Upgrade Helm release

    helm upgrade <release-name> <chart-name> 
    helm upgrade <release-name> <chart-name> -n <namespace-name>

## Rollback in Helm 

    helm rollback <release-name> <chart-name> <version> 
    

## Delete Release

    helm delete <release-name> <chart-name> 
   

## Show Helm Chart Directory Structure 

    tree <chart-name> 
    
## Show Installed Helm 

    helm list -a 
   

## Show All Helm Repository 

    helm repo list 
    

## Show Charts under a Repository 

    helm search repo <repo-name>

  
    
