pipeline {
    agent {
        node { label "maven" }
    }
    environment { 
        QUAY = credentials('QUAY_USER') 
        OPENSHIFT_SERVER = "https://api.eu46r.prod.ole.redhat.com:6443/"
        OPENSHIFT_TOKEN = credentials('Openshift-jenkins-account')
        RHT_OCP4_DEV_USER = 'jenkins'
        DEPLOYMENT_STAGE = 'home-automation-stage'
        DEPLOYMENT_PRODUCTION = 'home-automation-production'
    }

    stages {
        stage("Test") {
            steps {
                sh "./mvnw verify"
            }
        }
        stage("Build & Push Image") {
            steps {
                sh '''
                    ./mvnw quarkus:add-extension \
                    -Dextensions="container-image-jib"
                '''
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group=$QUAY_USR \
                    -Dquarkus.container-image.name=do400-deploying-lab \
                    -Dquarkus.container-image.username=$QUAY_USR \
                    -Dquarkus.container-image.password="$QUAY_PSW" \
                    -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                    -Dquarkus.container-image.additional-tags=latest \
                    -Dquarkus.container-image.push=true
                '''
            }
        }
        stage('Deploy to TEST') {
            when { not { branch "main" } }

            steps {
                sh """
                    oc login --token=${OPENSHIFT_TOKEN} --server=${OPENSHIFT_SERVER}
                    oc project jenkins
                    oc set image deployment ${DEPLOYMENT_STAGE} home-automation home-automation=quay.io/${QUAY_USR}/do400-deploying-lab:build-${BUILD_NUMBER} -n jenkins --record
                """
            }
        }    
    }
}