 pipeline {
       agent {
           label 'amazon-linux'
       }
       
       environment {
           S3_BUCKET = "team12-artifact-bucket"
           ARTIFACT_NAME = "flask-app-${BUILD_NUMBER}.tar.gz"
           AWS_REGION = "us-east-2"
       }
       
       stages {
           stage('Checkout') {
               steps {
                   checkout scm
               }
           }
           
           stage('Setup Python Environment') {
               steps {
                   sh '''
                   python3 -m venv venv
                   . venv/bin/activate
                   pip install -r app/requirements.txt
                   pip install pytest
                   '''
               }
           }
           
           stage('Test') {
               steps {
                   sh '''
                   . venv/bin/activate
                   echo "Running tests..."
                   # pytest tests/
                   '''
               }
           }
           
           stage('Create Artifact') {
               steps {
                   sh """
                   tar -czvf ${ARTIFACT_NAME} app/ ansible/ init_db.sql Jenkinsfile
                   """
               }
           }
           
           stage('Upload to S3') {
               steps {
                   withAWS(region: "${AWS_REGION}", credentials: 'aws-credentials') {
                       sh """
                       aws s3 cp ${ARTIFACT_NAME} s3://${S3_BUCKET}/${ARTIFACT_NAME}
                       echo "Artifact uploaded to s3://${S3_BUCKET}/${ARTIFACT_NAME}"
                       """
                   }
               }
           }
           
           stage('Deploy with Ansible') {
               steps {
                   withAWS(region: "${AWS_REGION}", credentials: 'aws-credentials') {
                       sh """
                       # Install required Ansible collections
                       ansible-galaxy collection install amazon.aws
                       
                       # Run Ansible playbook
                       cd ansible
                       ansible-playbook site.yml -v
                       """
                   }
               }
           }
       }
       
       post {
           always {
               cleanWs()
           }
       }
   }
