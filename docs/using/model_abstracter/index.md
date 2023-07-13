<a id="model_abstracter" />

Model abstracter
====================


## Enable/Disable Model Abstracter with env-variable MODEL_ABSTRACTER_STATUS
The environment variable MODEL_ABSTRACTER_STATUS is used in two scenarios.

## docker compose file

We could use docker-compose command to deploy OB-BAA. And add MODEL_ABSTRACTER_STATUS variable in node of services.baa.environment as follows. It's only a part of docker-compose.yml file to demonstrate the usage.

~~~
version: '3.5'
networks:
    baadist_default:
        driver: bridge
        name: baadist_default
services:
    baa:
        image: baa
        build: ./
        container_name: baa
        restart: always
        ports:
            - "8080:8080"
            - "5005:5005"
            - "9292:9292"
            - "4335:4335"
            - "162:162/udp"
        environment:
            #- EXTRA_JAVA_OPTS=-Xms128M -Xmx512M -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
            - BAA_USER=admin
            - BAA_USER_PASSWORD=password
            #Possible Values for PMA_SESSION_FACTORY_TYPE are REGULAR,TRANSPARENT, Default value is REGULAR
            - PMA_SESSION_FACTORY_TYPE=REGULAR
            - MAXIMUM_ALLOWED_ADAPTER_VERSIONS=3
            - VOLTMF_NAME=vOLTMF
            # Enable Model Abstracter or Disable Model Abstracter, Default value is Disable
            - MODEL_ABSTRACTER_STATUS=Disable
            # Below tag shall be set as false if the BAA is going to be tested for Scalability/Performance
            - NC_ENABLE_POST_EDIT_DS_VALIDATION_SUPPORT=True
~~~

## helm charts file
The helm chart file is used in Kubenetes, and need to add the variable in the deployment.yml of the baa chart. This variable should be added in the node of env as follows. It's also a part of the deployment.yml file.
~~~
spec:
  strategy:
    type: Recreate
  replicas: 1
  selector:
    matchLabels:
      app: baa
  template:
    metadata:
      labels:
        app: baa
    spec:
      volumes:
        - name: baa-store
          persistentVolumeClaim:
            claimName: baa-pvclaim
      hostname: baa
      containers:
        - name: baa
          image: broadbandforum/baa:latest
          imagePullPolicy: Always
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: debug
              containerPort: 5005
              protocol: TCP
            - name: ncnbissh
              containerPort: 9292
              protocol: TCP
            - name: callhometls
              containerPort: 4335
              protocol: TCP
            - name: snmp
              containerPort: 162
              protocol: UDP
          stdin: true
          tty: true
          env:
            - name: BAA_USER
              value: admin
            - name: BAA_USER_PASSWORD
              value: password
            - name: PMA_SESSION_FACTORY_TYPE
              value: REGULAR
            - name: VOLTMF_NAME
              value: vOLTMF
            # Enable Model Abstracter or Disable Model Abstracter, Default value is Disable
            - name: MODEL_ABSTRACTER_STATUS
              value: Disable
~~~
# L2 service configuration

The following describes how to config L2 service using the new YANG model.
Note: the steps related to environment preparation which contains the pOLT and onu simulator are omitted below.


1. Add OLT and ONU using 'vomci-end-to-end-config'.
2. Add ani and v-ani interfaces.
3. Create a link table between ani and v-ani.
4. Prepare profiles, e.g., line profile, service profile, and so on.
5. Create OLT node and TPs.
6. Create ONT node and TPs and LINK.
7. Create L2-V-UNI and L2-V-NNI TPs.
8. Create a link-only 1:1 scenario, this step is optional if you decide to create an N:1 scenario.
9. Create an N:1 VLAN forwarding profile.
10. Create N:1 L2 vlan tp.
11. Create a link in the N:1 scenario.

The XML samples mentioned above are as follows:

~~~add-interfaces-onu Expand source~~~
~~~create-link-table Expand source~~~
~~~create-line-bandwidth-profile Expand source~~~
~~~create-line-profile Expand source~~~
~~~create-vlan-traslation-profile Expand source~~~
~~~create-service-profile Expand source~~~
~~~create-OLT-node-and-TPs Expand source~~~
~~~create-ONU-node-and-TPs-and-LINK Expand source~~~
~~~create-1-1-l2-v-uni-and-nni-tp Expand source~~~
~~~optional-create-link-only-1-1 Expand source~~~

From here, it's the N:1 scenario, and this profile is only used in N:1 scenario.

~~~~create-N-1-vlan-forwarding-profile Expand source
create-N-1-l2-vlan-tp Expand source
optional-create-link-only-N-1 Expand source
