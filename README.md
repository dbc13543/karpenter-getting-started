# karpenter-getting-started
Set of scripts to provision a simple EKS cluster with Karpenter as your node autoscaler

## Prequisites
1. AWS CLI must be installed and configured
2. eksctl must be installed

## Contents
    build.ps1              Used to build the cluster and install karpenter
    cluster.config         The config file passed to eksctl.  This is generated by build.ps1 from the template and should not be edited.
    cluster.template       Template used to create the cluster.config for eksctl.  Feel free to edit.
    destroy.ps1            Tears down the cluster after you're finished with it
    envsubst.exe           Used to substitute values into the config templates
    Get-Pods-By-Node.ps1   Simple PowerShell script that lists all the pods running in a specified namespace by node.
    karpenter.cftemplate   Cloud Formation template supplied by AWS for deploying Karpenter
    nodepool.template      Template for a nodepool definitions passed to kubectl.  Feel free to edit.
    nodepool.yaml          The nodepool manifest passed to kubectl.  This is generated by build.ps1 from the template and should not be edited.
    php-apache.yaml        A sample deployment you can use for testing

## Instructions

1. Start by running 

        build.ps1
        
    It will prompt you for your user id.  Enter only the id, without any spaces, or @domain.com.  The process will take around 30 minutes and while there may be a couple warnings there should not be any fatal errors.

2. Check how many nodes are part of your cluster by typing 

        kubectl get no
        
    You'll notice there is one node running at all times.  Karpenter requires at least one node be created first, to host kube-system components and karpenter itself.  This first node is NOT managed by Karpenter but is instead just a regular EKS Managed Node Pool with a single node.

4. Start your test deployment by typing

        kubectl apply -f php-apache.yaml

5. Check how and where your deployment is running by typing 

    Get-Pods-By-Node.ps1 -n testing
    
    You'll notice only a single pod running in the "testing" namespace on your single node.

6. Scale up your deployment to add more replicas by typing 

        kubectl scale deploy php-apache -n testing --replicas=9
        
    Have more nodes been added to your cluster?  To check, try running "kubectl get no" again.  If yes, check how the pods have been distributed by running Get-Pods-By-Node.ps1 again.  If not, try scaling up again by moving to the next step.  Any new ndoes that appear have been provisioned by Karpenter.

7. Scale up your deployment again to see what happens.

        kubectl scale deploy php-apache -n testing --replicas=15
        
    Have more nodes been added?  If you run "kubectl get no" quickly you may be able to see them being provisioned.

8. Go to the AWS console and find your EKS cluster.  Click on it to see properties then click the "Compute" tab to see any nodes assigned to it.  Note that the first node created will be part of a managed node group, but any nodes that aren't in a node group were scaled out by Karpenter.

9. Click on one of the Karpenter nodes, then click on Labels, and find the label karpenter.sh/capacity-type.  This will read either "on demand" or "spot".  Note that the instance type can be any one of many.  The allowable instance types are defined in nodepool.yaml.

<img src="https://i.imgur.com/74AtlHK.png" width=500>

10. Scale your deployment back down and see what happens with the nodes by typing

        kubectl scale deploy php-apache -n testing --replicas=1
        
    After a few minutes you should start seeing the Karpenter.

11. When you've finished your testing, deprovision the resources by running destroy.ps1.  This will take several minutes.
