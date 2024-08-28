#!/usr/bin/env groovy
pipeline {

    agent {
        node {
            label NODE_LABEL
        }
    }

    stages {
        stage('Create tar.gz of repo') {
            steps{
                script {
                    sh 'hostname'
                    version = readFile "VERSION_PREFIX"
                    env.CURRENT_VERSION = version
                    sh '''
                    pwd
                    ls -l
                    cd ..
                    tar -czvf apt-config-$CURRENT_VERSION.tar.gz --exclude-vcs -X apt-config_publish/exclude.txt -C apt-config_publish/ .
                    ls -l
                    tar -tvf apt-config-$CURRENT_VERSION.tar.gz
                    mv apt-config-$CURRENT_VERSION.tar.gz apt-config_publish/.
                    cd -
                    ls -l
                    '''
                }
            }


        }
        stage('Publish to nexus') {
            steps{
                script {
                    sh '''
                    echo $M2_HOME
                    export M2_HOME=/usr/share/maven
                    echo $M2_HOME

                    mvn deploy:deploy-file \
                        -DgroupId=com.ericsson.oss.ci.rtd \
                        -DartifactId=apt-config \
                        -Dversion=$CURRENT_VERSION \
                        -DgeneratePom=true \
                        -Dpackaging=tar.gz \
                        -DrepositoryId=nexus \
                        -Durl=https://arm1s11-eiffel004.eiffel.gic.ericsson.se:8443/nexus/content/repositories/releases/ \
                        -Dfile=apt-config-$CURRENT_VERSION.tar.gz
                    
                    '''
                }
            }


        }
        stage('Tag repo and Bump Version') {
            steps {
                script {
                    sh 'hostname'
                    version = readFile "VERSION_PREFIX"
                    env.CURRENT_VERSION = version
                    currentBuild.displayName = "${BUILD_NUMBER} - Version - " + version
                    
                    sh '''
                        git tag -af $CURRENT_VERSION -m "Release $CURRENT_VERSION"
                        git push origin $CURRENT_VERSION || true
                    '''

                    sh 'docker run --rm -v $PWD/VERSION_PREFIX:/app/VERSION -w /app armdocker.rnd.ericsson.se/proj-enm/bump patch'
                    newVersion = readFile "VERSION_PREFIX"
                    env.UPDATED_VERSION = newVersion
                    
                    sh '''
                        git add VERSION_PREFIX
                        git commit -m "Version $UPDATED_VERSION"
                        git push ssh://gerrit-gamma.gic.ericsson.se:29418/OSS/com.ericsson.oss.ci.rtd/apt-config HEAD:master
                    '''
                }
            }
        }
    }
}