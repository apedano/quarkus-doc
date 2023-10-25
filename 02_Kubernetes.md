# Kubernetes deployment of a Quarkus application

Quakus implement the possibility to build kubernetes deployment application YAML files automatically

## Add Quarkus Kubernetes extension

Add the following dependency

````xml
<dependency>
 <groupId>io.quarkus</groupId>
 <artifactId>quarkus-kubernetes</artifactId>
</dependency>
````

When running ```clean install``` the dependency will generate both json and yaml config files in ```/target/kubernetes```.

In case we want to deploy to an Openshift cluster we can add the following dependency

````xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-openshift</artifactId>
</dependency>
````

[Guide](https://access.redhat.com/documentation/en-us/red_hat_build_of_quarkus/2.2/guide/9f629efb-0765-445e-9c2f-6ff721bd86bd)

We are using the deployment in the native mode (non jvm)

## Add configuration

Add the configuration for openshift build in the ````application.properties```` file as specified in the guide

## Login into the Openshift cluster

do the login action from the local machine with the desired project

## Build and deploy to Openshift the application

Execute the following goal

````mvn clean package -Pnative -Dquarkus.kubernetes.deploy=true````

### Native image build

The first step is the native image creation based on the Mandrel container  

````shell
[INFO] [io.quarkus.deployment.pkg.steps.NativeImageBuildStep] Running Quarkus native-image plugin on native-image 22.3.2.1-Final Mandrel Distribution (Java Version 17.0.7+7)
GraalVM Native Image: Generating 'account-service-1.0.0-SNAPSHOT-runner' (executable)...
========================================================================================================================
[1/7] Initializing...                                                                                    (9,5s @ 0,13GB)
 Version info: 'GraalVM 22.3.2.1-Final Java 17 Mandrel Distribution'
 Java version info: '17.0.7+7'
 C compiler: gcc (redhat, x86_64, 8.5.0)
 Garbage collector: Serial GC
 3 user-specific feature(s)
 - io.quarkus.runner.Feature: Auto-generated class by Quarkus from the existing extensions
 - io.quarkus.runtime.graal.DisableLoggingFeature: Disables INFO logging during the analysis phase
 - org.eclipse.angus.activation.nativeimage.AngusActivationFeature
[2/7] Performing analysis...  [*******]                                                                 (32,9s @ 2,63GB)
  12.117 (89,21%) of 13.583 classes reachable
  17.294 (58,97%) of 29.329 fields reachable
  61.637 (56,26%) of 109.551 methods reachable
     561 classes,   216 fields, and 3.043 methods registered for reflection
      63 classes,    68 fields, and    55 methods registered for JNI access
       4 native libraries: dl, pthread, rt, z
[3/7] Building universe...                                                                               (4,3s @ 5,00GB)
[4/7] Parsing methods...      [**]                                                                       (3,4s @ 4,75GB)
[5/7] Inlining methods...     [***]                                                                      (1,7s @ 2,49GB)
[6/7] Compiling methods...    [****]                                                                    (20,8s @ 3,11GB)
[7/7] Creating image...                                                                                  (5,4s @ 4,98GB)
  23,37MB (49,26%) for code area:    40.096 compilation units
  23,75MB (50,06%) for image heap:  299.861 objects and 13 resources
 331,80KB ( 0,68%) for other data
  47,45MB in total
------------------------------------------------------------------------------------------------------------------------
Top 10 packages in code area:                               Top 10 object types in image heap:
   1,63MB sun.security.ssl                                     5,23MB byte[] for code metadata
 992,37KB java.util                                            2,88MB java.lang.Class
 744,69KB java.lang.invoke                                     2,76MB java.lang.String
 717,64KB com.sun.crypto.provider                              2,50MB byte[] for general heap data
 648,16KB io.quarkus.runtime.generated                         2,25MB byte[] for java.lang.String
 455,22KB java.lang                                            1,02MB com.oracle.svm.core.hub.DynamicHubCompanion
 450,58KB sun.security.x509                                  678,76KB byte[] for reflection metadata
 429,25KB java.util.concurrent                               656,72KB java.util.HashMap$Node
 404,44KB io.netty.buffer                                    548,12KB java.lang.String[]
 400,74KB java.io                                            429,30KB c.o.svm.core.hub.DynamicHub$ReflectionMetadata
  16,32MB for 449 more packages                                4,62MB for 3005 more object types
------------------------------------------------------------------------------------------------------------------------
                        3,0s (3,6% of total time) in 31 GCs | Peak RSS: 6,31GB | CPU load: 7,80
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /project/account-service-1.0.0-SNAPSHOT-runner (executable)
 /project/account-service-1.0.0-SNAPSHOT-runner-build-output-stats.json (json)
 /project/account-service-1.0.0-SNAPSHOT-runner-timing-stats.json (raw)
 /project/account-service-1.0.0-SNAPSHOT-runner.build_artifacts.txt (txt)
========================================================================================================================
Finished generating 'account-service-1.0.0-SNAPSHOT-runner' in 1m 23s.
````

### Cluster image build (BuildConfig and ImageStream)

The first step is the native image creation based on the Mandrel container

````shell
Starting (in-cluster) container image build for jar using: DOCKER on server: https://api.sandbox-m4.g2pi.p1.openshiftapps.com:6443/ in namespace:xan80-dev.
...
````

The build config is a Docker build (the option supporting native images).
The content of the Dockerfile is the same as from  ```src/main/docker/Dockerfile.native```. The builds is launched from the local machine (so it has visibility on the target folder containing the built native image)
The generated Openshift resources are


````yaml
kind: BuildConfig
apiVersion: build.openshift.io/v1
metadata:
  annotations:
    app.openshift.io/vcs-url: <<unknown>>
    app.quarkus.io/build-timestamp: '2023-06-24 - 14:16:31 +0000'
    app.quarkus.io/commit-id: 015e167037eec95776b10f99a102aef536aad29f
  name: account-service
  namespace: xan80-dev
  labels:
    app.kubernetes.io/managed-by: quarkus
    app.kubernetes.io/name: account-service
    app.kubernetes.io/version: 1.0.0-SNAPSHOT
    app.openshift.io/runtime: quarkus
spec:
  runPolicy: Serial
  source:
    type: Dockerfile
    dockerfile: "####\r\n# This Dockerfile is used in order to build a container that runs the Quarkus application in native (no JVM) mode.\r\n#\r\n# Before building the container image run:\r\n#\r\n# ./mvnw package -Pnative\r\n#\r\n# Then, build the image with:\r\n#\r\n# docker build -f src/main/docker/Dockerfile.native -t quarkus/account-service .\r\n#\r\n# Then run the container using:\r\n#\r\n# docker run -i --rm -p 8080:8080 quarkus/account-service\r\n#\r\n###\r\nFROM registry.access.redhat.com/ubi8/ubi-minimal:8.6\r\nWORKDIR /work/\r\nRUN chown 1001 /work \\\r\n    && chmod \"g+rwX\" /work \\\r\n    && chown 1001:root /work\r\nCOPY --chown=1001:root target/*-runner /work/application\r\n\r\nEXPOSE 8080\r\nUSER 1001\r\n\r\nCMD [\"./application\", \"-Dquarkus.http.host=0.0.0.0\"]\r\n"
  strategy:
    type: Docker
    dockerStrategy: {}
  output:
    to:
      kind: ImageStreamTag
      name: 'account-service:1.0.0-SNAPSHOT'
  resources: {}
  postCommit: {}
  nodeSelector: null
status:
  lastVersion: 2
````

````yaml
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  annotations:
    app.openshift.io/vcs-url: <<unknown>>
    app.quarkus.io/build-timestamp: '2023-06-24 - 14:16:31 +0000'
    app.quarkus.io/commit-id: 015e167037eec95776b10f99a102aef536aad29f
  name: account-service
  namespace: xan80-dev
  labels:
    app.kubernetes.io/managed-by: quarkus
    app.kubernetes.io/name: account-service
    app.kubernetes.io/version: 1.0.0-SNAPSHOT
    app.openshift.io/runtime: quarkus
spec:
  lookupPolicy:
    local: true
status:
  dockerImageRepository: 'image-registry.openshift-image-registry.svc:5000/xan80-dev/account-service'
  publicDockerImageRepository: >-
    default-route-openshift-image-registry.apps.sandbox-m4.g2pi.p1.openshiftapps.com/xan80-dev/account-service
  tags:
    - tag: 1.0.0-SNAPSHOT
      items:
        - created: '2023-06-24T14:16:54Z'
          dockerImageReference: >-
            image-registry.openshift-image-registry.svc:5000/xan80-dev/account-service@sha256:dc23a1f088233b5a3631ee5e776f4214469e06f42ceed148db59efe92cb9cb22
          image: >-
            sha256:dc23a1f088233b5a3631ee5e776f4214469e06f42ceed148db59efe92cb9cb22
          generation: 1
        - created: '2023-06-24T13:31:52Z'
          dockerImageReference: >-
            image-registry.openshift-image-registry.svc:5000/xan80-dev/account-service@sha256:76ffd5b404e2a3528020d8a02a62c7d56a0bea34f39bba1dd1ccf0d98630909f
          image: >-
            sha256:76ffd5b404e2a3528020d8a02a62c7d56a0bea34f39bba1dd1ccf0d98630909f
          generation: 1
````

After the build in openshift happens, the image stream will be ready to be used in the Openshift application 

### Deploy the application to the cluster

The process will generate the _DeploymentConfig_, the _Service_ and the _Route_ to expose the app endpoints externally. 

````shell

[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Deploying to openshift server: https://api.sandbox-m4.g2pi.p1.openshiftapps.com:6443/ in namespace: xan80-dev.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: Service account-service.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: ImageStream s2i-java.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: ImageStream account-service.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: BuildConfig account-service.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: DeploymentConfig account-service.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] Applied: Route account-service.
[INFO] [io.quarkus.kubernetes.deployment.KubernetesDeployer] The deployed application can be accessed at: http://account-service-xan80-dev.apps.sandbox-m4.g2pi.p1.openshiftapps.com
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 133466ms
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:26 min
[INFO] Finished at: 2023-06-24T16:17:00+02:00
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0

````