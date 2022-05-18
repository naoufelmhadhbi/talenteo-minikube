# Talenteo-minikube (Int environment)
+ Backend micro-services were containerized using JIB, you can follow this command syntax to build a local image:
    
    ```
    > mvn compile com.google.cloud.tools:jib-maven-plugin:1.8.0:dockerBuild -Dmaven.test.skip=true -Dimage=<image-name>:<tag>
    ```
+ Frontend App was containerized using a dockerfile, here is an example:
    ```
    FROM nginx:latest
    RUN apt update && apt -y install nginx-extras
    COPY dist/talenteo-ui /var/www/
    COPY nginx.conf /etc/nginx/nginx.conf
    RUN rm /etc/nginx/sites-enabled/default
    EXPOSE 80
    ENTRYPOINT ["nginx","-g","daemon off;"]
    ```
+ All Talenteo micro-services deployed in minikube int environment. 
+ int environment = namespace in minikube called "talenteo-int".
+ After creating "talenteo-int" namespace, you have to switch to this namespace.
+ use "minikube ip" command to get the IP @ of your minikube.
+ Add at the end of /etc/hosts this line:  
    ```
     <IP-@-minikube> int.talenteo.com
    ```

# Summary of set up
+ Install docker
+ Install minikube
+ Install helm
+ Install kubectl


# Postgres Database Config
+ create a kubernetes Secret into "talenteo-int" namespace which contain the value of POSTGRES_PASSWORD
    
    ```
    > kubectl apply -f Int/pg-helm/secrets.yml 
    ```

+ install pg-helm chart into minikube

    ```
    > helm install pg-helm-int Int/pg-helm/
    ```
+ use minikube dashboard to execute into "pg-helm-int-xxxxxxx" pod these folowing commands:

    ```
    > psql -U postgres
    ```
    ```
    > create database talenteo_db_int;
    ```
    ```
    > \c talenteo_db_int
    ```
    ```
    create user talenteo_hr_int with password 'secret';
    create schema hr authorization talenteo_hr_int;

    create user talenteo_oauth2_int with password 'secret';
    create schema oauth2 authorization talenteo_oauth2_int;

    create user talenteo_td_int with password 'secret';
    create schema td authorization talenteo_td_int;

    create user talenteo_mission_int with password 'secret';
    create schema mission authorization talenteo_mission_int;

    create user talenteo_career_int with password 'secret';
    create schema career authorization talenteo_career_int;

    create user talenteo_notification_int with password 'secret';
    create schema notification authorization talenteo_notification_int;
    ```
+ expose postgres kubernetes service
    ```
    > kubectl expose deployment pg-helm-int --type=ClusterIP
    ```

# Config-server-ms :
+ install config-server-helm chart into "talenteo-int" namespace

    ```
    > helm install config-server-helm-int Int/config-server-helm/
    ```

# Create a Certificate and Key :
+ create a self-signed key TLS certificate

    ```
    > openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls_self_int.key -out tls_self_int.crt -subj "/CN=int.talenteo.com" -days 365
    ```

# Create Kubernetes TLS Secret :
+ add key and certificate generated before as Kubernetes TLS Secret to used it in kubernetes Ingress in order to ensure a secured access and encrypted data channel via HTTPS

    ```
    > kubectl create secret tls int-talenteo-com-tls --key Int/tls_self_int.key --cert Int/tls_self_int.crt
    ```

# Oauth2-ms :
+ install oauth2-helm chart into "talenteo-int" namespace

    ```
    > helm install oauth2-helm-int Int/oauth2-helm/
    ```


# hr-ms :
+ install hr-helm chart into "talenteo-int" namespace

    ```
    > helm install hr-helm-int Int/hr-helm/
    ```
# td-ms :
+ install td-helm chart into "talenteo-int" namespace

    ```
    > helm install td-helm-int Int/td-helm/
    ```
# career-ms :
+ install career-helm chart into "talenteo-int" namespace

    ```
    > helm install career-helm-int Int/career-helm/
    ```
# mission-ms :
+ install mission-helm chart into "talenteo-int" namespace

    ```
    > helm install mission-helm-int Int/mission-helm/
    ```
# notification-ms :
+ install notification-helm chart into "talenteo-int" namespace

    ```
    > helm install notification-helm-int Int/notification-helm/
    ```
# talenteo-ui-ms :
+ install talenteo-ui-helm chart into "talenteo-int" namespace

    ```
    > helm install talenteo-ui-helm-int Int/talenteo-ui-helm/
    ```
# Test :
+ apply these files into "talenteo-int" namespace which will be helpful for ingress to route traffic correctly

    ```
    > kubectl apply -f Int/talenteo-ui-change-password.yml
    ```
    ```
    > kubectl apply -f Int/talenteo-ui-dashboard.yml
    ```
    ```
    > kubectl apply -f Int/talenteo-ui-human-resources.yml
    ```
    ```
    > kubectl apply -f Int/talenteo-ui-login.yml
    ```
    ```
    > kubectl apply -f Int/talenteo-ui-logout.yml
    ```
+ type https://int.talenteo.com in the browser url