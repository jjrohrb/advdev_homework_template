#!groovy
podTemplate(
  label: "skopeo-pod",
  cloud: "openshift",
  inheritFrom: "maven",
  containers: [
    containerTemplate(
      name: "jnlp",
      image: "docker-registry.default.svc:5000/8a0c-jenkins/jenkins-agent-appdev",
      resourceRequestMemory: "1Gi",
      resourceLimitMemory: "2Gi",
      resourceRequestCpu: "1",
      resourceLimitCpu: "2"
    )
  ]
) {
  node('skopeo-pod') {
    // Define Maven Command to point to the correct
    // settings for our Nexus installation
    def mvnCmd = "mvn -s ../nexus_settings.xml"

    // Checkout Source Code.
    stage('Checkout Source') {
      checkout scm
    }

    // Build the Tasks Service
    dir('openshift-tasks') {
      // The following variables need to be defined at the top level
      // and not inside the scope of a stage - otherwise they would not
      // be accessible from other stages.
      // Extract version from the pom.xml
      def version = getVersionFromPom("pom.xml")

      // TBD Set the tag for the development image: version + build number
      def devTag  = "${version}-" + currentBuild.number
      // Set the tag for the production image: version
      def prodTag =  "${version}"
      def devProject  = "8a0c-tasks-dev"
      def prodProject = "8a0c-tasks-prod"

      // Using Maven build the war file
      // Do not run tests in this step
      stage('Build war') {
        echo "Building version ${devTag}"
        //Execute Maven Build
        sh "${mvnCmd}  clean package -DskipTests=true" 
      }

      //The next two stages should run in parallel
      stage("Parallel Stage") {
        parallel (
          // Using Maven run the unit tests
          "Unit Tests": {
            echo "Running Unit Tests"
            //Execute Unit Tests
            sh "${mvnCmd}  test"
          },
          // Using Maven to call SonarQube for Code Analysis
          "Code Analysis": {
            echo "Running Code Analysis"
            //Execute Sonarqube Tests
            sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube-gpte-hw-cicd.apps.na311.openshift.opentlc.com -Dsonar.projectName=${JOB_BASE_NAME} -Dsonar.projectVersion=${devTag}"
          },
          failFast: true
        )
      }

      // Publish the built war file to Nexus
      stage('Publish to Nexus') {
        echo "Publish to Nexus"
        //Publish to Nexus
        sh "${mvnCmd} deploy -DskipTests=true -DaltDeploymentRepository=nexus::default::http://nexus3.gpte-hw-cicd.svc.cluster.local:8081/repository/releases"
      }

      // Build the OpenShift Image in OpenShift and tag it.
      stage('Build and Tag OpenShift Image') {
        echo "Building OpenShift container image tasks:${devTag}"
        //Build Image, tag Image
        // openshift.withCluster() {
        //   openshift.withProject("${devProject}") {
        //     openshift.selector("bc", "tasks").startBuild("--from-file=./target/openshift-tasks.war", "--wait=true")
        //     // OR use the file you just published into Nexus:
        //     // "--from-file=http://nexus3.gpte-hw-cicd.svc.cluster.local:8081/repository/releases/org/jboss/quickstarts/eap/tasks/${version}/tasks-${version}.war"
        //     openshift.tag("tasks:latest", "tasks:${devTag}")
        //   }
        // }
      }

      // Deploy the built image to the Development Environment.
      stage('Deploy to Dev') {
        echo "Deploying container image to Development Project"
        //Deploy to development Project
        //      Set Image, Set VERSION
        //      Make sure the application is running and ready before proceeding
        // openshift.withCluster() {
        //    openshift.withProject("${devProject}") {
        //      openshift.set("image", "dc/tasks", "tasks=image-registry.openshift-image-registry.svc:5000/${devProject}/tasks:${devTag}")
          
        //      openshift.selector('configmap', 'tasks-config').delete()
        //      def configmap = openshift.create('configmap', 'tasks-config', '--from-file=./configuration/application-users.properties', '--from-file=./configuration/application-roles.properties' )

        //      openshift.selector("dc", "tasks").rollout().latest();
       
        //      def dc = openshift.selector("dc", "tasks").object()
        //      def dc_version = dc.status.latestVersion
        //      def rc = openshift.selector("rc", "tasks-${dc_version}").object()

        //      echo "Waiting for ReplicationController tasks-${dc_version} to be ready"
        //      while (rc.spec.replicas != rc.status.readyReplicas) {
        //       sleep 5
        //       rc = openshift.selector("rc", "tasks-${dc_version}").object()
        //     }
        //   }
        // } 
      }

      // Copy Image to Nexus container registry
      stage('Copy Image to Nexus container registry') {
        echo "Copy image to Nexus container registry"

        //Copy image to Nexus container registry
        // sh("skopeo copy --src-tls-verify=false --dest-tls-verify=false --src-creds openshift:\$(oc whoami -t) --dest-creds admin:redhat docker://image-registry.openshift-image-registry.svc.cluster.local:5000/${devProject}/tasks:${devTag} docker://nexus3-registry.gpte-hw-cicd.svc.cluster.local:5000/tasks:${devTag}")
        // //Tag the built image with the production tag.
        // openshift.withCluster() {
        //    openshift.withProject("${prodProject}") {
        //    	openshift.tag("${devProject}/tasks:${devTag}", "${devProject}/tasks:${prodTag}")
        //    }
        //  }
      }

      // Blue/Green Deployment into Production
      // -------------------------------------
      def destApp   = "tasks-green"
      def activeApp = ""

      stage('Blue/Green Production Deployment') {
        // TBD: Determine which application is active
        //      Set Image, Set VERSION
        //      Deploy into the other application
        //      Make sure the application is running and ready before proceeding
        // openshift.withCluster() {
        //    openshift.withProject("${prodProject}") {
        //     activeApp = openshift.selector("route", "tasks").object().spec.to.name
        //     if (activeApp == "tasks-green") {
        //      destApp = "tasks-blue"
        //     }
        //     echo "Active Application:      " + activeApp
        //     echo "Destination Application: " + destApp
            
        //     // Update the Image on the Production Deployment Config
        //     def dc = openshift.selector("dc/${destApp}").object()

        //     dc.spec.template.spec.containers[0].image="image-registry.openshift-image-registry.svc:5000/${devProject}/tasks:${prodTag}"
            
        //     openshift.apply(dc)
            
        //     // Update Config Map in change config files changed in the source
        //     openshift.selector("configmap", "${destApp}-config").delete()
        //     def configmap = openshift.create("configmap", "${destApp}-config", "--from-file=./configuration/application-users.properties", "--from-file=./configuration/application-roles.properties" )

        //     // Deploy the inactive application.
        //     openshift.selector("dc", "${destApp}").rollout().latest();

        //     // Wait for application to be deployed
        //     def dc_prod = openshift.selector("dc", "${destApp}").object()
        //     def dc_version = dc_prod.status.latestVersion
        //     def rc_prod = openshift.selector("rc", "${destApp}-${dc_version}").object()
        //     echo "Waiting for ${destApp} to be ready"
        //     while (rc_prod.spec.replicas != rc_prod.status.readyReplicas) {
        //       sleep 5
        //       rc_prod = openshift.selector("rc", "${destApp}-${dc_version}").object()
        //     }
        //   }
        //  } 
      }

      stage('Switch over to new Version') {
        echo "Switching Production application to ${destApp}."
        //Execute switch
        // openshift.withCluster() {
        //    openshift.withProject("${prodProject}") {
        //      def route = openshift.selector("route", "tasks").object()
        //      route.spec.to.name = "${destApp}"
        //      openshift.apply(route)
        //    }
        // }
      }
    }
  }
}

// Convenience Functions to read version from the pom.xml
// Do not change anything below this line.
// --------------------------------------------------------
def getVersionFromPom(pom) {
  def matcher = readFile(pom) =~ '<version>(.+)</version>'
  matcher ? matcher[0][1] : null
}