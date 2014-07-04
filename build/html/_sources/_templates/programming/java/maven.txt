


========================================
Maven 
========================================

Introduction
========================================
Maven is a wildly used project builder tool in programming.


How we use maven
----------------------------------------
we can test, install, deploy the project by maven.
When we use maven, we must config the settings.xml in maven's configuration.We can use it as its defualt settings, it will download the dependencies from maven website.
But if we use nexus or other maven server, we must add repository{id, url, release, snapshot} into the settings file.
After we build a maven project, we must config the pom.xml.
In the pom file, we add project information like {parent{groupeId, artifactId, version, relativePath}, artifactId, description, properties, dependencies{dependency{groupId, artifactId, version}}}.

When we change pom file(we change dependencies),we have to maven update project to download the denpendencies.
Then we make test of maven, make install to build the project and create jar or war package.
And we maven deploy to upload the builded package to nexus. 
