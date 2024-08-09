# sonar-scanner-docker
Containers for [SonarSource](https://sonarsource.com/) Scanner, with C, C++, ObjC, and C# support!

Because SonarQube is a great product I enjoy using, so you should too. 

# Why?
Quoting SonarSource:

> [NB: These Docker images are not compatible with C/C#/C++/Objective-C projects.](https://github.com/SonarSource/sonar-scanner-cli-docker/blob/master/README.md)

C, C#, C++, and ObjC are arguably the areas where you MOST need SonarSource's tools! Nor is it possible to say, load their container up, slap in the missing bits, and go off to the races any longer with the move to non-root user. (Trust me, that's how I used to do it.) And while I 150% agree that a non-root user is important, and that loading up your SBOM with boost and it's dependencies is not great? It's a lot less great to lose your scanning capabilities. 

# How do I use it?

Instead of the official SonarSource Docker images, use these! They're hosted on ghcr.io because I'm not paying Docker Hub. The source is right here for you to examine too, because hello, it's a code quality and static analysis tool. These images are built the same way SonarSource does their own, mostly. (We don't have the infrastructure or their signing keys. Sorry.) It's a pretty simple process. We grab the latest version of the sonar-scanner-cli or dotnet tool. We confirm the signature. And we toss that into a container that installs the bits you need to perform C, C++, ObjC, or C# analysis. We reuse as much of SonarSource's original code as possible, because the whole point is extending the functionality, not changing it. (Except dotnet, we'll get to that in a minute.)

Because these behave nearly identically to SonarSource's official images, [please refer to the official SonarSource documentation](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/) for C, C++, and ObjC.

If you encounter behavior DIFFERENT from the documentation, **please open an issue against this repository NOT SonarSource!** This is not an official SonarSource repository, it is not maintained by SonarSource, and it is not the responsibility of SonarSource.

## Using it with dotnet

Because SonarSource does NOT provide an official .NET container, use is slightly different. The .NET container uses the same variables as the C/C++/ObjC container, but uses the `.NET Core global tool invocation` method. Because additional arguments ARE required, you must provide an additional `SONAR_PROJECT_KEY` environment variable which populates `/k:`. 

All arguments will be passed verbatim to the `dotnet build` step, NOT the `dotnet sonarscanner` step. Please see [the .NET documentation for build](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build) for information on these arguments.

**ALL** problems with the dotnet scanner, please open an issue in this repository and this repository alone. SonarSource does not provide or support any .NET scanning Docker images at this time. If you encounter a .NET bug in this scanner, **ALSO** please open the issue in this repository, NOT the .NET repositories. This will ensure any issues are validated, identified, and prioritized much more quickly.

# Okay but why?

Look. I've worked with SonarQube for a few years now. And maybe there's better tools - I wouldn't know. Because they either aren't open source, they don't work in Kubernetes worth a crap if at all, they don't play nice with Docker, their C# rulesets are *just wrong*, or take your pick! SonarQube is 1) GPLv3 2) free 3) a great product 4) easy for entry level people and non-developers to use 5) highly accurate. 

SonarQube is a product that in most environments, a competent senior engineer can have up and running in literal *minutes*. And which can provide excellent guidance for both risks and priorities to nearly anyone in the company at a glance. You really should check it out. It's easy to spin up a test instance in a few minutes.

## Why not submit a PR to SonarSource and add these to the official images?

For one very simple reason: the official images have made an operational and maintenance decision to not include these features. While I make it sound (and sometimes look) easy, it is very much not. They would need to take on the additional workload of monitoring all of the supporting/dependent packages for any vulnerabilities on a daily basis, and there are hundreds. They would likely need to write many, many additional unit tests. And it would make their images (and Docker Hub bills) much, much larger. A combined C/C# image even with mimization tips in at nearly 1GB - it's a chonky boi.

I don't promise (at all) to be tracking the 300+ dependencies involved here, because I can't. It's just not reasonable. What I do promise is that any security patches published are integrated to the `latest` tag within 7 days of release, and I'll do my best to fix any scanning problems specific to these images. Could it be better? Sure, probably. But that would require a significant investment of time, effort, and above all else, money. 

## But the dotnet one isn't signed?

So, hi. I'm @rootwyrm. As you can see from my profile, I'm actually part of the .NET Foundation (just not a very visible one. I'm more a behind the scenes type.) And right off the bat, I'm going to tell you on behalf of your infrastructure engineers: please use the [SonarLint for Visual Studio](https://www.sonarsource.com/products/sonarlint/) extension first. It'll make everyone's lives easier. Great. Thanks. Secondly, they do sign their nuget package - this is done in their Azure pipelines. [You're welcome to look for yourself.](https://github.com/SonarSource/sonar-dotnet/blob/master/azure-pipelines.yml) 

That signing certificate is checked by nuget.org (Microsoft) because SonarQube is a Reserved Prefix. If an upload isn't signed by that certificate, it's automatically rejected with violence. Secondly, the signature verification happens transparent to users; as of .NET 8, DOTNET_NUGET_SIGNATURE_VERIFICATION defaults to true. We only use .NET 8 and later with a default environment. We do NOT add any root certificates, so a package which is improperly signed will return an error during installation. So it's signed, we just don't need to do a manual check for the signing. (And yes, you may pass this explanation on to your security team. For more information, [see this Microsoft page](https://learn.microsoft.com/en-us/dotnet/core/tools/nuget-signed-package-verification).)

# ... wait, these images are _opinionated_!
Yes and no, no and yes. Upstream SonarSource uses Amazon Linux 2023 for the actual scanning container, which is an RPM/DNF glibc based distribution. These images use Alpine Linux for the scanning container, which is an APK and musl based distribution. This is done for a couple reasons. One, it's a much smaller base container. Way, way smaller with far, far fewer components. Two, sonar-scanner-cli is designed to work just fine on Alpine Linux (which is what was used previously.) Three, you really have no reason to care about the underlying OS because these containers are for scanning files. 

The objective here is to provide you, the user, with a sonar-scanner-cli Docker image in the minimum reasonable size, with maximum capability, and a minimum of _behavioral_ deviation. Which means we use the same UID, the same GID, the same entrypoint, and the same environment variables so that these containers are completely interchangeable with the SonarSource images.

# This is fantastic and saved (me|my company|my project) (a ton of work|from an embarrassing security issue|possibly the entire world), how can I (help|say thanks)?

Found an issue and found a fix? Send a pull request!

These specific containers helped you out, the Github Sponsors button over there on the right. [Or click here](https://github.com/sponsors/rootwyrm). 

If you don't want to give me your hard earned money, donate to your local animal shelter or animal protective league. (Though you should do this even if you don't want to thank me.)

And if you want to thank SonarQube/SonarCloud, [consider purchasing from SonarSource](https://www.sonarsource.com/plans-and-pricing/).
