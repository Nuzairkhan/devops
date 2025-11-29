pipeline {
   agent any

   environment {
      SONAR_HOST_URL = "http://44.223.27.240:9000"
      SONAR_TOKEN = "squ_ef68f131502f5878f3a9c97d8e88c65100d5d0d5"
      NEXUS_URL = "http://44.223.27.240:8081/repository/lms/"
      NEXUS_USER = "admin"
      NEXUS_PASS = "lms12345"
   }

   stages {

       /* -------------------- 1️⃣ SONARQUBE -------------------- */
       stage('Code Quality') {
           steps {
               echo 'Sonar Analysis Started'
               sh '''
                   sudo docker run --rm \
                   -e SONAR_HOST_URL="${SONAR_HOST_URL}" \
                   -e SONAR_TOKEN="${SONAR_TOKEN}" \
                   -v "$WORKSPACE:/usr/src" \
                   sonarsource/sonar-scanner-cli \
                   -Dsonar.projectKey=lms
               '''
               echo 'Sonar Analysis Completed'
           }
       }

       /* -------------------- 2️⃣ BUILD -------------------- */
       stage('Build LMS') {
           steps {
               echo 'LMS Build Started'
               sh '''
                   npm install
                   npm run build
               '''
               echo 'LMS Build Completed'
           }
       }

       /* -------------------- 3️⃣ PUBLISH ARTIFACT TO NEXUS -------------------- */
       stage('Publish LMS') {
           steps {
               script {
                   def packageJson = readJSON file: 'package.json'
                   def version = packageJson.version
                   echo "Version: ${version}"

                   sh """
                       zip lms-${version}.zip -r build
                       curl -v -u ${NEXUS_USER}:${NEXUS_PASS} \
                       --upload-file lms-${version}.zip \
                       ${NEXUS_URL}
                   """
               }
           }
       }

       /* -------------------- 4️⃣ DEPLOY TO NGINX -------------------- */
       stage('Deploy LMS') {
           steps {
               script {
                   def packageJson = readJSON file: 'package.json'
                   def version = packageJson.version
                   echo "Deploying version: ${version}"

                   sh """
                       curl -u ${NEXUS_USER}:${NEXUS_PASS} \
                       -X GET '${NEXUS_URL}lms-${version}.zip' \
                       --output lms-${version}.zip

                       sudo rm -rf /var/www/html/*
                       sudo unzip -o lms-${version}.zip
                       sudo cp -r build/* /var/www/html/
                   """
               }
           }
       }

       /* -------------------- 5️⃣ CLEAN WORKSPACE -------------------- */
       stage('Clean Up Workspace') {
           steps {
               echo 'Cleaning Work Space'
               cleanWs()
           }
       }
   }
}
