pipeline {

 agent any

 environment {
   DOCKER_IMAGE="yourdockerhub/devops-node"
 }

 stages {

  stage('Checkout') {
   steps {
     checkout scm
   }
  }

  stage('Terraform') {

   when {
     branch 'devops'
   }

   steps {

    dir('terraform') {

     sh 'terraform init'
     sh 'terraform apply -auto-approve'

    }

   }
  }

  stage('Ansible') {

   when {
     branch 'devops'
   }

   steps {

    dir('ansible') {

      sh 'ansible-playbook -i inventory install-docker.yml'

    }

   }
  }

  stage('Build Docker Image') {

   when {
     branch 'dev'
   }

   steps {

    sh 'docker build -t $DOCKER_IMAGE .'

   }
  }

  stage('Push Docker Image') {

   when {
     branch 'dev'
   }

   steps {

    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {

      sh 'docker login -u $USER -p $PASS'
      sh 'docker push $DOCKER_IMAGE'

    }

   }
  }

  stage('Deploy To EC2') {

   when {
     branch 'dev'
   }

   steps {

    sh '''
    ssh ec2-user@EC2_PUBLIC_IP << EOF
    docker pull $DOCKER_IMAGE
    docker stop nodeapp || true
    docker rm nodeapp || true
    docker run -d -p 80:3000 --name nodeapp $DOCKER_IMAGE
    EOF
    '''

   }
  }

 }

}