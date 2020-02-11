if (env.CHANGE_ID) {
    properties([
        buildDiscarder(logRotator(numToKeepStr: '10')),
    ])
}
else {
    properties([
        buildDiscarder(logRotator(numToKeepStr: '96')),
        pipelineTriggers([[$class:"SCMTrigger", scmpoll_spec:"H/10 * * * *"]]),
    ])
}

pipeline {
  agent {
    docker {
      args '--entrypoint=""'
      image 'hashicorp/packer:1.5.1'
    }
  }

  stages {
    stage('Validate') {
      steps {
        sh 'packer validate --var-file validate-vars.json ubuntu-18-agent.json'
      }
    }

    stage('Build') {
      environment {
        AZURE_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        AZURE_CLIENT_ID = credentials('azure-client-id')
        AZURE_CLIENT_SECRET = credentials('azure-client-secret')
      }

      when {
        branch 'master'
      }

      steps {
        sh """
          packer build \
            --var location="East US" \
            --var resource_group_name="packer-images" \
            --var subscription_id="$AZURE_SUBSCRIPTION_ID" \
            --var client_id="$AZURE_CLIENT_ID" \
            --var client_secret="$AZURE_CLIENT_SECRET" \
            ubuntu-18-agent.json
        """
      }
    }
  }
}
