# Use generated application.yml to configure static-creds-broker

How to deploy the broker by using a ```application.yml``` file generated from ```APPLICATION_YML``` environment variable to configure static-creds-broker.

## Deploy
### Download the latest binary release. (Same step as using environment variables)
```
# Create a clean directory for the deployment.
$ mkdir broker_deployment
$ cd broker_deployment/
$ LATEST_RELEASE_URL=$(curl -s https://api.github.com/repos/Orange-OpenSource/static-creds-broker/releases/latest | grep browser_download_url | head -n 1 | cut -d '"' -f 4)
$ LATEST_RELEASE_VERSION=${LATEST_RELEASE_URL#*/download/v}
$ LATEST_RELEASE_VERSION=${LATEST_RELEASE_VERSION%/static-creds-broker.jar}
$ echo "Latest version: $LATEST_RELEASE_VERSION"
$ echo "Downloading $LATEST_RELEASE_URL"
$ curl -O -L $LATEST_RELEASE_URL
```

### Create the configuration file application.yml.
A sample application configuration file (application.tmpl.yml) is provided at https://github.com/orange-cloudfoundry/static-creds-broker/blob/master/application.tmpl.yml.
```
# application.yml sample can be found at https://github.com/orange-cloudfoundry/static-creds-broker/blob/master/application.tmpl.yml
$ curl -O -L https://github.com/orange-cloudfoundry/static-creds-broker/blob/master/application.tmpl.yml
$ cp ../application.tmpl.yml application.yml
# edit the configuration file application.yml to your proper services properties.
$ vi application.yml
services:  
    TRIPADVISOR:
        NAME: MyService
        DESCRIPTION: My existing service 
        METADATA:
            LONG_DESCRIPTION: A long description for my service
        CREDENTIALS:
            URI: mysql://USERNAME:PASSWORD@HOSTNAME:PORT/NAME
            ACCESS_KEY: AZERT23456664DFDSFSDFDSF
```
### Configure the manifest file used by CF CLI and deploy the broker.
A sample manifest file (manifest.tmpl.yaml-config.yml) is provided, create a manifest.yml file by adapting it to your environment (in particular set the domain)
```
$ cp manifest.tmpl.yaml-config.yml manifest.yml
$ vi manifest.yml
---
applications:
- name: my-broker
  memory: 256M
  instances: 1
  host: mybroker
  domain: my-admin-domain.cf.io
  path: static-creds-broker-xx.jar
  env:
   SECURITY_USER_PASSWORD: secret

# deploy the broker without starting it
$ cf push --no-start
# set APPLICATION_YML environment variable
$ cf set-env my-broker APPLICATION_YML "$(cat application.yml)"
#start the broker
$ cf start my-broker
# you should see APPLICATION_YML environment variable detection in logs
$ cf logs --recent my-broker | grep APPLICATION_YML
[...]
APPLICATION_YML environment variable is set. Will use it to create /home/vcap/app/application.yml.
[...]
```

Note: broker security properties could be configured either in application.yml or in environment variables. If they are configured in both, the value configured in environment variables will be taken, according to the precedence of Spring property source.

### Alternative : Configure the manifest file with inlined application.yml and deploy the broker.
A sample manifest file (manifest.tmpl.yaml-config.yml) is provided, create a manifest.yml file by adapting it to your environment (in particular set the domain)
```
$ cp manifest.tmpl.yaml-config.yml manifest.yml
$ vi manifest.yml # you can then inline application.yml content into APPLICATION_YML environment variable
---
applications:
- name: my-broker
  memory: 256M
  instances: 1
  host: mybroker
  domain: my-admin-domain.cf.io
  path: static-creds-broker-xx.jar 
  env:
   SECURITY_USER_PASSWORD: secret
   APPLICATION_YML: |
    services:  
        TRIPADVISOR:
            NAME: MyService
            DESCRIPTION: My existing service 
            METADATA:
                LONG_DESCRIPTION: A long description for my service
            CREDENTIALS:
                URI: mysql://USERNAME:PASSWORD@HOSTNAME:PORT/NAME
                ACCESS_KEY: AZERT23456664DFDSFSDFDSF

# deploy the broker
$ cf push
# you should see APPLICATION_YML environment variable detection in logs
$ cf logs --recent my-broker | grep APPLICATION_YML
[...]
APPLICATION_YML environment variable is set. Will use it to create /home/vcap/app/application.yml.
[...]
```

Note: broker security properties could be configured either in application.yml or in environment variables. If they are configured in both, the value configured in environment variables will be taken, according to the precedence of Spring property source.

## Config syntax
Note: <service_id> and <plan_id> should not contain "."
The properties meaning and default value could be consulted in [README](https://github.com/Orange-OpenSource/static-creds-broker#config-reference)
```
security:
    user:
        name: <broker_username>
        password: <broker_password>
services:  
    <service_id>:
        NAME: <service_name>
        DESCRIPTION: <service_description> 
        TAGS: <service_tags_separated_by_comma>
        METADATA:
            DISPLAY_NAME: <service_displayname>
            IMAGE_URL: <service_imageurl>
            SUPPORT_URL: <service_supporturl>
            DOCUMENTATION_URL: <service_documentationurl>
            PROVIDER_DISPLAY_NAME: <service_provider_displayname>
            LONG_DESCRIPTION: <service_long_description>
        CREDENTIALS:
            <service_credential_key>: <service_credential_value>
        PLAN:
            <plan_id>:
                NAME: <plan_name>
                DESCRIPTION: <plan_description>
                METADATA: <plan_metadata_string_holding_json_object>
                FREE: <whether_plan_is_free>
                CREDENTIALS:
                    <plan_credential_key>: <plan_credential_value>
    <another_service_id>:
        NAME: <another_service_name>
        DESCRIPTION: <another_service_description>
        CREDENTIALS: <credentials_in_format_string_holding_json_hash>
```