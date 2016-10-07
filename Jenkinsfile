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

node {
    
    //Stage checkout source
    stage name: 'Checkout source', concurrency: 1
    
    checkout scm
    sh 'zypper update'
    
    //Stage unit tests
    stage name: 'Unit tests', concurrency: 1
    
    dir ('src') { 
        sh 'make --warn-undefined-variables test_valgrind'
        sh 'make clean'
    }
    
    //Stage code scanners
    stage name: 'Code scanners', concurrency: 1
    
    dir ('src') {
        
    }
    
    //Stage standard ossec compilations
    stage name: 'Standard OSSEC compilations', concurrency: 1 
    
    dir ('src') {
        sh 'make --warn-undefined-variables V=1 TARGET=agent install'
        sh 'make clean && sudo rm -rf /var/ossec/'
        sh 'make --warn-undefined-variables V=1 TARGET=server install'
        sh 'make clean'
    }  
    
    //Stage rule tests
    stage name: 'Rule tests', concurrency: 1 
    
    dir ('src') {
        sh 'cat /var/ossec/etc/ossec.conf'
        sh 'make V=1 TARGET=server test-rules'
    }
    
    //Stage advanced ossec compilation
    stage name: 'Advanced OSSEC compilations', concurrency: 1 
    
    dir ('src') {
        sh 'make --warn-undefined-variables V=1 TARGET=local install'
        sh 'make clean && sudo rm -rf /var/ossec/'
        sh 'make --warn-undefined-variables V=1 TARGET=hybrid install'
        sh 'make clean && sudo rm -rf /var/ossec/'
    
        def matrixOptionsX = ['DATABASE=none', 'DATABASE=pgsql', 'DATABASE=mysql']
        def matrixOptionsY = ['USE_GEOIP=1', '']    
        
        //sh 'sudo zypper -n install geoip-bin geoip-database libgeoip-dev libgeoip1 libpq-dev libpq5 libmysqlclient-dev'        
        
        for (optionX in matrixOptionsX){
            for (optionY in matrixOptionsY) {
                sh 'make --warn-undefined-variables V=1 TARGET=server '+optionX+' '+optionY+' install'
                sh 'make clean && sudo rm -rf /var/ossec/'
            }
        }
    }
    
    //Stage windows compilation
    stage name: 'Windows compilation', concurrency: 1
    
    dir ('src') {
        //sh 'sudo zypper -n install aptitude && sudo aptitude -y install mingw-w64 nsis'
        sh 'make --warn-undefined-variables TARGET=winagent'
        sh 'make clean && sudo rm -rf /var/ossec/'
    }
    
}
