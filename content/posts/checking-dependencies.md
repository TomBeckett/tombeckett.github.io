---
title: "Checking dependencies for security vulnerabilities"
date: "2022-01-03T01:00:00"
disableShare: true
hideFooter: true
tags: ['java', 'security', 'maven', 'owasp']
categories: ['programming']
---
A security vulnerability is a nightmare scenario. In my [Spring: Security Hardening](/posts/spring-security-hardening/) guide I explain that [static code analysis](https://en.wikipedia.org/wiki/Static_program_analysis) can check your code for common security mistakes. But that raises another question - what about other people's code?

When you consider that your code is actually a fraction of the deployed artifact it gets really scary. 

Luckily for us, there is a variety of automation we can deploy to cover this scenario. 

In this article I'm going to demonstrate integrating [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/) using Maven but it's important to understand what makes a dependency checker successful.

## Security context matters

The [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/) website describes the tool as:

> ...a Software Composition Analysis (SCA) tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies. It does this by determining if there is a Common Platform Enumeration (CPE) identifier for a given dependency. If found, it will generate a report linking to the associated CVE entries.

You may think - *"Well, this sounds like `npm audit`?"*. And, I agree, kind of. The approach `npm audit` takes is a common one but to quote Dan Abramov - it's [Broken by Design](https://overreacted.io/npm-audit-broken-by-design/). 

Why? Well in summary, if a dependency is vulnerable, `npm audit` says every other library using that dependency ***must also*** be vulnerable. This seems logical. 

However as Dan demonstrates, many flagged CVE's only occur at runtime despite the [Create React App](https://reactjs.org/docs/create-a-new-react-app.html) only producing static files - therefore having no runtime itself. These false positives cause a huge amount of confusion and administrative burden. 

Because of this typical 'all or nothing' approach, after a few build failures developers typically turn them off or ignore them entirely. Not what you want for a security tool.

> A vulnerability often requires a certain condition. If you or your library avoids that condition, your dependency is not vulnerable. 

The tool you choose must be ready to handle false positives. I would argue [scanning docker files](https://docs.docker.com/engine/scan/) or `npm audit` fails in this aspect as there is no way to suppress a false positive and reporting completely ignores context.

## The wishlist

Regardless of technology stack I would say we're looking for a tool that must:

- Run using the command line to allow [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration).
- Produce a report with detailed information on *how* the vulnerability affects the exact dependency in *what context*.
- Allow the CVE to be suppressed (ideally by minor version, SHA1, etc) with comments to future revisit.

## Integrating OWASP Dependency-Check

Add the following to your `pom.xml` as a [Maven Plugin](https://blogs.oracle.com/developers/post/mastering-maven-adding-plugins):

```xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>6.5.1</version>
    <configuration>
        <!-- Fail the build if any CVSS score is greater than 8. -->
        <failBuildOnCVSS>8</failBuildOnCVSS>
        <!-- Skip artifacts not bundled in distribution (provided scope) -->
        <skipProvidedScope>true</skipProvidedScope>
        <!-- .Disable Net content-->
        <assemblyAnalyzerEnabled>false</assemblyAnalyzerEnabled>
        <nugetconfAnalyzerEnabled>false</nugetconfAnalyzerEnabled>
        <nuspecAnalyzerEnabled>false</nuspecAnalyzerEnabled>
        <!-- Suppress false positives -->
        <suppressionFile>owasp-suppressions.xml</suppressionFile>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

While there is a wide a variety of [configuration options](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/configuration.html) available to the maven plugin, I want to highlight a few:

- **Line 7:** We will fail any builds which have a high risk vulnerability.
- **Line 9:** We only care about dependencies directly bundled.
- **Lines 11-13:** We completely disable Microsoft DotNet as we're only using Java. I'm unsure why this is not default but it will remove a warning.
- **Line 15:** We've provided an additional configuration file to suppress any false positives.

Lets quickly create a suppression file named `owasp-suppressions.xml` in the root of the project with the following contents:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <!-- Waiting for dependency-check-maven 6.5.2 -->
    <suppress>
       <notes><![CDATA[
       file name: netty-tcnative-classes-2.0.46.Final.jar
       ]]></notes>
       <sha1>9937a832d9c19861822d345b48ced388b645aa5f</sha1>
       <cve>CVE-2014-3488</cve>
       <cve>CVE-2015-2156</cve>
       <cve>CVE-2019-16869</cve>
       <cve>CVE-2019-20444</cve>
       <cve>CVE-2019-20445</cve>
       <cve>CVE-2021-21290</cve>
       <cve>CVE-2021-21295</cve>
       <cve>CVE-2021-21409</cve>
       <cve>CVE-2021-37136</cve>
       <cve>CVE-2021-37137</cve>
       <cve>CVE-2021-43797</cve>
    </suppress>
     <!-- Waiting on https://github.com/jeremylong/DependencyCheck/issues/3894 -->
    <suppress>
       <notes><![CDATA[
       file name: h2-1.4.200.jar
       ]]></notes>
       <sha1>f7533fe7cb8e99c87a43d325a77b4b678ad9031a</sha1>
       <cve>CVE-2021-23463</cve>
    </suppress>
    <!-- We don't use MySQL 5 -->
    <suppress>
       <notes><![CDATA[
       file name: flyway-mysql-8.3.0.jar
       ]]></notes>
       <sha1>2c9fffc16febc8999205b85f1def5b2427cdf42e</sha1>
       <cve>CVE-2007-1420</cve>
       <cve>CVE-2007-2691</cve>
       <cve>CVE-2007-5925</cve>
       <cve>CVE-2009-0819</cve>
       <cve>CVE-2009-4028</cve>
       <cve>CVE-2010-1621</cve>
       <cve>CVE-2010-1626</cve>
       <cve>CVE-2010-3677</cve>
       <cve>CVE-2010-3682</cve>
       <cve>CVE-2012-5627</cve>
       <cve>CVE-2015-2575</cve>
       <cve>CVE-2017-15945</cve>
    </suppress>
</suppressions>
```

I want to highlight a few parts of the configuration schema:

- **Lines 3, 21 & 29:** We've provided a comment on why we believe this to be a false flag. This could also be provided in the `<notes>` tag, but I try to reserve that for the name of the jar.
- **Lines 8, 26 & 34:** We've provided a unique SHA1 of the jar which contains a vulnerability. This will change over time but I believe this to be a good thing. We want to be re-assessing this list frequently!
- **Lines 9-19, 27 & 35-45:** We've provided a list of [Common Vulnerabilities and Exposure (CVE)](https://cve.mitre.org/cve/) numbers to ignore.

> Note: There are *many* other ways to flag a [false positives](https://jeremylong.github.io/DependencyCheck/general/suppression.html) and even [false negatives](https://jeremylong.github.io/DependencyCheck/general/hints.html) so it's worth exploring the documentation. 

Let's next talk about why we're suppressing these vulnerabilities:

1. The `netty-tcnative-classes-2.0.46.Final.jar` related CVEs appears to be a false positive fixed in a [future version](https://github.com/jeremylong/DependencyCheck/issues/3865).
2. The `h2-1.4.200.jar` CVE appear to be a vulnerability in the checker itself *(oh the irony!)* and resolved in a [future version](https://github.com/jeremylong/DependencyCheck/pull/3928).
3. All `flyway-mysql-8.3.0.jar` CVEs are only present when using MySQL 5. It's worth repeating: *These are only false positives for our use case. If we were using MySQL 5 they very much apply*.

### Generating a report

Reports will automatically be generated by running `mvn verify`. 

[When generating a report](https://jeremylong.github.io/DependencyCheck/general/thereport.html) the command line output should be similar to:

```bash
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------------< app.squaddy:api >---------------------------
[INFO] Building api 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- dependency-check-maven:6.5.2:check (default-cli) @ api ---
[INFO] Checking for updates
[INFO] Skipping NVD check since last check was within 4 hours.
[INFO] Skipping RetireJS update since last update was within 24 hours.
[INFO] Check for updates complete (147 ms)
[INFO] 
[INFO] Analysis Started
[INFO] Finished Archive Analyzer (1 seconds)
[INFO] Finished File Name Analyzer (0 seconds)
[INFO] Finished Jar Analyzer (1 seconds)
[INFO] Finished Dependency Merging Analyzer (0 seconds)
[INFO] Finished Version Filter Analyzer (0 seconds)
[INFO] Finished Hint Analyzer (0 seconds)
[INFO] Created CPE Index (1 seconds)
[INFO] Finished CPE Analyzer (5 seconds)
[INFO] Finished False Positive Analyzer (0 seconds)
[INFO] Finished NVD CVE Analyzer (0 seconds)
[INFO] Finished RetireJS Analyzer (1 seconds)
[INFO] Finished Sonatype OSS Index Analyzer (0 seconds)
[INFO] Finished Vulnerability Suppression Analyzer (0 seconds)
[INFO] Finished Dependency Bundling Analyzer (0 seconds)
[INFO] Analysis Complete (13 seconds)
[INFO] Writing report to: /Users/tombeckett/Documents/workspace/api/target/dependency-check-report.html
[WARNING] 

One or more dependencies were identified with known vulnerabilities in api:

flyway-mysql-8.3.0.jar (pkg:maven/org.flywaydb/flyway-mysql@%24%7Bproject.parent.version%7D, pkg:maven/org.flywaydb/flyway-mysql@8.3.0, cpe:2.3:a:mysql:mysql:\$\{project.parent.version\}:*:*:*:*:*:*:*) : CVE-2007-2691


See the dependency-check report for more details.


[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  25.149 s
[INFO] Finished at: 2022-01-03T21:46:29Z
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0
```

**Line 29** mentions *"Writing report to: xxxx/api/target/dependency-check-report.html"* which occurs even on build failure. 

Opening this report will look something like:

![Dependency-Check Report](/images/posts/checking-dependencies/dependency-check-report.png)

There's a wealth of information in this report such as your dependency tree, a list of CVE sources and a list of any vulnerabilities found.

I should also praise the`Copy suppression XML` button which can be pasted into our `owasp-suppressions.xml` file 🙏🏻.

## Finishing up

This really does only scratch the surface of the [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/) tool but I hope I've shown how it meets our requirements above. Since it is command line based, it could easily be integrated into any automated build process.

Checking your dependencies (and their dependencies) is essential as more and more of our programs depend on others.

Finally, as a challenge I recommend exploring the  [configuration options](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/configuration.html) and in particular [hints](https://jeremylong.github.io/DependencyCheck/general/hints.html). This allows dependencies to be 'banned' from programmatically only allowing certain versions of a dependency for example Logback over Log4J. Extremely useful for implementing [security as code](https://www.bmc.com/blogs/security-as-code/).