# The differents steps for testing and fixing

This is based on this repo providing a vulnerable app by Christophe Tafani-Dereeper:
https://github.com/christophetd/log4shell-vulnerable-app.git


## Step 1: show the log4j version we are using

The `Dockerfile` can be found in `for-building-with-different-log4j-version`.

1. Add `RUN echo "The proof is in the pudding" && ./gradlew dependencyInsight --dependency log4j-core` to the first stage of the Dockerfile
1. Rebuild: `docker build --no-cache --progress=plain . -t docker.io/asksven/l4-poc:2-14-1-1`

This shows that we are using `org.apache.logging.log4j:log4j-core:2.14.1` 

## Step 2: Update to 2.16

This step is based on another repo: `https://github.com/tejas-nagchandi/CVE-2021-45105`.

1. Copy the `Dockerfile` from `for-log4j-2-16-0`
1. Rebuild: `docker build --no-cache --progress=plain . -t docker.io/asksven/l4-poc:2-16-0-2`

## Step 3: update to 2.17.0

Back to the repo: `https://github.com/christophetd/log4shell-vulnerable-app.git
`
`build.grade` can be found in `for-log4j-2-17-0`.

1. Add the below to your `build.gradle` 
1. Rebuild: `docker build --no-cache --progress=plain . -t docker.io/asksven/l4-poc:2-17-0-1`

```
configurations.all {
	resolutionStrategy.eachDependency { DependencyResolveDetails details ->
		if (details.requested.group == 'org.apache.logging.log4j') {
			details.useVersion '2.17.0'
		}
	}
}
```

The same can be tested on `https://github.com/tejas-nagchandi/CVE-2021-45105`:

1. Edit `pom.xml` and update the olog4j version by editing this line: `<log4j2.version>2.16.0</log4j2.version>`
1. Rebuild: `docker build --no-cache --progress=plain . -t docker.io/asksven/l4-poc:2-17-0-1`
 
## Step 4: Completely remove the JndiLookup class

The `Dockerfile` can be found in `for-removing-jndilookup`.

**Notes:** 

- if you build your app yourself this should be done in the first stage of the `Dockerfile`. The reason why I decided to do it int the second stage is that this how you would go about handling COTS applications
- putting the jar-file together requires the list of all directories. In our example aside from `BOOT-INF` and `META_INF` it is only `org`, but this may vary depending on your app; `jar -c0Mf /tmp/jar-fix/spring-boot-application.jar -C /tmp/jar-fix/ BOOT-INF/ -C /tmp/jar-fix/ META-INF/ -C /tmp/jar-fix/ org/`

1. Install zip: `apk add zip`
1. Uncompress the jar-file into `tmp/jar-fix`
1. Delete `org/apache/logging/log4j/core/lookup/JndiLookup.class`
1. Put the jar-file back together
1. Copy the jar-file to the `/app/`


