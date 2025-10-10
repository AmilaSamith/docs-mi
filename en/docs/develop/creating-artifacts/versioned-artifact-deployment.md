# Versioned Artifact Deployment

## Overview

By default, you cannot deploy multiple versions of the same Composite Application (CApp) in WSO2 Integrator: MI. This limitation occurs because all artifacts inside CApps are deployed with only their simple names, leading to conflicts when two projects contain the same artifact names.

With versioned artifact deployment, artifacts are deployed using fully qualified names that include the project’s `groupId`, `artifactId`, and `version`. This allows you to safely deploy multiple versions of the same CApp at the same time.

!!!Note
    By default, services such as **APIs**, **Proxy services**, and **Data services** are not exposed as versioned services, even when versioned artifact deployment is enabled at the project level.
    To expose versioned services, you must configure WSO2 Integrator: MI to allow it. See [Expose versioned services](#expose-versioned-services) for more information.

## Enable versioned deployment

You can enable versioned deployment at the project level in VS Code.

1. Open the **Project Overview** in your integration project.
2. In the **Build Details** section, enable or disable versioned artifact deployment for the project.

By default, versioned deployment is enabled for newly created projects.

<a href="{{base_path}}/assets/img/develop/create-artifacts/versioned-artifact-deployment-for-the-project.png"><img src="{{base_path}}/assets/img/develop/create-artifacts/versioned-artifact-deployment-for-the-project.png" alt="Enable or disable versioned artifact deployment for the project" width="80%"></a>

When this option is enabled, all artifacts in the project except services are deployed using their fully qualified names.

## Expose versioned services

By default, services such as APIs, Proxy Services, and Data Services are not exposed as versioned services.
To expose them, set the following property to `true` in the `<MI_HOME>/conf/deployment.toml` file:

```toml
[synapse_properties]
'expose.versioned.services' = true
```

After enabling this property, MI exposes services as versioned endpoints. When invoking a service, use the fully qualified name in the URL. For details, see [Resulting deployment names and invocation URLs](#resulting-deployment-names-and-invocation-urls).

## Example versioned deployment

Consider a project with the following configuration:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.microintegrator.project</groupId>
    <artifactId>payment</artifactId>
    <version>1.0.1</version>
    <packaging>pom</packaging>
    <name>HelloWorldService</name>
</project>
```

**Sample API**

```xml
<api context="/payment" name="PaymentAPI" xmlns="http://ws.apache.org/ns/synapse">
    <resource methods="GET">
        <inSequence>
            <sequence key="sequence_in_call"/>
            <respond/>
        </inSequence>
    </resource>
</api>
```

**Sample Proxy Service**

```xml
<proxy name="PaymentProxy" startOnLoad="true" transports="http https" xmlns="http://ws.apache.org/ns/synapse">
    <target>
        <inSequence>
            <log level="full"/>
            <respond/>
        </inSequence>
    </target>
</proxy>
```

**Sample Data Service**

```xml
<data name="PaymentDS" serviceNamespace="http://ws.wso2.org/dataservice" xmlns="http://ws.wso2.org/ns/ns/data">
   <config id="default">
      <property name="org.wso2.ws.dataservice.driver">org.h2.Driver</property>
      <property name="org.wso2.ws.dataservice.protocol">jdbc:h2:mem:testDB</property>
      <property name="org.wso2.ws.dataservice.user">sa</property>
      <property name="org.wso2.ws.dataservice.password"/>
   </config>
   <query id="selectAll" useConfig="default">
      <sql>SELECT 1</sql>
   </query>
   <operation name="getAll">
      <call-query href="selectAll"/>
   </operation>
</data>
```
### Resulting deployment names and invocation URLs

When versioned artifact deployment is enabled, the deployment names and invocation URLs of services (APIs, Proxy Services, and Data Services) depend on the value of the `expose.versioned.services` property defined in the `<MI_HOME>/conf/deployment.toml` file.

| Service Type | When `expose.versioned.services = false` | When `expose.versioned.services = true` |
|------------------|----------------------------------------------|---------------------------------------------|
| API | **Name:** `PaymentAPI`<br>**URL:** `http://localhost:8290/paymentapi` | **Name:** `com.microintegrator.project__payment__1.0.1__PaymentAPI`<br>**URL:** `http://localhost:8290/com.microintegrator.project/payment/1.0.1/paymentapi` |
| Proxy Service | **Name:** `PaymentProxy`<br>**URL:** `http://localhost:8290/services/PaymentProxy` | **Name:** `com.microintegrator.project__payment__1.0.1/PaymentProxy`<br>**URL:** `http://localhost:8290/services/com.microintegrator.project__payment__1.0.1/PaymentProxy` |
| Data Service | **Name:** `PaymentDS`<br>**URL:** `http://localhost:8290/services/PaymentDS` | **Name:** `com.microintegrator.project__payment__1.0.1/PaymentDS`<br>**URL:** `http://localhost:8290/services/com.microintegrator.project__payment__1.0.1/PaymentDS` |

For the rest of the artifacts, when versioned artifact deployment is enabled, the deployment names follow the format `<groupId>__<artifactId>__<version>__<artifactName>`.  

For example, the sequence `sequence_in_call` will be deployed as:

```
com.microintegrator.project__payment__1.0.1__sequence_in_call
```

!!! Note
    - References between artifacts (e.g., sequences, endpoints) are automatically updated to use fully qualified names—no manual changes are required within the project.  
    - When referencing an artifact from a **dependent project**, you should use the fully qualified name **without the version** in the `<groupId>__<artifactId>__<artifactName>` format. For example:  
      ```
      <sequence key="com.microintegrator.project__utils__loggerSeq"/>
      ```
      If you use the [WSO2 Integrator: MI extension for VS Code]({{base_path}}/develop/mi-for-vscode/mi-for-vscode-overview/), the references will be automatically listed in this format, so no manual changes are necessary. The version of the referenced artifacts will be resolved from the dependent project details defined in the `pom.xml`.
    - The [WSO2 Integrator: MI extension for VS Code]({{base_path}}/develop/mi-for-vscode/mi-for-vscode-overview/) lists services by their simple names. If you have enabled versioned service exposure in WSO2 Integrator: MI, make sure to update the service references in your project accordingly.
