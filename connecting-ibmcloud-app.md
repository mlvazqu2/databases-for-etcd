---

copyright:
  years: 2018, 2022
lastupdated: "2022-06-07"

keywords: etcd, kubernetes, cloud foundry

subcollection: databases-for-etcd

---

{:external: .external target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:deprecated: .deprecated}


# Connecting an {{site.data.keyword.cloud_notm}} application
{: #ibmcloud-app}

Applications running in {{site.data.keyword.cloud_notm}} can be bound to your {{site.data.keyword.databases-for-etcd_full}} deployment. 

## Connecting a Kubernetes Service application
{: #connecting-kub-service-app}

There are two steps to connecting a Cloud databases deployment to a Kubernetes Service application. First, your deployment needs a to be bound to your cluster and its connection strings that are stored in a secret. The second step is configuring your application to use the connection strings.

The sample app in the [Connecting a Kubernetes Service Tutorial](/docs/databases-for-etcd?topic=cloud-databases-tutorial-k8s-app) provides a sample application that uses Node.js and demonstrates how to bind the sample application to a {{site.data.keyword.databases-for}} deployment.
{: .tip}

Before connecting your Kubernetes Service application to a deployment, make sure that the deployment and cluster are both in the same region and resource group.

### Binding your deployment
{: #binding-deployment}

**Public Endpoints** - If you are using the default public service endpoint to connect to your deployment, you can run the `cluster service bind` command with your cluster name, the resource group and your deployment name.
```sh
ibmcloud ks cluster service bind <your_cluster_name> <resource_group> <your_database_deployment>
```
OR  
**Private Endpoints** - If you want to use a private endpoint (if one is enabled on your deployment), then first you need to create a service key for your database so Kubernetes can use it when binding to the database. 
```sh
ibmcloud resource service-key-create <your-private-key> --instance-name <your_database_deployment> --service-endpoint private  
```
The private service endpoint is selected with `--service-endpoint private`. After that, you bind the database to the Kubernetes cluster through the private endpoint with the `cluster service bind` command.
```sh
ibmcloud ks cluster service bind <your_cluster_name> <resource_group> <your_database_deployment> --key <your-private-key>
```

**Verify** - Verify that the Kubernetes secret was created in your cluster namespace. Running the following command, you get the API key for accessing the instance of your deployment that's provisioned in your account.
```sh
kubectl get secrets --namespace=default
```
More information on binding services is found in the [Kubernetes Service documentation](/docs/containers?topic=containers-service-binding#bind-services).

### Configuring in your Kubernetes app 
{: #config-kub-app}

When you bind your application to Kubernetes Service, it creates an environment variable from the cluster's secrets. Your deployment's connection information lives in `BINDING` as a JSON object. Load and parse the JSON object into your application to retrieve the information your application's driver needs to make a connection to the database. 

The [Connection Strings](/docs/databases-for-etcd?topic=databases-for-etcd-connection-strings#connection-string-breakdown) page contains a reference of the JSON fields.

More information on the environment variables is in the [Kubernetes Service docs](https://cloud.ibm.com/docs/containers?topic=containers-service-binding#reference_secret).

## Connecting a Cloud Foundry application
{: #connecting-cf-app}

{{site.data.keyword.ibmcf_full}} is deprecated. As of 30 November 2022 new {{site.data.keyword.ibmcf_full}} applications cannot be created and only existing users will be able to deploy applications. End-of-support happens on 1 June 2023. Any instances that still exist on 1 June 2023 will be deleted. For more information, see [Deprecation of IBM Cloud Foundry](/docs/cloud-foundry-public?topic=cloud-foundry-public-deprecation).
{: deprecated}

There are three steps to connecting a Cloud databases deployment to a Cloud Foundry application. First, your deployment needs a Cloud Foundry alias. The alias represents the database deployment as a Cloud Foundry service. The second step is creating a manifest file that Cloud Foundry used to associate the application with the deployment when you push the application into Cloud Foundry. The last step is configuring your application to use the connection strings generated by the binding.

The sample app in the [Getting Started](/docs/databases-for-etcd?topic=databases-for-etcd-getting-started) tutorial provides a sample Cloud Foundry application that uses Node.js and demonstrates how to bind the sample application to a {{site.data.keyword.databases-for-etcd}} deployment.
{: .tip}

### Creating a Cloud Foundry alias
{: #create-cf-alias}

Log in to the {{site.data.keyword.cloud_notm}} CLI and use the command:

`ibmcloud resource service-alias-create alias-name --instance-name instance-name`

The alias name can be the same as the database service instance name. So, for a {{site.data.keyword.databases-for-etcd}} service named "example-es", use the following command:

`ibmcloud resource service-alias-create example-es --instance-name example-es`

The alias appears in the list of _Cloud Foundry Apps_ in your _Resource List_. More information on aliases is available in the [Cloud Foundry documentation](/docs/cloud-foundry-public?topic=cloud-foundry-public-connect_app).

### Creating the manifest 
{: #create-manifest}

Cloud Foundry uses a manifest file - `manifest.yml` to associate an application with another {{site.data.keyword.cloud_notm}} service.

To create the file, open a new file and add the text:
   ```sh
   ---
   applications:
   - name:    example-application
     routes:
     - route: example-application.us-south.cf.appdomain.cloud
     memory:  128M
     services:
       - example-etcd
   ```

- Change the route value to something unique. The route that you choose determines the subdomain of your application's URL: `<route>.{region}.cf.appdomain.cloud`. Be sure the `{region}` matches where your application is deployed.
- Change the name value. The value that you choose is the name of the app as it appears in your {{site.data.keyword.cloud_notm}} dashboard.
- Update the services value to match  Cloud Foundry alias of your {{site.data.keyword.databases-for-etcd}} deployment.

If you have an existing application that you are just adding a Cloud Databases deployment to, then you probably already have a `manifest.yml`. In that case, you need to be sure to add the alias of your deployment under `services`.

You can verify that the services are connected by navigating to the _Connections_ panel. If the deployment and the application are connected, the connection shows up in both services.

More information on the manifest file is available in the [Cloud Foundry documentation](/docs/cloud-foundry-public?topic=cloud-foundry-public-deployingapps#appmanifest).

### Configuring in your Cloud Foundry app
{: #config-cf-app}

When you push your application to Cloud Foundry, it generates environment variables for connected services. Your deployment's connection information lives in `VCAP_SERVICES` as a JSON object. Load and parse the JSON object into your application to retrieve the information your application's driver needs to make a connection to the database. 

The [Connection Strings](/docs/databases-for-etcd?topic=databases-for-etcd-connection-strings#connection-string-breakdown) page contains a reference of the JSON fields.

More information on the environment variables is in the [Cloud Foundry docs](/docs/cloud-foundry-public?topic=cloud-foundry-public-deployingapps#app_env).
