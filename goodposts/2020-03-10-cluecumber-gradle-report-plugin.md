---
layout: post
title: Cluecumber Gradle Report Plugin
---
# Host your own Cluecumber Gradle Plugin to workaround Corporate Firewall
I'm using cucumber at work and I use the JUnit runner and gradle.
I'm severly underwhelmed with the free options for reporting.  **placeholder for thoughts on this**  
I googled around and found the https://github.com/trivago/cluecumber-report-plugin which is a maven plugin.  I chose to use it over the masterthought report generator mainly because it can expand all the step hook outputs and scenario hook outputs with a button click.  Logs and error messgaes are important for my tests and I can't have my users clicking too many times.  
Problem is I use gradle, not maven,  so I looked and then I found https://github.com/JavaanseHZ/cluecumber-report-gradle-plugin.  
I went to use it and... boom, can't use it because my company's firewall blocked the request.
I thought hmm.. well I can get other JARs from the company's artifactory JCenter proxy,
are gradle plugins not on there?  
https://mvnrepository.com/artifact/de.javaansehz.cluecumber-report-gradle-plugin/de.javaansehz.cluecumber-report-gradle-plugin.gradle.plugin/1.1.4  
Notice the "Note:" in that link, it says the artifact is in plugins.gradle.org/m2. 

https://discuss.gradle.org/t/what-is-the-correct-url-to-mirror-plugins-repository/12668
As discussed there, there are plugins in JCenter and S3. 
I don't have enough arcane sorcery knowledge to explain clearly but it suffices to say, I can't get the plugin from behind the firewall, so what do I do?

I decided to try to look a bit deeper at the gradle plugin and maven plugin
and try to see how they work and gradle plugins work in general.  I learned that there really is not much involved with making plugins.  The actual plugin part is a
thin layer in between the functionality (ie. report generation) and the core (ie. gradle). 

This article was helpful:  https://docs.gradle.org/current/userguide/custom_plugins.html

Interestingly, the cluecumber gradle plugin names the maven plugin as a dependency.
In that, the functionality of the maven plugin is used, and the maven plugin part is discarded.  The gradle plugin consists of defining the gradle task to generate the report,
and the collecting of flags to supply to the report generating classes. 

So I thought 1 option would be, can I rip the code out of cluecumber-report-gradle-plugin
and stick it in my project's `build.gradle` and follow something like the tutorial?
I tried that and didn't like where it was going so then of course I noticed the 2nd
half of that tutorial, standalone project.

I then realized, the reason I can't use the gradle plugin is that it wasn't published to JCenter or some repository I could access.  Well, what if I just deploy it myself
to artifactory?  I have my own repo there for my projects, can I just deploy it there?

I tried a really easy solution and luckily it worked.
I have code in my `build.gradle`'s of other projects to publish JARs to artifactory.
I simply copy pasted that snippet into the `build.gradle` of the `cluecumber-report-gradle-plugin`,
built it locally on my machine, then ran the artifactoryPublish task.

Then manually inspected artifactory and it indeed was there.  Then, to use the plugin in my test runner projects, I just had to tell `build.gradle` to look in my company's artifactory for plugin JARs.

Basically the top of my `build.gradle` looks like this:
```

buildscript {
repositories {

    maven {
        url 'https://artifactory.ACME.com/artifactory/automated-testing'
    }
    maven {
     url 'https://artifactory.ACME.com/artifactory/jcenter-cache'
     }
     maven {
         url 'https://artifactory.ACME.com/artifactory/jcenter'
     }
}
dependencies {
    classpath "de.javaansehz:cluecumber-report-gradle-plugin:1.1.5-SNAPSHOT"
}
}
plugins {
id 'java'
}
apply plugin: "de.javaansehz.cluecumber-report-gradle-plugin"
```

And to use it all I have to do is stick this some where
```
cluecumberReports {
sourceJsonReportDirectory = projectDir.getAbsolutePath() + "/target"
generatedHtmlReportDirectory = projectDir.getAbsolutePath() + "/target/cluecumber-html"
}
```
The getAbsolutePath()  was a gotcha for me (gotme...?) it seems that only absolute paths
can be used there though the examples did not indicate that.
The symptom was seeing successfully generated reports with no data, ie. no scenarios, no steps, all zeros.

Another note about Cluecumber, it does not insert line breaks \<br> in the scenario/step outputs.
ie. if you have scenario.write() in your hooks and you have linebreaks, they wont
render in the HTML report. 
Somehow the masterthought report generator does render the linebreaks.  Haven't looked at the code yet.
But, a workaround for Cluecumber is to call scenario.write() for each line in your string.
ie split the string by "\n" and call scenario.write() with each line.
Converting "\n" to \<br> in my step hook was unacceptable, as I don't want HTML inside the cucumber.json,
but this workaround was an OK compromise.

