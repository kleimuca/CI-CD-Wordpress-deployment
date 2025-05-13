pipeline {
  agent any

  environment {
    KUBECONFIG = credentials('kubeconfig-secret')  // Add this secret in Jenkins
  }

  stages {
    stage('Checkout Code') {
      steps {
        git 'https://github.com/your-org/wordpress-theme-cicd'
      }
    }

    stage('Find WordPress Pod') {
      steps {
        script {
          env.WP_POD = sh(
            script: "kubectl get pods -n wordpress -l app.kubernetes.io/component=wordpress -o jsonpath='{.items[0].metadata.name}'",
            returnStdout: true
          ).trim()
        }
      }
    }

    stage('Copy Theme to Pod') {
      steps {
        sh """
        kubectl cp custom-wp/wp-content/themes/cyberflow-theme \$WP_POD:/bitnami/wordpress/wp-content/themes/cyberflow-theme -n wordpress
        """
      }
    }

    stage('Restart Apache (optional)') {
      steps {
        sh """
        kubectl exec -n wordpress \$WP_POD -- apachectl -k restart || true
        """
      }
    }
  }
}
