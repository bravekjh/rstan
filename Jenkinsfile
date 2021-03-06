pipeline {
    agent any
    environment {
        MAKEFLAGS="-j${env.PARALLEL}"
        R_LIBS="~/.RLibs"
        export USE_CXX14=1
    }
    stages {
        stage('Install dependencies') {
            steps {
                sh """
                    mkdir -p ~/.RLibs
                    R -e 'install.packages("devtools", repos="http://cran.us.r-project.org")'
                    R -e 'update(devtools::package_deps("rstan"))'
                    R -e 'install.packages("RInside", repos="http://cran.us.r-project.org")'
                """
            }
        }
        stage('Build') {
            steps {
                sh """
                    cd StanHeaders
                    git submodule update --init --recursive --remote
                    cd ..
                    R CMD build StanHeaders
                    R CMD build --no-build-vignettes rstan/rstan
                """
            }
        }
        stage("Check timings and output") {
            steps {
                sh """
                    R CMD check --as-cran --timings StanHeaders_*.tar.gz || \
                      cat StanHeaders.Rcheck/00check.log
                    R CMD INSTALL --library=~/.RLibs StanHeaders_*.tar.gz
                    R CMD check --as-cran --timings --run-donttest --run-dontrun rstan_*.tar.gz || \
                      cat rstan.Rcheck/00check.log
                """
            }
        }
        stage("Check additional unit tests") {
            steps {
                sh """
                    R CMD INSTALL rstan_*.tar.gz
                    cd rstan
                    make test-R || echo "extra unit tests failed"
                    cd ..
                """
            }
        }
        stage("Check rstanarm") {
            steps {
                sh """
                    R -e 'update(devtools::package_deps("rstanarm"))'
                   curl -O https://cran.r-project.org/src/contrib/rstanarm_2.15.3.tar.gz
#                    R CMD check --as-cran --timings --run-donttest --run-dontrun rstanarm_*.tar.gz || \
#                      cat rstanarm.Rcheck/00check.log
                """
            }
        }
    }
    post {
        always {
            warnings consoleParsers: [[parserName: 'GNU C Compiler (gcc)']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
            warnings consoleParsers: [[parserName: 'Clang (LLVM based)']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
            deleteDir()
        }
    }
}
