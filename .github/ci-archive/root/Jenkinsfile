<![CDATA[
// Source: https://github.com/jenkinsci/pipeline-examples (MIT License)
// Test case: Config file provider and complex scripts
node {
    stage('Setup') {
        configFileProvider([configFile(fileId: 'global-settings', variable: 'MAVEN_SETTINGS')]) {
            sh 'echo "Using Maven settings: $MAVEN_SETTINGS"'
            sh 'cp $MAVEN_SETTINGS ~/.m2/settings.xml'
        }
        
        configFileProvider([configFile(fileId: 'app-config', targetLocation: 'config/app.properties')]) {
            sh 'cat config/app.properties'
        }
    }

    stage('Checkout') {
        checkout scm
    }

    stage('Environment Setup') {
        withEnv(["JAVA_HOME=${tool 'jdk-11'}", "PATH+MAVEN=${tool 'maven-3.6.3'}/bin:${env.JAVA_HOME}/bin"]) {
            sh '''
                echo "Java version:"
                java -version
                echo "Maven version:"
                mvn -version
                
                # Create directories
                mkdir -p target/reports
                mkdir -p target/logs
                
                # Setup test data
                echo "test.data.path=./test-data" > test.properties
                echo "test.database.url=jdbc:h2:mem:testdb" >> test.properties
            '''
        }
    }

    stage('Build and Test') {
        withEnv(["MAVEN_OPTS=-Xmx2048m -XX:MaxPermSize=512m"]) {
            sh '''
                mvn clean compile \
                    -Dmaven.test.failure.ignore=true \
                    -Dcheckstyle.skip=false \
                    -Dfindbugs.skip=false
                    
                mvn test \
                    -Dmaven.test.failure.ignore=true \
                    -Dsurefire.useFile=false \
                    -Dtest.properties.file=./test.properties
            '''
        }
        
        publishTestResults testResultsPattern: '**/target/surefire-reports/*.xml'
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/site',
            reportFiles: 'index.html',
            reportName: 'Maven Site Report'
        ])
    }
}
    ]]>