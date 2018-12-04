pipeline {
  agent {
    label 'mule-builder'
  }
  environment {
    DEPLOY_CREDS = credentials('deploy-anypoint-user')
    MULE_VERSION = '4.1.4-AM'
    BG = "1Platform\\Retail\\Marketing"
    WORKER = "Small"
  }
  stages {
    stage('Prepare') {
      steps {
        configFileProvider([configFile(fileId: "${BRANCH_NAME}-proc-notification-api", replaceTokens: true, targetLocation: './src/main/resources/config/configuration.yaml')]) {
          sh 'echo "Branch NAME: $BRANCH_NAME"'
        }
        withCredentials([file(credentialsId: 'self-signed-keystore.jks', variable: 'KEYSTORE_PATH')]){  
            sh 'cp "$KEYSTORE_PATH" ./src/main/resources/'
            sh 'echo "Keystore copied."'
        }
      }
    }
    stage('Build') {
      steps {
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh 'mvn -B clean package -DskipTests'
          }
      }
    }

    stage('Test') {
      steps {
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh "mvn -B test"
          }
      }
    }

    stage('Deploy Development') {
      when {
        branch 'develop'
      }
      environment {
        ENVIRONMENT = 'Development'
        ANYPOINT_ENV = credentials('DEV_ANYPOINT_MARKETING')
        APP_CLIENT_CREDS = credentials('develop-api-mgr-proc-notification-api')
        APP_NAME = 'dev-nto-notification-api-v1'
      }
      steps {
        sh 'mvn -V -B -DskipTests deploy -DmuleDeploy -Dmule.version=$MULE_VERSION -Danypoint.username=$DEPLOY_CREDS_USR -Danypoint.password=$DEPLOY_CREDS_PSW -Dcloudhub.app=$APP_NAME -Dcloudhub.environment=$ENVIRONMENT -Denv.ANYPOINT_CLIENT_ID=$ANYPOINT_ENV_USR -Denv.ANYPOINT_CLIENT_SECRET=$ANYPOINT_ENV_PSW -Dcloudhub.bg=$BG -Dcloudhub.worker=$WORKER -Denv.APP_CLIENT_ID=$APP_CLIENT_CREDS_USR -Denv.APP_CLIENT_SECRET=$APP_CLIENT_CREDS_PSW'
      }
    }
    stage('Deploy Production') {
        when {
          branch 'master'
        }
        environment {
          ENVIRONMENT = 'Production'
          ANYPOINT_ENV = credentials('PRD_ANYPOINT_MARKETING')
          APP_CLIENT_CREDS = credentials('master-api-mgr-proc-notification-api')
          APP_NAME = 'nto-notification-api-v1'
        }
        steps {
          sh 'mvn -V -B -DskipTests deploy -DmuleDeploy -Dmule.version=$MULE_VERSION -Danypoint.username=$DEPLOY_CREDS_USR -Danypoint.password=$DEPLOY_CREDS_PSW -Dcloudhub.app=$APP_NAME -Dcloudhub.environment=$ENVIRONMENT -Denv.ANYPOINT_CLIENT_ID=$ANYPOINT_ENV_USR -Denv.ANYPOINT_CLIENT_SECRET=$ANYPOINT_ENV_PSW -Dcloudhub.bg=$BG -Dcloudhub.worker=$WORKER -Denv.APP_CLIENT_ID=$APP_CLIENT_CREDS_USR -Denv.APP_CLIENT_SECRET=$APP_CLIENT_CREDS_PSW'
        }
    }
  }

  tools {
    maven 'M3'
  }
}