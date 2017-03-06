#!groovy

/**
 * Jenkinsfile for OSSEC Wazuh
 * Copyright (C) 2016 Wazuh Inc.
 * June 22, 2016.
 *
 * This program is a free software; you can redistribute it
 * and/or modify it under the terms of the GNU General Public
 * License (version 2) as published by the FSF - Free Software
 * Foundation.
 */

def labels = ['debian-jessie-slave']

//Stage checkout source
def check_source(label){
    sh 'sudo apt-get update'
    sh 'sudo apt-get install -y curl apt-transport-https'
    sh 'curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -'
    sh 'echo "deb https://packages.wazuh.com/apt xenial main" | tee /etc/apt/sources.list.d/wazuh.list'
    sh 'sudo apt-get update'
    sh 'sudo apt-get install -y wazuh-manager'
    sh 'service wazuh-api status'
}


//Stage unit tests
def unit_tests(label){
    dir('src'){
        sh 'sudo make --warn-undefined-variables test_valgrind'
        sh 'sudo make clean'
    }
}


//Stage standard ossec compilations
def  standard_compilations(label){  
    dir ('src') {
        sh 'sudo make --warn-undefined-variables V=1 TARGET=agent install'
        sh 'sudo make clean && sudo rm -rf /var/ossec/'
        sh 'sudo make --warn-undefined-variables V=1 TARGET=server install'
        sh 'sudo make clean'
    } 
} 


//Stage rule tests
def rule_tests(label){ 
    dir ('src') {
        sh 'sudo make V=1 TARGET=server test-rules'
        sh 'sudo make clean && sudo rm -rf /var/ossec/'
    }
}


//Stage advanced ossec compilation
def advanced_compilations(label){
    
    dir ('src') {
        sh 'sudo make --warn-undefined-variables V=1 TARGET=local install'
        sh 'sudo make clean && sudo rm -rf /var/ossec/'
        sh 'sudo make --warn-undefined-variables V=1 TARGET=hybrid install'
        sh 'sudo make clean && sudo rm -rf /var/ossec/'
    
        def matrixOptionsX = ['DATABASE=none', 'DATABASE=pgsql', 'DATABASE=mysql']
        def matrixOptionsY = ['USE_GEOIP=1', '']    
        
        
        for (optionX in matrixOptionsX){
            for (optionY in matrixOptionsY) {
                sh 'sudo make --warn-undefined-variables V=1 TARGET=server '+optionX+' '+optionY+' install'
                sh 'sudo make clean && sudo rm -rf /var/ossec/'
            }
        }
    }
}


//Stage windows compilation
def windows_compilation(label){ 
 
    dir ('src') {
        sh 'sudo make --warn-undefined-variables TARGET=winagent'
        sh 'sudo make clean && sudo rm -rf /var/ossec/'
    }
}



for (label in labels) {
    
    stage (label){
        node(label) {
   
            
            stage ('Add repositories'){
                check_source(label)
            }
            
            stage ('Unit Tests'){
               // unit_tests(label)
            }
            
            stage ('Standard Compilations'){
                //standard_compilations(label)
            }
            
            stage ('Rule Tests'){
               // rule_tests(label)
            }
            
            stage ('Advanced Compilations'){
              //  advanced_compilations(label)
            }
            
            stage ('Windows Compilation'){
             //   windows_compilation(label)
            }
        }
    }
}
