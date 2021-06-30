ballerina-kubernetes-plugin-headstart
# Ballerina Kubernetes Plugin - Headstart

Based on "Ballerina Kubernetes Extension" at https://porter.io/github.com/ballerinax/kubernetes

## 100 - How to build
- Download and install JDK 11
- Install Docker
- Get a clone or download the source from this repository (https://github.com/ballerinax/kubernetes)
- Run the Gradle command gradle build from within the kubernetes directory.
- Copy build/kubernetes-extension-***.jar file to <BALLERINA_HOME>/bre/lib directory.

### 100 - Enabling debug logs
- Use the "BAL_DOCKER_DEBUG=true" environment variable to enable docker related debug logs when building the ballerina source(s).
- Use the "BAL_KUBERNETES_DEBUG=true" environment variable to enable kubernetes related debug logs when building the ballerina source(s).

## 200 - Deploy ballerina service directly using kubectl command.
This repository also provides a kubectl plugin which allows to build ballerina programs and deploy their kubernetes artifacts directly to a kuberetes cluster. The plugin is located at kubernetes-extension/src/main/resources/kubectl-extension/kubectl-ballerina-deploy. Follow the steps mentioned in "Extend kubectl with plugins" . Check if the plugin is available using the command kubectl plugin list.

## 300 - Replacing values with environment variables.
You can replace values in an annotation using environment variables. The replacement is done with a string placeholder like "$env{ENV_VAR}". As an example lets say that you want to set the namespace field in the @kubernetes:Deployment{} annotation with a environment variable and the name of the environment variable is K8S_NAMESPACE. Following is how the annotation would look like:

```
@kubernetes:Deployment {
    namespace: "$env{K8S_NAMESPACE}"
}
```

Note: You cannot use the ballerina/config module to replace values in the annotation. This is because the kubernetes artifacts are generated during compile time. The ballerina/config module works in the runtime.

## 100 - How to execute:

```
$> kubectl ballerina deploy hello_world_k8s.bal
> building ballerina source...
Compiling source
    hello_world_k8s.bal

Generating executable
    hello_world_k8s.jar

Generating artifacts...

    @kubernetes:Service              - complete 1/1
    @kubernetes:Deployment           - complete 1/1
    @kubernetes:Docker           - complete 2/2
    @kubernetes:Helm             - complete 1/1

    Execute the below command to deploy the Kubernetes artifacts:
    kubectl apply -f ./kubernetes

    Execute the below command to install the application using Helm:
    helm install --name hello-world-k8s-deployment ./kubernetes/hello-world-k8s-deployment

> deploying artifacts...
executing 'kubectl apply -f ./kubernetes'
service/helloworld-svc unchanged
deployment.apps/hello-world-k8s-deployment configured

> deployment complete!
Annotation Usage Sample:
import ballerina/http;
import ballerina/log;
import ballerina/kubernetes;

@kubernetes:Ingress {
    hostname: "abc.com"
}
@kubernetes:Service {
    name:"hello"
}
listener http:Listener helloEP = new(9090);

@kubernetes:Deployment {
    livenessProbe: true
}
@http:ServiceConfig {
    basePath: "/helloWorld"
}
service helloWorld on helloEP {
    resource function sayHello(http:Caller caller, http:Request request) {
        http:Response response = new;
        response.setTextPayload("Hello, World from service helloWorld ! ");
        var responseResult = caller->respond(response);
        if (responseResult is error) {
            log:printError("error responding back to client.", err = responseResult);
        }
    }
}
```

The kubernetes artifacts will be created in following structure.

```
$> tree kubernetes
kubernetes/
├── docker
│   └── Dockerfile
├── hello-world-deployment
│   ├── Chart.yaml
│   └── templates
│       └── hello-world.yaml
└── hello-world.yaml
```
