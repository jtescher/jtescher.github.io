---
layout: post
title:  "Like Heroku But Open Source, Getting Started With OpenShift"
date:   1970-01-01 00:00:00
categories:
---


```shell
$ boot2docker ssh "echo $'EXTRA_ARGS=\"--insecure-registry 172.30.0.0/16\"' | \
    sudo tee -a /var/lib/boot2docker/profile && sudo /etc/init.d/docker restart"

  EXTRA_ARGS="--insecure-registry 172.30.0.0/16"
  Need TLS certs for boot2docker,127.0.0.1,10.0.2.15,192.168.59.103
  -------------------
```

```shell
$ docker run -d --name "openshift-origin" --net=host --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock \
    openshift/origin start

  Unable to find image 'openshift/origin:latest' locally
  latest: Pulling from openshift/origin
  6941bfcbbfca: Pull complete 
  41459f052977: Pull complete 
  fd44297e2ddb: Pull complete 
  35cada882f29: Pull complete 
  1351fd0b9cc4: Pull complete 
  605a97a7945f: Pull complete 
  d18b3556d7c6: Pull complete 
  7ae18d9bf395: Pull complete 
  1d5e3f0fd454: Pull complete 
  d03e1d4fbad2: Pull complete 
  2efc094a60bf: Already exists 
  Digest: sha256:3de2d9b8f4dc3ec330e970eb1a422d45a10924512354e2c1b35b576b4888b5af
  Status: Downloaded newer image for openshift/origin:latest
  697dc6f993926ba76fa606b603395d502ac8f78f0d38eb17acdefc516aea437d
```

```shell
$ docker exec -it openshift-origin bash
```

```shell
[root@boot2docker openshift]# openshift admin policy add-role-to-group cluster-admin system:authenticated config=/var/lib/openshift/openshift.local.config/master/admin.kubeconfig
```

```shell
[root@boot2docker openshift]# osadm registry --create --credentials="${OPENSHIFTCONFIG}"

  deploymentConfigs/docker-registry
  services/docker-registry
```

```shell
[root@boot2docker openshift]# osadm router --create --credentials="${OPENSHIFTCONFIG}"

  deploymentConfigs/router
  services/router
```

```shell
[root@boot2docker openshift]# osc new-project ember-test --display-name="Ember Test Project" --description="This is an example of an ember app deployment."
```

```shell
[root@boot2docker openshift]# echo '---
  kind: "List"
  apiVersion: "v1beta3"
  items:
    - kind: "ImageStream"
      apiVersion: "v1beta3"
      metadata:
        name: "nodejs-010-centos7"
    - kind: "BuildConfig"
      apiVersion: "v1beta3"
      metadata:
        name: "nodejs-010-centos7-build"
      spec:
        source:
          type: "Git"
          git:
            uri: "https://github.com/pat2man/sti-nodejs.git"
            ref: "master"
          contextDir: "0.10"
        strategy:
          type: "Docker"
          dockerStrategy: {}
        output:
          to:
            kind: "ImageStream"
            name: "nodejs-010-centos7"' | osc create -f - 

