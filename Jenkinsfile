import groovy.sql.Sql
pipeline {
   agent
    node{
        def conn = Sql.newInstance("jdbc:mysql://127.0.0.1:3306/new_schema?user=root&useUnicode=" +
                "true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC",  "com.mysql.jdbc.Driver")
        def rows = conn.rows("select username from users LIMIT 10")
        assert rows.size() == 10
        println rows.join('\n')
    }
   tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "M3"
   }
   parameters {
    gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'BRANCH', type: 'PT_BRANCH'
   }

   stages {
      stage('Build') {
         steps {
            // Get some code from a GitHub repository
            git branch: '''${params.BRANCH}''', url: 'https://github.com/AnnSchiretz/connectToDBAndAPI.git'

            // Run Maven on a Unix agent.
            sh "mvn -Dmaven.test.failure.ignore=true clean test"

            // To run Maven on a Windows agent, use
            // bat "mvn -Dmaven.test.failure.ignore=true clean package"
         }

         post {
            // If Maven was able to run the tests, even if some of the test
            // failed, record the test results and archive the jar file.
            success {
               junit '**/target/surefire-reports/TEST-*.xml'
            }
         }
      }
        stage('reports') {
            steps {
                script {
                        allure([
                                includeProperties: false,
                                properties: [],
                                reportBuildPolicy: 'ALWAYS',
                                results: [[path: 'target/allure-results']]
                        ])
                 }
             }
        }
   }
}