---
title: "Spring Boot: Security Hardening"
date: "2021-12-28T01:00:00"
disableShare: true
hideFooter: true
tags: ['java', 'spring', 'spring-boot', 'security', 'spring-security']
categories: ['programming']
---

When working on the [Squaddy App](https://squaddy.app) I've seen a LOT of [internet background noise](https://en.wikipedia.org/wiki/Internet_background_noise) in our logs. It's especially timely given the recent [Log4j2 Remote Code Execution](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-45105) exploit.

We've done a lot of work to ensure we stay as safe as possible and I wanted to go through some recommendations.

## What we mean by 'Safe'?

Whenever talking about security we're really asking *"How confident am I this will work as I expect?"*. No system is entirely secure and all security is ultimately based upon trust. Even in a [zero trust security model](https://en.wikipedia.org/wiki/Zero_trust_security_model), someone, somewhere, has to be trusted.

Ultimately it's a business decision where and who is burdened with that trust. 

To give a concrete example, we assume that all [JSON Web Tokens](https://en.wikipedia.org/wiki/JSON_Web_Token) are valid using [Firebase Authentication SDK](https://firebase.google.com/docs/auth). For us, Firebase is our golden key and we believe that to be a small (and acceptable) amount of risk.

> Its important before doing *any* security hardening to know what needs to be secured and what are deemed as acceptable risks.

Our aims should be:

- [Minimize the attack surface.](#minimize-the-attack-surface)
- [Secure all secrets in a vault.](#secure-all-secrets-in-a-vault) *No one should know production passwords.*
- [Sanitize all user inputs.](#sanitize-all-user-inputs)
- [Redact sensitive information from logging.](#redact-sensitive-information-from-logging)
- [Add automated tooling to prevent security flaws before they are merged.](#add-automated-tooling-to-prevent-security-flaws-before-they-are-merged)

## Minimize the attack surface

Rather than providing lots of information thats already out there, I recommend:

- [Enable Tomcat HTTPS](https://www.mulesoft.com/tcat/tomcat-ssl) or at least, have HTTPS for all services hitting your ingress.
- [Use a Content Security Policy (CSP)](https://docs.spring.io/spring-security/site/docs/5.2.0.RELEASE/reference/html/default-security-headers-2.html#webflux-headers-csp) for Cross Site Scripting (XSS) protection.
- [Configure a Cross Site Request Forgery (CSRF) policy](https://docs.spring.io/spring-security/site/docs/5.0.x/reference/html/csrf.html). This *may* include disabling it if using only REST.
- [Test your response headers](https://gf.dev/http-headers-test) and remove any that give away more than they should (server type, allowed methods, etc).
- [Use a Web Application Firewall (WAF)](https://aws.amazon.com/waf/) to block traffic your services should not serve, e.g. countries you don't operate in or requests missing a JWT `Authorization` header.


Also, if your application is only available via REST, I recommend adding a default `IndexController` to serve browser traffic which is typically automated crawler/bots.

```java
/**
 * This controller is used to handle the case where the application is accessed via a browser.
 * </p>
 * We want to return unauthorized to any request to prevent bots and crawlers from accessing the API.
 */
@Hidden //Hide from Swagger
@RestController
public class IndexController implements ErrorController {
    private static final Logger LOGGER = LoggerFactory.getLogger(IndexController.class);

    @RequestMapping(value = "/")
    public ResponseEntity<Void> index() {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }

    @RequestMapping(value = "/error")
    public ResponseEntity<Void> error() {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }

    @RequestMapping(value = "/robots.txt")
    public void robots(final HttpServletRequest request, final HttpServletResponse response) {
        try {
            response.getWriter().write("User-agent: *\nDisallow: /\n");
        } catch (final IOException e) {
            LOGGER.error(SecurityMarkers.EVENT_FAILURE, "Failed to write robots.txt", e);
        }
    }
}
```

The above will return `401` for any default or error pages and also return a `disallow *` for any `robots.txt`. This should help reduce the amount of crawler traffic.

### Choose a minimal runtime

A primary concern is trusting that the runtime on our laptops is identical to production. Ensuring deployed dependencies - and therefore attack surface - is as small as possible.

A key mitigation strategy is to create a [Twelve-Factor App](https://12factor.net/) that can be securely tested and deployed by taking the same artifact through each environment. By only changing a few settings, it limits the amount of change and prevents unforeseen drift which leads to vulnerabilities.

[Docker](https://www.docker.com/) has proven to be a good way to bundle an application and using [Spring Profiles](https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/boot-features-profiles.html) you can ensure the application can be configured in predictable ways.

> In a future post we'll deep dive how to configure automated docker deployments to AWS Fargate using GitHub Actions.

### Configuring Maven Docker

While it is possible to manually create a Dockerfile, I've found this produces surprisingly large images. It also creates *another* technology to learn and maintain security updates for.

We want a technology that always keeps up to date on every build and can produce a jar with only the required libraries. Luckily, [Buildpack](https://buildpacks.io/) can do this for us using a [Maven Plugin](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1).

Adding the following to your `pom.xml` and running `mvn package` will produce a docker image.

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <image>
            <name>${project.version}</name>
            <env>
                <JAVA_TOOL_OPTIONS>-Xms2048m -Xmx2048m</JAVA_TOOL_OPTIONS>
            </env>
        </image>
    </configuration>
</plugin>
```

Compared to a manual Dockerfile I've found this image to be much smaller and preconfigured with the correct Java flags.

> I've set the maximum allocation pool and initial memory size flags using `JAVA_TOOL_OPTIONS`. This is important as we want the to automatically set the correct heap size based on a proper memory size. I've found that the container does not quite understand its size when this is not set. YMMV.

The buildpack used is the [Bellsoft Liberica](https://github.com/paketo-buildpacks/bellsoft-liberica) JDK. I would recommend also using the same JDK when developing locally to ensure consistency.

### Injecting environment variables

You may be wondering, *"How do we set things like database connections? They can't be baked into the image right?"*. 

Fear not, when we deploy this image we can override any Spring environment variable we want for example, setting [@Profile](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Profile.html).

```xml
<env name="AWS_REGION" value="eu-west-2" />
<env name="spring.profiles.active" value="prod" />
<env name="app.squaddy.api.devMockUser.enabled" value="false" />
<env name="spring.datasource.username" value="AUserName" />
<env name="spring.datasource.password" value="************" />
<env name="spring.datasource.url" value="jdbc:mysql://squaddy.******.eu-west-2.rds.amazonaws.com:3306/db" />
```

While our API has around fifty environment settings, the difference between `local`, `integration` and `production` environments are only the most essential ones - database connection, AWS account information, etc. This gives us good confidence that the code we tested and the code being deployed is the same.

### Running production, locally

Now we're confident that `integration` and `production` are similar, we should be able to make the reverse true - our `local` like `integration` or `production`.

I recommend setting up a configuration in [IntelliJ](https://www.jetbrains.com/help/idea/run-debug-configuration.html) that is identical to `integration` or `production`. That way, you can switch over quickly to connect to the real services to aid debugging. 

> If you are developing locally, having mock services is fine, but always have a ripcord to switch back to real ones.

## Secure all secrets in a vault

Having a web certificate auto renew or a password auto-rotate is truly one of the best things about [cloud services](https://en.wikipedia.org/wiki/Cloud_computing). It's absolutely something you should be using. However, one of the challenges of getting those secrets into the runtime in a performant way.

Currently there seems to be two ways of achieving this (on AWS):

- [Having the secret pull during runtime.](#pull-during-runtime)
- [Injecting the secret into build or deployment.](#inject-into-build-or-deployment)
- [Injecting the secret as an environment variable during startup.](#inject-as-environment-variable-during-startup)

They both have pro's and con's and go back to risk appetite in the business. 

### Pull during runtime

{{< detail-tag "Expand for Java example" >}}
```java
// Use this code snippet in your app.
// If you need more information about configurations or implementing the sample code, visit the AWS docs:
// https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-samples.html#prerequisites

public static void getSecret() {

    String secretName = "arn:aws:secretsmanager:eu-west-2:260023781495:secret:/secret/squaddy_dev/mysql-TFDMIj";
    Region region = Region.of("eu-west-2");

    // Create a Secrets Manager client
    SecretsManagerClient client = SecretsManagerClient.builder()
            .region(region)
            .build();

    // In this sample we only handle the specific exceptions for the 'GetSecretValue' API.
    // See https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html
    // We rethrow the exception by default.

    String secret, decodedBinarySecret;
    GetSecretValueRequest getSecretValueRequest = GetSecretValueRequest.builder()
            .secretId(secretName)
            .build();
    GetSecretValueResponse getSecretValueResponse = null;

    try {
        getSecretValueResponse = client.getSecretValue(getSecretValueRequest);
    } catch (DecryptionFailureException e) {
        // Secrets Manager can't decrypt the protected secret text using the provided KMS key.
        // Deal with the exception here, and/or rethrow at your discretion.
        throw e;
    } catch (InternalServiceErrorException e) {
        // An error occurred on the server side.
        // Deal with the exception here, and/or rethrow at your discretion.
        throw e;
    } catch (InvalidParameterException e) {
        // You provided an invalid value for a parameter.
        // Deal with the exception here, and/or rethrow at your discretion.
        throw e;
    } catch (InvalidRequestException e) {
        // You provided a parameter value that is not valid for the current state of the resource.
        // Deal with the exception here, and/or rethrow at your discretion.
        throw e;
    } catch (ResourceNotFoundException e) {
        // We can't find the resource that you asked for.
        // Deal with the exception here, and/or rethrow at your discretion.
        throw e;
    }

    // Decrypts secret using the associated KMS key.
    // Depending on whether the secret is a string or binary, one of these fields will be populated.
    if (getSecretValueResponse.secretString() != null) {
        secret = getSecretValueResponse.secretString();
    }
    else {
        decodedBinarySecret = new String(Base64.getDecoder().decode(getSecretValueResponse.secretBinary().asByteBuffer()).array());
    }

    // Your code goes here.
}
```
{{< /detail-tag >}}

Personally I am not a fan of the first one (which AWS gives the example for). I don't like having a surprise when the secret doesn't get pulled in runtime. This could happen for a variety of reasons but configuration/permissions change or network issue both seem like reasonable risks. I like things to fail early, ideally in the build or deployment process. 

### Inject into build or deployment

So why not always use injection? Well theres issues there too. Injecting a secret into the build is a *no go* as it breaks the [Twelve-Factor App](https://12factor.net/) principle.

### Inject as environment variable during startup

Finally, we have injecting the secret as an environment variable. This is commonly seen in [AWS provided examples using Lambda](https://aws.amazon.com/blogs/security/how-to-securely-provide-database-credentials-to-lambda-functions-by-using-aws-secrets-manager/). 

It's possible (although not well documented) that you can use a [ECS Task Definition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-taskdefinition.html) to get secrets during startup:

```json
"secrets": [
    {
        "name": "spring.datasource.password",
        "valueFrom": "arn:aws:secretsmanager:eu-west-2:12345:secret:/secret/squaddy/mysql-:12345::password::"
    },
]
```

This one I like the most as it keeps secrets separate and not 'baked in'. The negatives are:

- If secrets rotation will cause errors if they rotate during the runtime. 
- Other issues may also arise if the secret needs to change.
- The secrets are in plain text in the AWS Console.

## Sanitize all user inputs

A key strategy in [defensive programming](https://en.wikipedia.org/wiki/Defensive_programming) is not to trust anything another client or user gives you. This doesn't just mean security, it also means 'bad data'.

### Ensure you understand who your client is
Your tolerance for what a *'client'* means can vary between teams and businesses. For example, in a microservice architecture, do you trust other services? What about third party webhooks? Does having a shared secret or SSL help?

### Common sanitization techniques

Here's an example from the Squaddy API: 

```java
@PostMapping(value = "/group/chat")
public ResponseEntity<UploadRequestResponseDTO> generateChatFileSignedUrl(@RequestParam("mimeType")
                                                                          @NotBlank(message = "MimeType must not be blank.") final String mimeType,
                                                                          @RequestParam("extension")
                                                                          @NotBlank(message = "Extension must not be blank.") final String extension,
                                                                          @RequestParam("groupId")
                                                                          @Min(value = 1L, message = "The value must be positive.") final Long groupId) {
    final var safeMimeType = Encode.forJava(mimeType);
    final var safeExtension = Encode.forJava(extension);
    return new ResponseEntity<>(
            storageService.generateSignedUrl(safeMimeType.toLowerCase(Locale.ROOT), safeExtension.toUpperCase(Locale.ROOT), groupId.toString()), HttpStatus.CREATED
    );
}
```

Firstly, I highly recommend using [Spring Validation](https://www.baeldung.com/javax-validation) to check the inputs. For example `@Min` will reject any `groupId` less than 1.

Secondly, know `mimeType` and `extension` will be present (as it must pass `@NotBlank`), we can't be sure they are safe as they are strings. Common techniques to resolve this would be to trim whitespace or emoji's from strings by running through regex to only allow numbers or alphanumeric. I am not a fan of this approach. [Regex is famously complicated](https://xkcd.com/208/) and I would recommend instead using the [OWASP Java Encoder](https://owasp.org/www-project-java-encoder/).

Finally, you'll note that we don't do any sanitization on `groupId`. We know this to be a integer and (hopefully) fairly safe.

The key takeaways here are:

- A mix of validation and sanitization is required. No one approach is enough.
- Use a proper library over rolling your own regex or trimming. It's cleaner and less prone to errors.

## Redact sensitive information from logging

One overlooked aspect to security is logging. When developing locally logging feels cheap but once deployed to production is hard to see the wood for the trees.

There are a few things to remember when it comes to security and logging:

### Mask your logs

Here is a quick introduction to adding [owasp-security-logging](https://github.com/javabeanz/owasp-security-logging) and masking.

Add the following to your `pom.xml`:

```xml
<dependency>
  <groupId>org.owasp</groupId>
  <artifactId>security-logging-logback</artifactId>
  <version>LATEST</version>
</dependency>
```

Now make sure to update any `LOGGER` calls with a new `SecurityMarker`.

```java
LOGGER.info("userid={}", userid);  
LOGGER.info(SecurityMarkers.CONFIDENTIAL, "password={}", password);
```

This will later produce something similar to:

```bash
2014-12-16 13:54:48,860 [main] INFO - userid=joebob
2014-12-16 13:54:48,860 [main] [CONFIDENTIAL] INFO - password=***********
```

As you can see, the second line as been redacted due to being `CONFIDENTIAL`.

### Don't over log. 

Using [Log Levels](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-logging.html), [owasp-security-logging](https://github.com/javabeanz/owasp-security-logging) and [logstash-logback-encoder
](https://github.com/logfellow/logstash-logback-encoder) a useful (but redacted) log can be produced.

```java
LOGGER.debug(SecurityMarkers.SECURITY_AUDIT, CHECKING_USER_MESSAGE, firebaseId);
```

Will produce:

```json
{
    "@timestamp": "2021-12-28T14:42:54.118Z",
    "@version": "1",
    "message": "Checking user exists: **********************", // Redacted
    "logger_name": "app.squaddy.api.persistence.UserPersistenceService",
    "thread_name": "http-nio-8080-exec-9",
    "level": "DEBUG",
    "level_value": 10000,
    "transaction.id": "8b6f99ac-272e-4476-b29c-e0f75cc87bd0", // A unique id for this request
    "transaction.requestMethod": "GET",
    "transaction.requestUrl": "/v2/user/groups/0",
    "transaction.proxyRemoteIp": "*****", // Redacted
    "transaction.firebaseId": "**********************", // Redacted
    "transaction.host": "api.squaddy.app",
    "transaction.remoteIp": "10.0.0.159",
    "transaction.isPublicApi": "false",
    "transaction.userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36",
    "tags": [
        "SECURITY AUDIT" // A security marker added
    ]
}
```

> By using a security markers and using levels we can restrict the overhead of logging for each environment without altering code.

## Encode your logs

[Log Forging or Log Injection](https://owasp.org/www-community/attacks/Log_Injection) is one of the top OWASP vulnerabilities and not often talked about. The same [owasp-security-logging](https://github.com/javabeanz/owasp-security-logging) library can also be configured add [CRLF encoding](https://github.com/javabeanz/owasp-security-logging/wiki/Log-Forging) to any logged text.

For example:

```xml
<conversionRule conversionWord="crlf"
                converterClass="org.owasp.security.logging.mask.CRLFConverter" />

<layout class="ch.qos.logback.classic.PatternLayout">
    <Pattern>STDOUT %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %crlf(%msg) %n
    </Pattern>
</layout> 
```

I would also recommend switching to JSON in production over using standard console out as it's a far more flexible (and searchable) format.

## Add automated tooling to prevent security flaws before they are merged.

A final piece in the puzzle is to introduce more automation. There are many options for [static code analysis](https://owasp.org/www-community/controls/Static_Code_Analysis) and as usual a mix (and configuration) is required to get a balanced view.

At Squaddy we use [SonarQube's hosted SonarCloud](https://sonarcloud.io/) and [Snyk](https://snyk.io/). They both have excellent IntelliJ plugins and integrate well with GitHub Actions.

As mentioned no tool is perfect and SonarQube in particular can sometimes blur the lines between *"we think this looks better"*, *"we believe this to be bad practice"* and *"this is a very bad security practice"*. However on balance they are an absolute upgrade and even if after a bit of research I disagree, I am at least wiser for it.

## Conclusion

Thank you for reading and I hope this was useful. My intention was not to deep dive any one subject but more to provide thought on various aspects to security. There is no one size fits all and it doesn't all need to happen in [Sprint 0](https://www.bmc.com/blogs/sprint-zero/).

Ultimately security is an on-going and multi aspect concern. I've learnt over time that the better you understand your business needs, the easier it is to make security decisions.