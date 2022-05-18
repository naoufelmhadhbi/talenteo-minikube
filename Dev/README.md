# Talenteo-minikube (Dev environment)
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
+ All Talenteo micro-services deployed in minikube Dev environment. 
+ Dev environment = namespace in minikube called "talenteo-dev".
+ After creating "talenteo-dev" namespace, you have to switch to this namespace.
+ use "minikube ip" command to get the IP @ of your minkube.
+ Add at the end of /etc/hosts this line:  
    ```
     <IP-@-minikube> dev.talenteo.com
    ```

# Summary of set up
+ Install docker
+ Install minikube
+ Install helm
+ Install kubectl


# Postgres Database Config
+ create a kubernetes Secret into "talenteo-dev" namespace which contain the value of POSTGRES_PASSWORD
    
    ```
    > kubectl apply -f Dev/pg-helm/secrets.yml 
    ```

+ install pg-helm chart into minikube

    ```
    > helm install pg-helm-dev Dev/pg-helm/
    ```
+ use minikube dashboard to execute into "pg-helm-dev-xxxxxxx" pod these folowing commands:

    ```
    > psql -U postgres
    ```
    ```
    > create database talenteo_db_dev;
    ```
    ```
    > \c talenteo_db_dev
    ```
    ```
    create user talenteo_hr_dev with password 'secret';
    create schema hr authorization talenteo_hr_dev;

    create user talenteo_oauth2_dev with password 'secret';
    create schema oauth2 authorization talenteo_oauth2_dev;

    create user talenteo_td_dev with password 'secret';
    create schema td authorization talenteo_td_dev;

    create user talenteo_mission_dev with password 'secret';
    create schema mission authorization talenteo_mission_dev;

    create user talenteo_career_dev with password 'secret';
    create schema career authorization talenteo_career_dev;

    create user talenteo_notification_dev with password 'secret';
    create schema notification authorization talenteo_notification_dev;
    ```

+ expose postgres kubernetes service
    ```
    > kubectl expose deployment pg-helm-dev -n talenteo-dev --type=ClusterIP
    ```


# Config-server-ms :
+ install config-server-helm chart into "talenteo-dev" namespace

    ```
    > helm install config-server-helm-dev Dev/config-server-helm/
    ```

# Create a Certificate and Key :
+ create a self-signed key TLS certificate

    ```
    > openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls_self_dev.key -out tls_self_dev.crt -subj "/CN=dev.talenteo.com" -days 365
    ```

# Create Kubernetes TLS Secret :
+ add key and certificate generated before as Kubernetes TLS Secret to used it in kubernetes Ingress in order to ensure a secured access and encrypted data channel via HTTPS

    ```
    > kubectl create secret tls dev-talenteo-com-tls --key tls_self_dev.key --cert tls_self_dev.crt
    ```

# Oauth2-ms :
+ install oauth2-helm chart into "talenteo-dev" namespace

    ```
    > helm install oauth2-helm-dev Dev/oauth2-helm/
    ```

# hr-ms :
+ install hr-helm chart into "talenteo-dev" namespace

    ```
    > helm install hr-helm-dev Dev/hr-helm/
    ```
# td-ms :
+ install td-helm chart into "talenteo-dev" namespace

    ```
    > helm install td-helm-dev Dev/td-helm/
    ```
# career-ms :
+ install career-helm chart into "talenteo-dev" namespace

    ```
    > helm install career-helm-dev Dev/career-helm/
    ```
# mission-ms :
+ install mission-helm chart into "talenteo-dev" namespace

    ```
    > helm install mission-helm-dev Dev/mission-helm/
    ```
# notification-ms :
+ install notification-helm chart into "talenteo-dev" namespace

    ```
    > helm install notification-helm-dev Dev/notification-helm/
    ```
# talenteo-ui-ms :
+ install talenteo-ui-helm chart into "talenteo-dev" namespace

    ```
    > helm install talenteo-ui-helm-dev Dev/talenteo-ui-helm/
    ```
# Test :
+ apply these files into "talenteo-dev" namespace which will be helpful for ingress to route traffic correctly

    ```
    > kubectl apply -f Dev/talenteo-ui-change-password.yml
    ```
    ```
    > kubectl apply -f Dev/talenteo-ui-dashboard.yml
    ```
    ```
    > kubectl apply -f Dev/talenteo-ui-human-resources.yml
    ```
    ```
    > kubectl apply -f Dev/talenteo-ui-login.yml
    ```
    ```
    > kubectl apply -f Dev/talenteo-ui-logout.yml
    ```
+ type https://dev.talenteo.com in the browser url
