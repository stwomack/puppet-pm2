puppet-pm2
==========

Puppet Module to deploy a nodejs application using PM2 

This is a pretty hacky first version. 
It relies on puppet module willdurand/nodejs to install nodejs and npm.
It would be good to make the create_app a provider called say pm2_app.
and use the nodejs::npm provider instead of executing npm install etc.  



Minimal Usage: 
=============

     class { 'pm2': }

     class { 'pm2::create_app':
       name    => 'my-nodejs-app',
       require => Class['pm2']
     } 
 

Detailed Usage:
===============

     class { 'pm2':
       npm_repository    => "https://registry.npmjs.org",
       npm_auth          => 'Ashtyhy=+as',
       npm_always_auth   => true,
       pm2_version       => "latest",
       install_root      => '/opt',
       install_dir       => 'nodejs',
       deamon_user       => 'nodejs',  
     }

     class { 'pm2::create_app':
       name            => 'my-nodejs-app',
       app             => 'myapp',
       appversion      => 'latest',
       path            => "/opt/nodejs/myapp",
       script          => "lib/app.js",              
       args            => ["arg1","arg2"],
       env             => '{ "env.NODE_ENV" : "test" }',
       install_root    => '/opt',
       install_dir     => 'nodejs',
       deamon_user     => 'nodejs',     
       require => Class['pm2']
     } 
     
App Deployment:
===============

     Instead of deploying the nodejs app at puppet configuration time a bash script is generated by puppet 
     so deployment can be done later via ssh: 
     To do deployment this way run: 
        sudo su - nodejs
       /opt/nodejs/deploy_app_script.sh myapp 0.0.1-110  serve_app.js '{ "env.NODE_ENV" : "test" }'  "[arg='value']"
     This way a generic nodejs/pm2 server can be build and ssh used to deploy a given app as part of a continuous integration 
     workflow using tools like jenkins or teamcity.

     It is also possible to test that a deployment was successful by running via ssh: 
        sudo -u nodejs /opt/nodejs/deploy_test.sh 

     NOTE: Both the puppet and bash script deployment is done via npm so the application is assumed to be in a public or private 
     repository like sinopia. The other alternative is deploy from a source repository like git. PM2-Deploy supports this form of 
     deployment but currently there is no puppet or bash script to support this but it can be easily done.    
 
Nodejs Configuration: 
====================

 Module "willdurand/nodejs" is called to do the nodejs install. 
 Assuming using puppet 3.x with hiera then nodejs can be configured by setting the parameters in hiera:

     nodejs::version:          'stable'   ('vX.Y.Z', 'latest' or 'stable')
     nodejs::target_dir:       '/usr/local/bin'
     nodejs::with_npm:         true
     nodejs::make_install:     true
     nodejs::create_symlinks:  false
     
 Also you need to set the path in Factor before running this as willdurand/nodejs relies on the $::path variable
  
     set path to '/usr/local/node/node-default/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin'

Working with PM2 - Commands, Debugging etc. 
===========================================
 logon to to the daemon user (by default nodejs) and you will be in home directory (/opt/nodejs by default) 

 To show running apps:

     pm2 list

 To restart my-app

    In the home directory (/opt/nodejs by default)

    pm2 kill
    pm2 start "my-app/pm2.json" --name "my-app"

 To see the logs 

    pm2 logs 

 Deploying my-app

    sudo -u nodejs /opt/nodejs/deploy_app.sh my-app 0.1.182 lib/app.js '{ "env.NODE_ENV" : "dev"}' 
