pipeline {
  agent none
  triggers {
    cron ('H H(0-2) * * *')
    pollSCM('H/10 * * * *')
  }
  stages {
    stage('Parallel Build') {
      //parallel {
        matrix {
          axes {
            axis {
              name 'configurator'
              values 'make', 'cmake'
            }
          } //axes
          stages {
          stage('caffeine - Build') {
            agent { node 'caffeine.icl.utk.edu' }
            steps {
                sh '''
                  hostname && pwd

                  source /home/jmfinney/spack/share/spack/setup-env.sh
                  spack load gcc@6.4.0
                  spack load cuda
                  spack load intel-mkl
                  spack load intel-mpi

                  if [ "${configurator}" = "make" ]; then
                    echo "make!!"
                    make config color=no && make CXXFLAGS='-Werror'
                    cd test && ./run_tests.py --xml report.xml
                    cp report.xml ../report_make.xml
                  fi
                  if [ "${configurator}" = "cmake" ]; then
                    echo "cmake!!"
                    spack load cmake
                    mkdir -p build
                    cd build
                    cmake -DCMAKE_INSTALL_PREFIX=/var/lib/jenkins/workspace/jmfinney/testsweeper/sw -DNO_COLOR=TRUE ..
                    make CXXFLAGS='-Werror'
                    cd test && ./run_tests.py --xml report.xml
                    cp report.xml ../../report_cmake.xml
                  fi
                '''
            } //steps
            post {
              always {
                junit '*.xml'
              }
              unstable {
                slackSend channel: '#slate_ci',
                  color: 'warning',
                  message: "Caffeine: ${configurator} - ${currentBuild.fullDisplayName} completed, but has failing tests (<${env.BUILD_URL}|Open>)"
              }
              failure {
                slackSend channel: '#slate_ci',
                  color: 'danger',
                  message: "Caffeine: ${configurator} - ${currentBuild.fullDisplayName} failed.(<${env.BUILD_URL}|Open>)"
                mail to: 'slate-dev@icl.utk.edu',
                  subject: "Failure: ${currentBuild.fullDisplayName}",
                  body: "Caffeine: ${configurator} - See more at ${env.BUILD_URL}"
              }
            } //post
          } //stage - caffeine build
          stage('lips - Build') {
            agent { node 'lips.icl.utk.edu' }
            steps {
              sh '''
                hostname && pwd
                source /home/jmfinney/spack/share/spack/setup-env.sh
                spack load gcc@6.4.0
                spack load cuda
                spack load intel-mkl
                spack load intel-mpi

                if [ "${configurator}" = "make" ]; then
                  make config color=no
                  make config color=no && make CXXFLAGS='-Werror'
                  cd test && ./run_tests.py --xml report.xml
                  cp report.xml ../report_make.xml
                fi
                if [ "${configurator}" = "cmake" ]; then
                  spack load cmake
                  mkdir -p build
                  cd build
                  cmake -DCMAKE_INSTALL_PREFIX=/var/lib/jenkins/workspace/jmfinney/testsweeper/sw -DNO_COLOR=TRUE ..
                  make CXXFLAGS='-Werror'
                  cd test && ./run_tests.py --xml report.xml
                  cp report.xml ../../report_cmake.xml
                fi
              '''
            } //steps
            post {
              always {
                junit '*.xml'
              }
              unstable {
                slackSend channel: '#slate_ci',
                  color: 'warning',
                  message: "Lips: ${configurator} - ${currentBuild.fullDisplayName} completed, but has failing tests (<${env.BUILD_URL}|Open>)"
              }
              failure {
                slackSend channel: '#slate_ci',
                  color: 'danger',
                  message: "Lips: ${configurator} - ${currentBuild.fullDisplayName} failed.(<${env.BUILD_URL}|Open>)"
                mail to: 'slate-dev@icl.utk.edu',
                  subject: "Failure: Lips: ${configurator} - ${currentBuild.fullDisplayName}",
                  body: "See more at ${env.BUILD_URL}"
              }
            } //post
          } //stage - lips build
          } //stages
        } //matrix
      //}  //parallel
    } //stage - parallel build
  } //stages
} //pipeline
