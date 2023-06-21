# kubectl-patch-script

This repository provides a bash script that allows you to patch deployments in your Kubernetes environment using kubectl patch. It is particularly useful when encountering issues with spec updates during deployment/installation via Helm, such as tolerations not being applied correctly.
Problem Description

While deploying or installing via Helm using the provided values YAML file, we discovered an issue with the tolerations and potentially the nodeSelector fields. These fields were not being applied as expected. 

As a temporary workaround, we have created a bash script that utilizes kubectl patch to ensure that the NMS component pods are up and running in your environment.

To patch the deployments after the Helm installation, follow these steps:

    Save the following script to a file (e.g., patch-deployments.sh).
    Make the script executable by running chmod +x patch-deployments.sh.
    Execute the script using ./patch-deployments.sh to apply the patch to each deployment.
   
We have incorporated the necessary changes into the bash script to add an annotation to the "apigw" service in the "nms" namespace. 
This sample annotation instructs Azure to create an internal load balancer instead of a public-facing load balancer. 

```
#!/bin/bash
 
DEPLOYMENTS=("apigw" "clickhouse" "core" "dpm" "ingestion" "integrations")
NAMESPACE="nms"
SERVICE_NAME="apigw"
ANNOTATION="service.beta.kubernetes.io/azure-load-balancer-internal=true"
 
for deployment in "${DEPLOYMENTS[@]}"; do
  echo "Patching deployment: $deployment"
 
  kubectl patch deployment "$deployment" -n "$NAMESPACE" -p "$(cat patch.yaml)"
done
 
echo "Adding annotation to service: $SERVICE_NAME"
kubectl annotate service "$SERVICE_NAME" -n "$NAMESPACE" "$ANNOTATION"
```

Script Explanation

    The DEPLOYMENTS array contains the names of the deployments that need to be patched. If necessary, update the array elements to match your deployment names.
    The NAMESPACE variable specifies the namespace where the deployments reside. In our case, it's nms.
    The script iterates over each deployment in the DEPLOYMENTS array and performs the patching operation.
    The kubectl patch command is executed for each deployment, applying the changes specified in the patch.yaml file.

Ensure that you have a file named patch.yaml in the same directory as the script. The patch.yaml file should contain the exact YAML content provided below:

```
spec:
  template:
    spec:
      tolerations:
        - key: key1
          operator: Equal
          value: value1
          effect: NoSchedule
      nodeSelector:
         app: "mgmt"
```

Make sure to customize the contents of patch.yaml according to your environment's desired tolerations and node selector.

Note: Please ensure that you have the necessary permissions to patch the deployments and that the deployments exist in the specified namespace.

We recommend thoroughly testing this solution in your environment to ensure its effectiveness.
