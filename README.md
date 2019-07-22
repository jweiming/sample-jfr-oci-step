# Using Java Flight Recorder Wercker Step

This project demonstrates how to use the Java Flight Recorder Wercker Step to collect a
recording of your application's execution and upload it to your OCI object storage bucket.


## Usage

The example `wercker.yml` below demonstrates how you can use the Java Flight Recorder Wercker Step to run the `SampleApplication` and conduct a recrording. 

After the recording is done, the report file will be uploaded to OCI object storage for analysis with JMC later. In order, the following represents the various workflows in the the Wercker pipeline.

#### Gradle build

```
build:
  steps:
    - script:
      name: Build
      code: |
        echo "Hello Java Flight Recorder!"
        
    # install curl 
    - script:
      code : |
        apk --no-cache add curl  
        
    # https://github.com/wercker/step-gradle
    - java/gradle:
      task: build
      cache_project_cache: true
```

#### JFR recording


``` 
jfr:
  steps:
    - wercker/java-flight-recorder:
      application: com.oracle.sample.SampleApplication
      classpath: /pipeline/source/build/libs/source.jar
      filename: myrecording.jfr
      experimental: true
      duration: 4m
      delay: 30s
```

#### Upload to OCI object storage

The step below uses Wercker environment parameters. You need to set them beforehand in Wercker before running the pipeline.

```
upload-to-oci:
  steps:
    - script:
      name: Set API_KEY file
      code:  |
         echo -ne "-----BEGIN RSA PRIVATE KEY-----\n" > gen-api.pem
         echo -ne $OS_APIKEY | tr " " "\n" >> gen-api.pem
         echo -ne "\n-----END RSA PRIVATE KEY-----\n" >> gen-api.pem
         
    - oci-objectstore:
      command: put
      region: us-ashburn-1
      tenancy-ocid: $TENANCY_OCID
      user-ocid: $USER_OCID
      fingerprint: $FINGERPRINT
      api-key: "$(cat gen-api.pem)"
      bucket-name: $BUCKET_NAME
      namespace: $NAMESPACE
      local-file: myrecording.jfr
      overwrite: true
```

When the build is finished, the output will be available in the file `myrecording.jfr` in the OCI object storage bucket, as requested.  You can open the recording in Java Mission Control.

## To-do

1) Investigate using ```timeout``` parameter for the jfr step as it seems to cause issues (e.g. unregonised parameter)
2) Running it with Springboot/Micronaut applications (jar file)
2) Need find a better way to inject the api key into the wercker pipeline.

## Credits

* [https://github.com/wercker/step-oci-objectstore](https://github.com/wercker/step-oci-objectstore)
* [https://github.com/markxnelson/sample-jfr-step](https://github.com/markxnelson/sample-jfr-step)
