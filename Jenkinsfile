node{
    stage('Git Checkout'){
        cleanWs()
        git poll: true, url:'https://github.com/dorian95/insurance-project-demo.git'
    }
    
    stage('Test & Package'){
        sh 'mvn package'
    }
    
    stage('Docker build'){
        sh 'docker build -t irgaliyev/insure-me:1.0 .'
    }
    
    stage('Docker push'){
        withCredentials([usernamePassword(credentialsId: 'docker_creds', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
            sh "docker login -u ${docker_user} -p ${docker_pass}"
            sh "docker push irgaliyev/insure-me:1.0"
        }
    }
    
    stage("Configure Test Server"){
        cleanWs()
        git branch: 'main', poll: false, url: 'https://github.com/dorian95/ansible-insure-me.git'
        ansiblePlaybook become: true, credentialsId: 'ubuntu', disableHostKeyChecking: true, installation: 'ansible', 
            inventory: 'hosts', 
            playbook: 'playbook-test-server.yml'
    }
    
    stage('Test Server Deployment')
    {
        withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu', keyFileVariable: 'ubuntu_file', usernameVariable: 'ubuntu')]) {
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${test_server_name} docker stop insure-me"
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${test_server_name} docker rm insure-me"
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${test_server_name} docker run -itd -p 8080:8081 --name insure-me irgaliyev/insure-me:1.0"
            
        }
    }
    
    stage('Selenium Test'){
         cleanWs()
        git poll: false, branch: 'main', url: 'https://github.com/dorian95/insure-me-selenium.git'  
        sh 'mvn package'
        sh 'sudo java -ea -jar target/insure-me-selenium-1.0-SNAPSHOT.jar'
    }
    
     stage("Configure Prod Server"){
        cleanWs()
        git branch: 'main', poll: false, url: 'https://github.com/dorian95/ansible-insure-me.git'
        ansiblePlaybook become: true, credentialsId: 'ubuntu', disableHostKeyChecking: true, installation: 'ansible', 
            inventory: 'hosts', 
            playbook: 'playbook-prod-server.yml'
    }
    
    stage('Prod Server Deployment'){
        withCredentials([sshUserPrivateKey(credentialsId: 'ubuntu', keyFileVariable: 'ubuntu_file', usernameVariable: 'ubuntu')]) {
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${prod_server_name} docker stop insure-me || true"
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${prod_server_name} docker rm insure-me || true"
            sh "ssh -o StrictHostKeyChecking=no -i ${ubuntu_file} ubuntu@${prod_server_name} docker run -itd -p 8080:8081 --name insure-me irgaliyev/insure-me:1.0"
            
        }
    }
    
}
