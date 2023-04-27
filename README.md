##### BuildConfig with dockerfile content within (& **no** `build.spec.source.git` section):
- bc-basic.yaml
```
spec:
  triggers:
    - type: ConfigChange
  source:
    type: Dockerfile
    dockerfile: |
      FROM registry.access.redhat.com/ubi8/ubi

      RUN yum install -y nginx && \
          sed -i 's/80 default_server/8080 default_server/' /etc/nginx/nginx.conf && \
          echo 'daemon off;' >> /etc/nginx/nginx.conf && \
          sed -i 's!error_log.*!error_log /dev/stderr;!' /etc/nginx/nginx.conf && \
          sed -i 's!access_log.*!access_log /dev/stdout main;!' /etc/nginx/nginx.conf && \
          sed -i 's/user nginx;//' /etc/nginx/nginx.conf  && \
          sed -i 's!pid /run/nginx.pid;!pid /tmp/nginx.pid;!' /etc/nginx/nginx.conf  && \
          yum -y clean all --enablerepo='*'

      ENTRYPOINT ["nginx"]
  strategy:
    type: Docker
  output:
    to:
      kind: ImageStreamTag
      name: 'simple-docker-build:basic'
```
- istag basic:
```
> $ oc get istag simple-docker-build:basic -oyaml | grep -i commit  
> <no output>
```

##### BuildConfig with `build.spec.source.git` section (as an add-on):
- bc-advanced.yaml
```
spec:
  triggers:
    - type: ConfigChange
  source:
    type: Dockerfile
    dockerfile: |
      FROM registry.access.redhat.com/ubi8/ubi

      RUN yum install -y nginx && \
          sed -i 's/80 default_server/8080 default_server/' /etc/nginx/nginx.conf && \
          echo 'daemon off;' >> /etc/nginx/nginx.conf && \
          sed -i 's!error_log.*!error_log /dev/stderr;!' /etc/nginx/nginx.conf && \
          sed -i 's!access_log.*!access_log /dev/stdout main;!' /etc/nginx/nginx.conf && \
          sed -i 's/user nginx;//' /etc/nginx/nginx.conf  && \
          sed -i 's!pid /run/nginx.pid;!pid /tmp/nginx.pid;!' /etc/nginx/nginx.conf  && \
          yum -y clean all --enablerepo='*'

      ENTRYPOINT ["nginx"]
    git:
      uri: 'https://github.com/openshift-examples/container-build.git'
  strategy:
    type: Docker
  output:
    to:
      kind: ImageStreamTag
      name: 'simple-docker-build:advanced'
```
- istag advanced:
```
$ oc get istag simple-docker-build:advanced -oyaml | grep commit   
        io.openshift.build.commit.author: Robert Bohne <robert.bohne@redhat.com>
        io.openshift.build.commit.date: Thu May 27 16:42:02 2021 +0200
        io.openshift.build.commit.id: bbf247b286281c4dc1d132907e288db59a55d666
        io.openshift.build.commit.message: Add entitled-build/Containerfile
        io.openshift.build.commit.ref: master
        io.openshift.build.commit.author: Robert Bohne <robert.bohne@redhat.com>
        io.openshift.build.commit.date: Thu May 27 16:42:02 2021 +0200
        io.openshift.build.commit.id: bbf247b286281c4dc1d132907e288db59a55d666
        io.openshift.build.commit.message: Add entitled-build/Containerfile
        io.openshift.build.commit.ref: master
```

To test by yourself (on an OpenShift cluster):
$ oc create -f extras/is.yaml

$ oc create -f buildconfig_yamls/bc-basic.yaml
Check the output & similar could be repeated for bc-advanced.yaml.