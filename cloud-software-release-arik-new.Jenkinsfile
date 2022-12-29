/*  
DESCRIPTION:
This Jenkinsfile is responsible to pack the latest versions of the software (both MS and Core developments) for the Cloud

MAINTAINER:
arikr@centerity.com

USAGE:
To be used as Jenkinsfile in a Jenkins pipeline

*/

// imports for work with attached json file parameter
import groovy.json.JsonOutput
import groovy.json.JsonSlurper

    def infraImages = [
        [ImageName: "bitnami/redis", ImageTag: "5.0.7-debian-9-r12"],
        [ImageName: "postgresql-root", ImageTag: "12.1.0"],
        [ImageName: "bitnami/minideb", ImageTag: "stretch"],
        [ImageName: "bitnami/minideb", ImageTag: "latest"],
        [ImageName: "bitnami/kubectl", ImageTag: "latest"],
        [ImageName: "lwolf/kubectl_deployer", ImageTag: "0.4"],
        [ImageName: "confluentinc/cp-kafka", ImageTag: "5.0.1"],
        [ImageName: "create-paths", ImageTag: "5.2.41"],
        [ImageName: "bitnami/influxdb", ImageTag: "1.7.9"],
        [ImageName: "init-mkdir-enricher", ImageTag: "5.2.41"],
        [ImageName: "elasticsearch/elasticsearch", ImageTag: "6.8.12"],
        [ImageName: "google_samples/k8szk", ImageTag: "v3"],
        [ImageName: "depends-on", ImageTag: "latest"]
        // "bitnami/redis"  : "5.0.7-debian-9-r12",
        // "bitnami/postgresql" : "12.1.0",
        // "bitnami/minideb"  : "stretch",
        // "bitnami/minideb"  : "latest",
        // "lwolf/kubectl_deployer" : "0.4",
        // "google_samples/k8szk" : "v3",
        // "confluentinc/cp-kafka" : "5.0.1",
        // "elasticsearch/elasticsearch" : "6.8.12",
        // "create-paths" : "5.2.41",
        // "bitnami/influxdb" : "1.7.9",
        // "init-mkdir-enricher" : "5.2.41"
    ]

    // def appImages = [
    //     "configurationv5" : "latest"
    // ]

try {
    // Run all the stages on the Jenkins master server.
    node("dind") {        
        withCredentials([usernamePassword(credentialsId: 'centerity-ci-user', usernameVariable: 'CI_CREDS_USR', passwordVariable: 'CI_CREDS_PSW')]) {
            if (Package_Version.size() ==0) {
                echo "Package_Version param is empty"
                return
            }

            // Check attached json file
            validateJsonFileParameter(Components)

            Artifact = Artifact.split(',')
            Components = Components.split(',')
            // MariaDB is required part of app-infra
            if ( Artifact.contains('app-infra') && !Components.contains('MariaDB') ) {
                Components+="MariaDB"
            }

            // Get lastBuildNumber from git and use it+1 for current artifact
            Artifact_ID = lastBuildNumber.getArtifactID()

            def Issues=[]
            prefix="Centerity_Monitor_V5.1.${Artifact_ID}"
            dateSuffix=new Date().format( 'yyyyMMddHHmmSS' ).toString() //%Y%m%d%H%M%S
            dateTxt=new Date().format( 'HH:mm dd-MM-yyyy' ).toString()
            dateDir=new Date().format( 'yyyy-MM-dd' ).toString()
            withEnv(["dateTxt=${dateTxt}", "dateSuffix=${dateSuffix}", "rls_iso=${prefix}${Package_Suffix}.iso", "test_iso=${prefix}_TEST_${dateSuffix}", "test_dir=${TEST_SHARE}/${TEST_PATH}/${dateDir}", "rls_dir=${RELEASE_SHARE}/${RELEASE_PATH}/${dateDir}","JIRA_SITE=Centerity Jira"]) {
            
            stage('Checkout Repos') {
                checkout(scm)
                    sh "curl -ks https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz | tar -xzf - && mv ./linux-amd64/helm ./ && chmod +x ./helm "
                        dir('monitor'){
                            checkout([$class: 'GitSCM', 
                                    branches: [[name: MS_Branch ? "*/${MS_Branch}" :  "refs/tags/${MS_Tag}"]],
                                    depth: 1,
                                    extensions: [[$class: 'CloneOption', timeout: 180, shallow: true, honorRefspec: true, noTags: true],[$class: 'CleanBeforeCheckout']],
                                    userRemoteConfigs: [[credentialsId: 'Gitlab_Beanstalk_Token', refspec: '+refs/heads/${MS_Branch}:refs/remotes/origin/${MS_Branch}', url: 'https://gitlab.com/centerity/beanstalkapp/monitor.git']]])
                        }
                        dir('perl'){
                            checkout([$class: 'GitSCM', 
                                    branches: [[name: '*/master']],
                                    depth: 1,
                                    extensions: [[$class: 'CloneOption', timeout: 180, shallow: true, honorRefspec: true, noTags: true],[$class: 'CleanBeforeCheckout']],
                                    userRemoteConfigs: [[credentialsId: 'Gitlab_Beanstalk_Token', refspec: '+refs/heads/master:refs/remotes/origin/master', url: 'https://gitlab.com/centerity/beanstalkapp/perl.git']]])
                        }
                    sh "ls -la"
                        dir('resource_linux'){
                            checkout([$class: 'GitSCM', 
                                    branches: [[name: '*/containerized_build']], 
                                    depth: 1,
                                    extensions: [[$class: 'CloneOption', timeout: 180, shallow: true, honorRefspec: true, noTags: true], [$class: 'CleanBeforeCheckout']], 
                                    userRemoteConfigs: [[credentialsId: 'Gitlab_Resource_Linux_Token',  refspec: '+refs/heads/containerized_build:refs/remotes/origin/containerized_build', url: 'https://gitlab.com/centerity/resource_linux.git']]])
                        }
                    sh '''
                        echo "ISO Name: ${rls_iso}" >> resource_linux/ISO/CM/Centerity/version.txt
                        echo "ISO Build Date: ${dateTxt}" >> resource_linux/ISO/CM/Centerity/version.txt
                        ls -la
                    '''
                 }
            
            stage('Prepare Workspace') {
                    sh '''
                    echo $IMAGE_VERSION | base64 -d > image_version.json
                    ls -la
                    mkdir -p MS_Package/
                    tar -czf MS_Package/monitor.tar.gz monitor
                    tar -czf MS_Package/perl.tar.gz perl
                    mv resource_linux/ISO/CM/ MS_Package/.
                    mv MS_Package/CM/files/mysql_init/ MS_Package/.
                    cd monitor/install/dbs/
                    cp monitor/global-monitor-5.0.2.db nagios/global-nagios-5.0.db reports/global-reports-5.0.2.db stager/global-stager-4.0.db inventory/global-inventory-5.0.2.db netflow/global-netflow-5.0.db ../../../MS_Package/mysql_init/.
                    cd ../../../MS_Package/mysql_init/
                    chmod 777 * 
                    
                    for f in *.db; do 
                        mv -- "$f" "${f%.db}.sql"
                    done
                    
                    
                    sed -i 's/use reports;/CREATE DATABASE `reports`; use `reports`;/g' global-reports*.sql
                    sed -i 's/use nagios;/CREATE DATABASE `nagios`; use `nagios`;/g' global-nagios*.sql
                    sed -i 's/use stager;/CREATE DATABASE `stager`; use `stager`;/g' global-stager*.sql
                    sed -i 's/use inventory;/CREATE DATABASE `inventory`; use `inventory`;/g' global-inventory*.sql
                    sed -i 's/use netflow;/CREATE DATABASE `netflow`; use `netflow`;/g' global-netflow*.sql
                    '''
                }

            println Components.toString()
            if ((Components.contains('Timerider')) || (Components.contains('Dealer')) || (Components.contains('Notifier'))) {
                  stage('Get Engine Package') {
                    sh '''
                        if [[ `echo $ENGINE_PACKAGE_URL | grep production` ]]; then 
                            CLUSTER_FILE="$(curl -ls "$ENGINE_PACKAGE_URL/" | grep cluster | grep -e "${CLUSTER_VERSION}")"
                        else
                            PACKAGE_DIRS="$(curl -ls "$ENGINE_PACKAGE_URL/" | grep -E "^($(date '+%Y')|$(($(date '+%Y')-1)))" | sort -r)"
                            for dir in $PACKAGE_DIRS; do
                                if [ -z "$CLUSTER_FILE" ]; then
                                    CLUSTER_FILE="$(curl -ls "$ENGINE_PACKAGE_URL/$dir/" | grep "cluster.*${CLUSTER_VERSION}.*rel.tgz" | sort -r | head -1)"
                                    CLUSTER_DIR=$dir
                                fi
                                if [ -n "$CLUSTER_FILE" ]; then
                                    CLUSTER_FILE="$CLUSTER_DIR/$CLUSTER_FILE"
                                    break
                                fi
                            done
                        fi
                        
                        curl -ks -u "${CI_CREDS_USR}:${CI_CREDS_PSW}" "$ENGINE_PACKAGE_URL/$CLUSTER_FILE" -o release.tgz
                        tar -zxf release.tgz
                        cp docker/dealer.Dockerfile opt/centerity/dealer/
                        cp opt/centerity/dist/gateEnvelop-0.9.7.linux-x86_64.tar.gz opt/centerity/dealer/
                        cp docker/dealer-httpd.Dockerfile opt/centerity/dealer/
                        cp docker/dealer-jsonapi.Dockerfile opt/centerity/dealer/
                        cp docker/dealer-runnerscheduler.Dockerfile opt/centerity/dealer/
                        cp docker/db-sync.Dockerfile opt/centerity/dealer/
                        cp docker/cluster.Dockerfile opt/centerity/
                        cp docker/timerider.Dockerfile opt/centerity/
                        cp docker/notifier.Dockerfile opt/centerity/
                       
                    '''
                }
            }

            if (Components.contains('Notifier')){
                stage('Get Notifier files') {
                        node("msbuild") {
                            sh '''
                                SPTH=$(pwd) && cd /usr/local/centerity/Monitor/tools/ && tar -czvf notifier.tar.gz mqtt_notification/centerity_mqtt_notifier CenterityGnokii servicenow_ticketer  Centerity_sendEmail && mv notifier.tar.gz $SPTH/. && cd $SPTH
                            '''
                            stash includes: "notifier.tar.gz", name: 'NotifierFiles'
                            sh "rm -rf notifier.tar.gz"
                        }
                        unstash 'NotifierFiles'
                       sh '''
                            tar -xvf notifier.tar.gz -C opt/centerity/
                        '''   
                }
            }  

            if (Components.contains('CS')){
                stage('Get the CS file') {
                        node("msbuild") {
                            dir ('/usr/local/centerity/Monitor/tools/') {
                                stash includes: "CenterityCS", name: 'CSFile'
                            }
                            
                        }
                        dir ('MS_Package/') {
                                unstash 'CSFile'
                            } 
                }
            }

            if (Components.contains('QueueHandler')){
            stage('Get the QueueHandler file') {
                    node("msbuild") {
                        dir ('/usr/local/centerity/Monitor/tools/') {
                            stash includes: "QueueHandler", name: 'QueueHandlerFile'
                        }
                        
                    } 
                    dir ('MS_Package/') {
                            unstash 'QueueHandlerFile'
                        }
                        
                }
            }

            if (Components.contains('Transformer')){
                stage('Get the transformer files') {
                    checkout(scm)
                        dir('transformer'){
                            checkout([$class: 'GitSCM', 
                                    branches: [[name: '*/develop']],
                                    depth: 1,
                                    extensions: [[$class: 'CloneOption', timeout: 180, shallow: true, honorRefspec: true, noTags: true],[$class: 'CleanBeforeCheckout']],
                                    userRemoteConfigs: [[credentialsId: 'GITLAB_Transformer_Token', refspec: '+refs/heads/develop:refs/remotes/origin/develop', url: 'https://gitlab.com/centerity/ms/transformer.git']]])
                            sh '''
                            mv Dockerfile ../MS_Package/transformer.Dockerfile
                            mv package.json tsconfig.json src/ ../MS_Package/
                            '''
                        }  
                    }
            }

            // Create the relevant variables for the build
            dir=new Date().format( 'yyyy-MM-dd' ).toString()
            TAR_FILE="Centerity_Engine_${Package_Version}.${Artifact_ID}_${Release_Tag}${Package_Suffix}"

            // Start with the Build docker images block
            withEnv(["VERSION=${Package_Version}.${Artifact_ID}","CLUSTER_FILE=centerity-${Package_Version}.${Artifact_ID}-${Release_Tag}.tar","TAR_FILE=${TAR_FILE}",
            "TEST_DIR=${TEST_URL}/${dir}","RELEASE_DIR=${RELEASE_URL}/${dir}","HELM_OPTIONS=${Additional_Options}","REGISTRY_HTTP_SECRET=abcd1234"]) {
                docker.withRegistry('https://registry.gitlab.com/centerity/infra', 'Gitlab_Infra_Token') {
                    docker.withRegistry('https://registry.gitlab.com/centerity/devops', 'Gitlab_Devops_RW_Token') {
                        stage('Build Docker Images') {
                            sh """
                            cp docker/ui.Dockerfile MS_Package/ 
                            cp docker/api-ui.Dockerfile MS_Package/
                            cp docker/cs.Dockerfile MS_Package/
                            cp docker/bsm.Dockerfile MS_Package/
                            cp docker/queuehandler.Dockerfile MS_Package/
                            cp docker/mariadb.Dockerfile MS_Package/
                            cp docker/debian-tall-centerity.Dockerfile ./
                            mkdir app-images
                            """
                            def cluster = false
                            Components.each{ val ->
                            val = val.toLowerCase()
                            Archive = Archive.toLowerCase()
                            val_tag = GetTagFromJson(val)
                            println val
                            switch (val) {
                                case "mariadb":
                                    println("The value of a is mariadb");
                                    docker ('mariadb', 'MS_Package/', val_tag, Archive)
                                    break;
                                case "configapi":
                                        pullingCenterityImages('registry.centerity.com/centerity/ms/', [[ImageName: "configurationv5", ImageTag: val_tag, NewImageName: "configapi", NewImageTag: val_tag, OldImageTag: "latest-develop"]])
                                    break;
                                case "sqlrunner":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "sqlrunner", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest"]])
                                    break;
                                case "enricher":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "enricher", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest-develop"]])
                                    break;
                                case "correlator":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "correlator", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest-develop"]])
                                    break;
                                case "snmptrapd":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "snmptrapd", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest-develop"]])
                                    break; 
                                case "api":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "api-ui", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest"]])
                                    break; 
                                case "ui":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "ui", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest"]])
                                    break;
                                case "bsm":
                                        pullingCenterityImages('registry.centerity.com/centerity/', [[ImageName: "bsm", ImageTag: val_tag, NewImageTag: val_tag, OldImageTag: "latest"]])
                                    break; 
                                case "timerider":
                                    println("The value of a is timerider");
                                    if (cluster == false)
                                    {
                                        dockerCluster ('cluster', 'opt/centerity/', CLUSTER_VERSION, Archive)  
                                        cluster = true 
                                    }
                                    dockerfiles ('timerider', 'cluster', 'opt/centerity/', CLUSTER_VERSION)
                                    dockerCluster ('timerider', 'opt/centerity/', val_tag, Archive)
                                    break;
                                case "notifier":
                                    println("The value of a is notifier");
                                    if (cluster == false)
                                    {
                                        dockerCluster ('cluster', 'opt/centerity/', CLUSTER_VERSION, Archive)  
                                        cluster = true 
                                    }
                                    dockerfiles ('notifier', 'cluster', 'opt/centerity/', CLUSTER_VERSION)
                                    dockerCluster ('notifier', 'opt/centerity/', val_tag, Archive)
                                    break;
                                case "cs":
                                    println("The value of a is cs");
                                    docker ('cs', 'MS_Package/', val_tag, Archive)
                                    break;
                                case "queuehandler":
                                    println("The value of a is queuehandler");
                                    docker ('queuehandler', 'MS_Package/', val_tag, Archive)
                                    break;
                                case "transformer":
                                    println("The value of a is transformer");
                                    docker ('transformer', 'MS_Package/', val_tag, Archive)
                                    break;
                                case "dealer":
                                    println("The value of a is dealer");
                                    dockerCluster ('dealer', 'opt/centerity/dealer', val_tag, Archive)
                                    dockerfiles ('dealer-httpd', 'dealer', 'opt/centerity/dealer', val_tag)
                                    dockerfiles ('dealer-jsonapi', 'dealer', 'opt/centerity/dealer', val_tag)
                                    dockerfiles ('dealer-runnerscheduler', 'dealer', 'opt/centerity/dealer', val_tag)
                                    dockerfiles ('db-sync', 'dealer', 'opt/centerity/dealer', val_tag)
                                    dockerCluster ('dealer-httpd', 'opt/centerity/dealer', val_tag, Archive)
                                    dockerCluster ('dealer-jsonapi', 'opt/centerity/dealer', val_tag, Archive)
                                    dockerCluster ('dealer-runnerscheduler', 'opt/centerity/dealer', val_tag, Archive)
                                    dockerCluster ('db-sync', 'opt/centerity/dealer', val_tag, Archive)
                                    break;
                                }
                           }
                        }   
            
                    }  
                }
            }
            if (Archive == "yes"){
                stage('Publish to Nexus'){
                    withCredentials([usernamePassword(credentialsId: 'nexusProxMox', usernameVariable: 'NEXUS_CREDS_USR_PROXMOX', passwordVariable: 'NEXUS_CREDS_PSW_PROXMOX')]) {
                        withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_CREDS_USR', passwordVariable: 'NEXUS_CREDS_PSW')]) {
                            // Download required Artifact components ("k8s"/"app-infra"/"app")
                            if (Artifact.contains("k8s")) {
                                dir('devops'){
                                checkout([$class: 'GitSCM',
                                    branches: [[name: '*/deliver_to_harris_v5.1_containerized']],
                                    depth: 1,
                                    extensions: [[$class: 'CloneOption', timeout: 180, shallow: true, honorRefspec: true, noTags: true], [$class: 'CleanBeforeCheckout']], 
                                    userRemoteConfigs: [[credentialsId: 'Gitlab_Devops_Token', usernameVariable: 'GIT_USR', passwordVariable: 'GIT_PSW', refspec: '+refs/heads/deliver_to_harris_v5.1_containerized:refs/remotes/origin/deliver_to_harris_v5.1_containerized', url: 'https://gitlab.com/centerity/devops.git']]])
                                }

                                sh """
                                    rm -rf devops/.git/
                                    curl -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW -X GET http://192.168.2.47:8081/repository/Files/openebs-images.tar.gz -O  -X GET http://192.168.2.47:8081/repository/Files/longhorn-images.tar.gz -O  -X GET http://192.168.2.47:8081/repository/Files/k10_images.tar.gz -O  -X GET http://192.168.2.47:8081/repository/Files/prometheus-images.tar.gz -O -X GET http://192.168.2.47:8081/repository/Files/fluentd-kibana-images.tar.gz -O -X GET http://192.168.2.47:8081/repository/Files/k10-4.5.9.tgz -O 
                                    mv -v openebs-images.tar.gz longhorn-images.tar.gz k10_images.tar.gz prometheus-images.tar.gz fluentd-kibana-images.tar.gz k10-4.5.9.tgz devops/docker-playbook/files/.
                                    mv image_version.json devops/
                                """
                            }

                            if (Artifact.contains("app-infra")) {
                                docker.withRegistry('https://registry.centerity.com/centerity', 'HARBOR_CREDS') {
                                    pulling('registry.centerity.com/centerity/', infraImages)
                                }
                                LIST = 'app-images/mariadb.tar '
                                for (comp in infraImages) {
                                    if (comp.ImageName == "bitnami/minideb")
                                        a = "app-images/" + comp.ImageName.split('/')[-1] + "-" + comp.ImageTag  + ".tar"
                                    else
                                        a = "app-images/" + comp.ImageName.split('/')[-1] + ".tar"

                                    if ( fileExists(a) )
                                        LIST+="$a "
                                }
                                sh "tar -cvf app-infra-images.tar $LIST --remove-files"
                            }

                            if (Artifact.contains("app")) {
                                sh "tar -cvf app-images.tar app-images/"
                            }

                            //build final artifact and push
                            if (Artifact.contains("k8s")) {
                                 // if archives exist, add them to artifact
                                 sh """
                                     [ ! -f app-infra-images.tar ] || ( gzip app-infra-images.tar ; mv app-infra-images.tar.gz devops/docker-playbook/files/. )
                                     [ ! -f app-images.tar ] || ( gzip app-images.tar ; mv app-images.tar.gz devops/docker-playbook/files/. )
                                 """
                                if (Components.toString() == "[ui, api, CS, BSM, ConfigAPI, QueueHandler, Dealer, MariaDB, Enricher, Correlator, Timerider, Notifier, Transformer, snmptrapd, sqlrunner]") {
                                    // if full list of components - use Fullname of artifact (CenterityApp...)
                                    sh """
                                        tar -cvf CenterityApp_${Package_Version}_${Artifact_ID}.tar devops/
                                        gzip CenterityApp_${Package_Version}_${Artifact_ID}.tar
                                        curl -u jenkins:jenkins --upload-file CenterityApp_${Package_Version}_${Artifact_ID}.tar.gz 192.168.97.111:8081/repository/Files/
                                        curl -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file CenterityApp_${Package_Version}_${Artifact_ID}.tar.gz 192.168.2.47:8081/repository/Files/
                                    """
                                    // if its full version - also push to QA nexus 
                                    sh """
                                        curl --insecure -Lo -v --progress-bar -u $NEXUS_CREDS_USR_PROXMOX:$NEXUS_CREDS_PSW_PROXMOX --upload-file CenterityApp_${Package_Version}_${Artifact_ID}.tar.gz https://23.88.82.252:5443/repository/centerityRepo/home/public/v5.1_containerized/
                                    """
                                } else {
                                    sh """
                                        tar -cvf CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar devops/
                                        gzip  CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar
                                        curl -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar.gz 192.168.2.47:8081/repository/Files/
                                    """
                                }
                            } else {
                                sh """
                                    if [ -f app-infra-images.tar ] && [ -f app-images.tar ]; then
                                        tar --concatenate --file=app-images.tar app-infra-images.tar
                                        mv app-images.tar CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar
                                    elif [ -f app-images.tar ]; then
                                        mv app-images.tar CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar
                                    elif [ -f app-infra-images.tar ]; then
                                        mv app-infra-images.tar CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar
                                    fi

                                    gzip CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar
                                    curl -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file CenterityAppPatch_${Package_Version}_${Artifact_ID}.tar.gz 192.168.2.47:8081/repository/Files/
                                """
                            }
                        }
                    }
                }
            }
        } 
    }
  }
}

catch (e) {
    println(e)
    currentBuild.result = 'FAILURE'
    mailSubject="Job ${env.JOB_NAME} ${currentBuild.currentResult}"
    mailBody="""\
            <p>Job ${env.JOB_NAME} build #${env.BUILD_NUMBER} finished with status ${currentBuild.currentResult}</p>
            <p>More info at: ${env.BUILD_URL}</p>
        """
    emailext mimeType: 'text/html', 
            subject: mailSubject,
            body: mailBody,
            recipientProviders: [[$class: 'RequesterRecipientProvider']]
}


def docker (comp, path, version, archive) {
    sh """
    docker build -t ${comp}:${version} -f ${path}/${comp}.Dockerfile ${path} --no-cache 
     """
    if (archive == "yes"){ 
        sh """
        docker save ${comp}:${version} -o app-images/${comp}.tar
        """
    }
    docker.withRegistry('https://registry.centerity.com/centerity', 'HARBOR_CREDS') {
        sh """
        docker tag ${comp}:${version} registry.centerity.com/centerity/${comp}:${version}
        docker push registry.centerity.com/centerity/${comp}:${version}
        """
    }
}

def dockerCluster (comp, path, version, archive) {
    docker.withRegistry('https://registry.gitlab.com/centerity/devops', 'DEVOPS-TOKEN-1') {
        sh """
        docker build -t ${comp}:${version} -f ${path}/${comp}.Dockerfile ${path} --no-cache
        docker tag ${comp}:${version} registry.gitlab.com/centerity/devops/${comp}:${version}
        docker push registry.gitlab.com/centerity/devops/${comp}:${version}
        """
    }
    docker.withRegistry('https://registry.centerity.com/centerity', 'HARBOR_CREDS'){
        sh """
            docker tag ${comp}:${version}  registry.centerity.com/centerity/${comp}:${version}
            docker push registry.centerity.com/centerity/${comp}:${version}
        """
    }
    if (archive == "yes" && "${comp}" != "cluster" && "${comp}" != "dealer"){
        sh """
        docker save ${comp}:${version} -o app-images/${comp}.tar
        """
    }
}

def dockerfiles (comp, orig, path, version) {
    sh """
        sed -i '/gitlab\\|registry/d' ${path}/${comp}.Dockerfile
        sed -i '/gitlab\\|registry/d' docker/${comp}.Dockerfile
        sed -i '1 i\\FROM registry.gitlab.com/centerity/devops/${orig}:${version}' ${path}/${comp}.Dockerfile
        sed -i '1 i\\FROM registry.gitlab.com/centerity/devops/${orig}:${version}' docker/${comp}.Dockerfile
        git config --global user.email "ci-user@centerity.com"
        git config --global user.name "CI User"
        git add docker/${comp}.Dockerfile
        git commit docker/${comp}.Dockerfile --allow-empty -m "Edit the tag of ${comp}"
    """
    
    withCredentials([usernamePassword(credentialsId: 'Devops-pipeline', passwordVariable: 'DEVOPS_TOKEN_PSW', usernameVariable: 'DEVOPS_TOKEN_USR')]) {
        sh 'git push https://${DEVOPS_TOKEN_USR}:${DEVOPS_TOKEN_PSW}@gitlab.com/centerity/devops.git HEAD:containerized_build_v5.x'
    }
}

def pulling (gitUrl, imagesArray) {
    // example git url registry.gitlab.com/centerity/devops/

    imagesArray.each { entry ->
        def t = ''
        // docker.withRegistry('https://registry.gitlab.com/centerity/ms/configurationv5', 'ConfigurationV5_Token') {
            // docker.withRegistry('https://registry.gitlab.com/centerity/core/v6/framework', 'Core_FrameWork_Token') {
            sh """
                echo "pulling"
                docker pull ${gitUrl}${entry.ImageName}:${entry.ImageTag}
            """
            if(entry.NewImageName){
                t=entry.NewImageName
                t=t.split('/')[-1]
            if (t == "minideb"){ 
            sh """
            docker tag ${gitUrl}${entry.ImageName}:${entry.ImageTag} ${entry.ImageName}:${entry.ImageTag} 
            docker save ${entry.ImageName}:${entry.ImageTag} -o app-images/${t}-${entry.ImageTag}.tar 
            """
            }else{
                sh """
                    docker tag ${gitUrl}${entry.ImageName}:${entry.ImageTag} ${entry.NewImageName}:${entry.ImageTag}
                    docker save ${entry.NewImageName}:${entry.ImageTag} -o app-images/${t}.tar 
                """
            }

            }else{
                t=entry.ImageName
                t=t.split('/')[-1]
                if (t == "minideb"){ 
                sh """
                docker tag ${gitUrl}${entry.ImageName}:${entry.ImageTag} ${entry.ImageName}:${entry.ImageTag}
                docker save ${entry.ImageName}:${entry.ImageTag} -o app-images/${t}-${entry.ImageTag}.tar 
                """
                }else{
                sh """
                    docker tag ${gitUrl}${entry.ImageName}:${entry.ImageTag} ${entry.ImageName}:${entry.ImageTag}
                    docker save ${entry.ImageName}:${entry.ImageTag} -o app-images/${t}.tar 
                """
                }
            }
            // def t=entry.ImageName
            println t
            sh """
                ls -la 
                ls -la app-images/
            """
            // }
        // }
    }
}

def pullingCenterityImages (gitUrl, imagesArray) {
    // example git url registry.gitlab.com/centerity/devops/
    docker.withRegistry('https://registry.centerity.com/centerity', 'HARBOR_CREDS') {
        imagesArray.each { entry ->
            def t = ''
                sh """
                    echo "pulling"
                    docker pull ${gitUrl}${entry.ImageName}:${entry.ImageTag} || (echo " ${gitUrl}${entry.ImageName}:${entry.ImageTag} not found in registry" && exit 1)
                """
                if(entry.NewImageTag){
                    entry.OldImageTag = entry.ImageTag
                    entry.ImageTag = entry.NewImageTag
                }
                if(entry.NewImageName){
                    t=entry.NewImageName
                    t=t.split('/')[-1]
                    sh """
                        docker tag ${gitUrl}${entry.ImageName}:${entry.OldImageTag} ${entry.NewImageName}:${entry.ImageTag}
                        docker save ${entry.NewImageName}:${entry.ImageTag} -o app-images/${t}.tar
                        docker tag ${entry.NewImageName}:${entry.ImageTag} registry.centerity.com/centerity/${entry.NewImageName}:${entry.ImageTag}
                        docker push registry.centerity.com/centerity/${entry.NewImageName}:${entry.ImageTag}
                    """
                }else{
                    t=entry.ImageName
                    t=t.split('/')[-1]
                    sh """  
                        docker tag ${gitUrl}${entry.ImageName}:${entry.OldImageTag} ${entry.ImageName}:${entry.ImageTag}
                        docker save ${entry.ImageName}:${entry.ImageTag} -o app-images/${t}.tar 
                        docker tag ${entry.ImageName}:${entry.ImageTag} registry.centerity.com/centerity/${entry.ImageName}:${entry.ImageTag}
                        docker push registry.centerity.com/centerity/${entry.ImageName}:${entry.ImageTag}
                    """
                }
                // def t=entry.ImageName
                println t
                sh """
                    ls -la 
                    ls -la app-images/
                """
                // }
            // }
        }
    }
}

def validateJsonFileParameter(components) {
    if ( Artifact == "k8s") {
        // if only k8s, no need to process
        return
    }
    // Check that file parameter exist and save to file
    sh '''
        if [ -z "$IMAGE_VERSION" ];
            then
                echo "=================Uploaded image_version file is empty================="
                exit 1;
            else
                echo $IMAGE_VERSION | base64 -d > image_version.json
                cat image_version.json
            fi
       '''

    // Check json syntax of file is correct
    try {
        readJSON file: 'image_version.json'
    } catch (e) {
        echo "Caught: ${e} JSON  syntax not valid."
        currentBuild.result = 'FAILURE'
    }

    // Check selected components exist in file parameter
    str = Components.split(',')
    str.each{ val ->
            GetTagFromJson(val)
            }

    println("Attached json file valid")
}

def GetTagFromJson (component) {
        if (Artifact.toString() == "[k8s]") {
            return
        }
        // Retrieve image_list array from json file and get tag for specific component
        component = component.toLowerCase()
        def cfg = readJSON file: 'image_version.json'
        def image_list = JsonOutput.toJson(cfg['image_list'])
        def list = new JsonSlurper().parseText( image_list.toLowerCase() )
        def tag = null;

        for (entry in list) {
                if ( entry[component] != null) {
                        tag = entry[component]
                }
        }

        if( tag != null) {
            return tag
        }
        else {
            println("Selected component  " + component + "  not found in given json file parameter")
            sh 'exit 1'
        }
}
