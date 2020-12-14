pipeline {
      environment {
        registry = "10.1.0.60:8083"
        registryCredential = 'nexus'
         }
    agent any
    stages {
        //stage('Cleaning...'){
        //    steps{                
                //env.BUILDTOCLEAN = ${currentBuild.previousBuild.getNumber()}
                //echo "Will be implemented"
                //sh "docker tag \$(docker images | grep \${SCOPE}\${BUILD_NUMBER}  | awk '{print \$3}' 10.1.0.60:8083/shark:\${SCOPE}"
                //sh "docker images | grep \${SCOPE}\${BUILDTOCLEAN}  | awk '{print \$3}"
                //sh "echo Y | docker system prune -a"
           // }
       // }
        stage('Preparing to build:'){
            steps{
                script{
                    if (env.SCOPE == 'prd'){
                        echo 'PROD:'
                        env.RELEASE_PROD = input message: 'Select YES or NO to deploy on production...', ok: 'Release!', 
                        parameters: [choice(name: 'RELEASE_PROD', choices: ['yes', 'no'], description: 'Go ahead to deploy on prod (shark.alegra.com.br)?')]
                        if (env.RELEASE_PROD == 'yes') {
                            echo "You have accepted PROD."
                            git branch: 'master', url: 'https://github.com/luizhpriotto/amcom/'
                        }
                        else{
                            git branch: env.BRANCH, url: 'https://github.com/luizhpriotto/amcom/'
                        }
                    }
                } 
            }
        }
        stage('oO building Oo'){
            steps{
                script{                               
                    dir("node-project") {
                        dockerImage = docker.build "${registry}/shark:${SCOPE}${BUILD_NUMBER}"
                        docker.withRegistry( 'http://10.1.0.60:8083/', 'nexus') { 
                            dockerImage.push()
                            echo "Tagging new default image ${SCOPE}."
                            dockerImage.push("${SCOPE}") 
                        }
                    }
                }
            }
        }
        stage('Publishing Service...'){
                steps {
                    script {
                        if (env.SCOPE == 'prd'){
                                sh "docker service update shark_${SCOPE} --resolve-image=always --with-registry-auth --image=10.1.0.60:8083/shark:${SCOPE}${BUILD_NUMBER}"
                            }  
                        else{
                            env.RELEASE_QAS = input message: 'Is it an upate?', ok: 'Release!', 
                            parameters: [choice(name: 'RELEASE_QAS', choices: ['yes', 'no'], description: 'Go ahead to deploy on QAS (sharkh.alegra.com.br)?')]
                            if (env.RELEASE_QAS == 'yes') {
                                echo "updating..."
                                sh "docker service update shark_${SCOPE} --resolve-image=always --with-registry-auth --image=10.1.0.60:8083/shark:${SCOPE}${BUILD_NUMBER}"
                            }
                            else{
                                echo "creating.."
                                sh "docker service create --name shark_${SCOPE}${BUILD_NUMBER} --network shark_qas --with-registry-auth \
                                --label traefik.enable=true \
                                --label traefik.docker.network=shark_${SCOPE}${BUILD_NUMBER} \
                                --label traefik.http.middlewares.sharkh${SCOPE}${BUILD_NUMBER}-mid.redirectscheme.scheme=https \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-web.middlewares=sharkh${SCOPE}${BUILD_NUMBER}-mid \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-web.rule=Host`sharkh${SCOPE}${BUILD_NUMBER}.alegra.com.br` \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-web.entrypoints=web \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-websecure.rule=Host`sharkh${SCOPE}${BUILD_NUMBER}.alegra.com.br` \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-websecure.service=sharkh${SCOPE}${BUILD_NUMBER}-svc \
                                --label traefik.http.services.sharkh${SCOPE}${BUILD_NUMBER}-svc.loadbalancer.server.port=8080 \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-websecure.entrypoints=websecure \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-websecure.tls.certresolver=myresolver \
                                --label traefik.http.routers.sharkh${SCOPE}${BUILD_NUMBER}-websecure.tls=true \
                                ${registry}/shark:${SCOPE}${BUILD_NUMBER}"
                            }    
                        }
                    }
                }
        }
    }
    post {
        success {
            script{
                    if (env.RELEASE_QAS == 'no'){
                        env.FQDN = "shark-${SCOPE}${BUILD_NUMBER}.alegra.com.br"
                        echo "FQDN: ${FQDN}"
                        env.DATA = '{\"type\":\"A\",\"name\":\"'+"${FQDN}"+'\",\"content\":\"177.91.38.105\",\"ttl\":120,\"priority\":10,\"proxied\":false}'
                        echo "Creating the DNS to access de aplication on ${SCOPE}..."                
                        sh 'curl -X POST "https://api.cloudflare.com/client/v4/zones/cfb6a7f79905716da43fa085422ffcb3/dns_records" \
                            -H "X-Auth-Email: luiz_priotto@castrolanda.coop.br" \
                            -H "X-Auth-Key: 34bc1d0cde15163b7fde296322d0e54e05c4c" \
                            -H "Content-Type: application/json" \
                            --data $DATA'
                        echo "https://shark-${SCOPE}${BUILD_NUMBER}.alegra.com.br"
                    }
                    else if (env.RELEASE_QAS == 'yes'){
                        echo "https://sharkh.alegra.com.br"
                    }
                    else{
                         echo "https://shark.alegra.com.br"
                    }             
            }
        }
    }
}