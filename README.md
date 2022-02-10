# helm-oci-django-atp
[![License: UPL](https://img.shields.io/badge/license-UPL-green)](https://img.shields.io/badge/license-UPL-green) [![Quality gate](https://sonarcloud.io/api/project_badges/quality_gate?project=oracle-devrel_terraform-oci-arch-ci-cd)](https://sonarcloud.io/dashboard?id=oracle-devrel_terraform-oci-arch-ci-cd)


## Introduction

This Helm Chart will deploy an Oracle Cloud Infrastructure (OCI) Autonomous Database with an API server built with the Django framework. The API server will provide access to a user-profile database on the Autonomous Database to perfrom create, read, update and delete (CRUD) operations on the database. The API server will be accessible via a loadbalancer provisioned with an external IP address.

To access the API server use /api extenison to the IP address of the loadbalancer.

**Example**  
http://<ip address of laodbalancer>/api/



This Helm chart relies on the OCI Service Operator for Kubernetes (OSOK), and it is a pre-requisite to have OSOK deployed within the cluster to use this Helm chart.


## Pre-requisites

- A Kuberntes cluster deployed in OCI 
- [OCI Service Operator for Kuberntes (OSOK) deployed in the cluster](https://github.com/oracle/oci-service-operator/blob/main/docs/installation.md)
- [kubectl installed and using the context for the Kubernetes cluster where the ATP resource will be deployed](https://kubernetes.io/docs/tasks/tools/)
- [Helm installed](https://helm.sh/docs/intro/install/)
- [Docker installed](https://docs.docker.com/engine/install/)


##  Getting Started

**1. Clone or download the contents of this repo** 
     
     git clone https://github.com/chiphwang1/helm-oci-django-atp.git
**2. Change to the directory that holds the Helm Chart** 

      cd ./helm-oci-django-atp.git

**3. Populate the values.yaml file with the required information **


**4. Create the namespace where the ATP resource will be deployed**

     kubectl create ns <namespace name>

**5. Install the Helm chart. Best practice is to assign the databse password and wallet password during the installation of the Helm chart instead of adding it to the values.yam file.**

     helm -n <namespace name> install \
     --set dbPassword=<database password> \  
     --set walletPassword=<wallet password> \
       <name for this install> .
  
  ***Example:***
     helm -n autodb  --set dbPassword=Admin!2345  --set walletPassword=Admin!2345 autodb .   
     
The password must be between 8 and 32 characters long, and must contain at least 1 numeric character, 1 lowercase character, 1 uppercase character, and 1        special (nonalphanumeric) character.


**6. List the Helm installation**

     helm  -n <namespace name> ls


**7. To uninstall the Helm chart**

     helm uninstall -n <namespace name> <name of the install> .
     
     
   **Important Note**
 
Uninstalling the helm chart will only remove the ATP resource from the cluster and not OCI. You will need to use the console or the OCI CLI to remove the ATP from OCI. This function is to prevent accidental deletion of the database.

     
  **Notes/Issues:**
 
 Provisioning the Autonomous Database (ATP) can take up to 5 minutes. 
 
 To confirm that the ATP is active, run the following command and check the status of the ATP system.

```sh
$ kubectl -n <name of namespace> get autonomousdatabases -o wide
NAME            DISPLAYNAME     DBWORKLOAD   STATUS   OCID                                                                                            AGE
autodbtest301   autodbtest301   OLTP         Active   ocid1.autonomousdatabase.oc1.iad.anuwcljsnlc5nbyazyyzlqxytdmghb5eyafntqnxq6cupu3zmxf6jihz6vna   28m
```

 ## Accessing the Database

Once the  ATP is ready, a secret with the name defined in values.yaml file under wallet.walletName will be created to expose the wallet files required to connect to the ATP.


| Parameter          | Description                                                              | Type   |
| ------------------ | ------------------------------------------------------------------------ | ------ |
| `ewallet.p12`      | Oracle Wallet.                                                           | string |
| `cwallet.sso`      | Oracle wallet with autologin.                                            | string |
| `tnsnames.ora`     | Configuration file containing service name and other connection details. | string |
| `sqlnet.ora`       |                                                                          | string |
| `ojdbc.properties` |                                                                          | string |
| `keystore.jks`     | Java Keystore.                                                           | string |
| `truststore.jks`   | Java trustore.                                                           | string |
| `user_name`        | Pre-provisioned DB ADMIN Username.                                       | string |





     


**3. Deploy the container image into a Kubernetes cluster**. 

One way to use the packaged Python script is to use it in an init-container for the application container in a Kubernetes pod. The init-container will run, and when the ATP is ready, it will write the wallet files to a location that is accessible to the application container.

Since the Kubernetes pod will be contacting the Kubernetes API server to check for the creation of secrets, the appropriate permissions need to the assigned to the pod. In the templates directory, the role.yaml, the role-binding.yaml file sets the permissions to access secrets to the internal-kubectl service account. This service account will need to be assigned to the Kubernetes pod to read secrets from the Kubernetes API server.

In the following example, the pods in the deployment are assigned the service account internal-kubectl, which has permission to contact the Kubernetes API server to read secrets. The init-container and the application container have the wallet volume mounted and is accessible to both. The init-container will run the Python script and write the wallet files to the wallet volume, where the application container can read the files
```




```

## Autonomous Database Specification Parameters

The Complete Specification of the `AutonomousDatabase` Custom Resource (CR) is as detailed below:

| Parameter                          | Description                                                         | Type   | Mandatory |
| ---------------------------------- | ------------------------------------------------------------------- | ------ | --------- |
| `spec.id` | The Autonomous Database [OCID](https://docs.cloud.oracle.com/Content/General/Concepts/identifiers.htm). | string | no  |
| `spec.displayName` | The user-friendly name for the Autonomous Database. The name does not have to be unique. | string | yes       |
| `spec.dbName` | The database name. The name must begin with an alphabetic character and can contain a maximum of 14 alphanumeric characters. Special characters are not permitted. The database name must be unique in the tenancy. | string | yes       |
| `spec.compartmentId` | The [OCID](https://docs.cloud.oracle.com/Content/General/Concepts/identifiers.htm) of the compartment of the Autonomous Database. | string | yes       |
| `spec.cpuCoreCount` | The number of OCPU cores to be made available to the database. | int    | yes       |
| `spec.dataStorageSizeInTBs`| The size, in terabytes, of the data volume that will be created and attached to the database. This storage can later be scaled up if needed. | int    | yes       |
| `spec.dbVersion` | A valid Oracle Database version for Autonomous Database. | string | no        |
| `spec.isDedicated` | True if the database is on dedicated [Exadata infrastructure](https://docs.cloud.oracle.com/Content/Database/Concepts/adbddoverview.htm).  | boolean | no       |
| `spec.dbWorkload`  | The Autonomous Database workload type. The following values are valid:  <ul><li>**OLTP** - indicates an Autonomous Transaction Processing database</li><li>**DW** - indicates an Autonomous Data Warehouse database</li></ul>  | string | yes       |
| `spec.isAutoScalingEnabled`| Indicates if auto scaling is enabled for the Autonomous Database OCPU core count. The default value is `FALSE`. | boolean| no        |
| `spec.isFreeTier` | Indicates if this is an Always Free resource. The default value is false. Note that Always Free Autonomous Databases have 1 CPU and 20GB of memory. For Always Free databases, memory and CPU cannot be scaled. | boolean | no |
| `spec.licenseModel` | The Oracle license model that applies to the Oracle Autonomous Database. Bring your own license (BYOL) allows you to apply your current on-premises Oracle software licenses to equivalent, highly automated Oracle PaaS and IaaS services in the cloud. License Included allows you to subscribe to new Oracle Database software licenses and the Database service. Note that when provisioning an Autonomous Database on [dedicated Exadata infrastructure](https://docs.oracle.com/iaas/Content/Database/Concepts/adbddoverview.htm), this attribute must be null because the attribute is already set at the Autonomous Exadata Infrastructure level. When using [shared Exadata infrastructure](https://docs.oracle.com/iaas/Content/Database/Concepts/adboverview.htm#AEI), if a value is not specified, the system will supply the value of `BRING_YOUR_OWN_LICENSE`. <br>Allowed values are:<ul><li>LICENSE_INCLUDED</li><li>BRING_YOUR_OWN_LICENSE</li></ul>. | string | no       |
| `spec.freeformTags` | Free-form tags for this resource. Each tag is a simple key-value pair with no predefined name, type, or namespace. For more information, see [Resource Tags](https://docs.oracle.com/iaas/Content/General/Concepts/resourcetags.htm). `Example: {"Department": "Finance"}` | string | no |
| `spec.definedTags` | Defined tags for this resource. Each key is predefined and scoped to a namespace. For more information, see [Resource Tags](https://docs.oracle.com/iaas/Content/General/Concepts/resourcetags.htm). | string | no |
| `spec.adminPassword.secret.secretName` | The Kubernetes Secret Name that contains admin password for Autonomous Database. The password must be between 12 and 30 characters long, and must contain at least 1 uppercase, 1 lowercase, and 1 numeric character. It cannot contain the double quote symbol (") or the username "admin", regardless of casing. | string | yes       |
| `spec.wallet.walletName` | The Kubernetes Secret Name of the wallet which contains the downloaded wallet information. | string | yes       |
| `spec.walletPassword.secret.secretName`| The Kubernetes Secret Name that contains the password to be used for downloading the Wallet. | string |  no  |

## Useful commands 


**1. To check the status of the Autonomous Database System run the following command**
     
     kubectl -n <namespace of autonomousdatabase> get autonomousdatabases -o wide

**2. To describe the  Autonomous Database System  run the following command** 
     
     kubectl -n <namespace of autonomousdatabase> describe autonomousdatabases 

**3. To retreive the OCID of Autonomous Database System run the following command** 

      kubectl -n <namespace of autonomousdatabase> get autonomousdatabase <name of mysqldbsystem> -o jsonpath="{.items[0].status.status.ocid}
      

**4. To retrive the wallet password of the Autonomous Database System run the following command**
     
    kubectl -n   <autonomousdatabase>   get secret <name of wallet secret>  -o  jsonpath="{.data.walletPassword}" | base64 --decode





## Additional Resources

- [OCI Service Operator for Kuberntes (OSOK) deployed in the cluster](https://github.com/oracle/oci-service-operator)
- [OCI Autonomous Database Serice OSOK](https://github.com/oracle/oci-service-operator/blob/main/docs/adb.md)
- [Oracle Autonomous Database](https://www.oracle.com/database/what-is-autonomous-database/)
- [Developing Python Applications for Oracle ATP](https://www.oracle.com/database/technologies/appdev/python/quickstartpython.html)


## License
Copyright (c) 2022 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](LICENSE) for more details.
