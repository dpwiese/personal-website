---
title: "PlantUML Diagrams"
date: 2023-12-10T9:29:00-05:00
draft: true
toc: false
images:
tags: 
  - untagged
---

# Introduction

Having previously used [PGF/TikZ](https://github.com/pgf-tikz) almost exclusively for creating diagrams, I was recently introduced to [PlantUML](https://plantuml.com/).
I was quickly impressed by it, and while it's a very different tool than PGF/TikZ, it works very well for quickly creating the certain types of diagrams that it supports.
The PlantUML language is succinct, easy to understand and read, and the expression of diagrams focuses very much on the substance of the diagram, while laying out and styling it quite nicely.
While this lack of full control over the output may be limiting in certain cases, overall I've found it to be highly productive when wanting to generate clear diagrams easily, while focusing on the important aspects.

To see it in action, [planttext.com](https://www.planttext.com) has a great list of sample code that renders the diagram in the browser.
It's a very convenient way to see the various diagram types, get a sense for how it works, and even prototype small diagrams.
Here is an example component diagram:

<img src="/img/posts/plantuml-diagrams/sample-component.png" width="400" />

This diagram was produced by the following code.
I was able to adapt such examples and be productive with PlantUML nearly immediately.

*Sidenote: Highlight.js, which is used to provide syntax highlight on this site, does not support PlantUML. As such, there is no syntax highlighting until it becomes supported, perhaps when I have time to write such support myself.*

```plaintext
@startuml
skin rose
skinparam componentStyle uml2

title Packages - Component Diagram

package "Front End" {
    component [Graphic User\nInterface] as GUI
}

cloud Internet {
}

node "Middle Tier" {
    [Business Logic]
    [Data Access] as DA
    interface IMath as Math
    note left of Math : This is a web\nService Interface
    note right of Math : Notice the\nlabel below
    interface "IItems" as Items

    note left of [Business Logic]
        A note can also
        be on several lines
        like this one
    end note
}

database "PostgreSQL\n" {
    [Stored Procs]
}

GUI -down-> Internet
Internet -down-( Math
[Business Logic] -up- Math : Web Services
DA -up- Items  : .Net
[Business Logic] --( Items
DA .. [Stored Procs]

@enduml
```

# Installation

The following sections outline the basic installation steps for MacOS, but they should be easily adapted for other environments.
The result of `java --version` at the time of writing is shown below.

```zsh {linenos=false}&nbsp;
# openjdk 20.0.2 2023-07-18
# OpenJDK Runtime Environment Homebrew (build 20.0.2)
# OpenJDK 64-Bit Server VM Homebrew (build 20.0.2, mixed mode, sharing)
```

## PlantUML

First install PlantUML.
Go to [plantuml.com/download](https://plantuml.com/download) and download the compiled JAR released under whichever license you choose.
The filename will be something like `plantuml-1.2023.12.jar`.
Place that file wherever appropriate on your local environment; on Mac `/Library/Java/Extensions` seems to be an appropriate place.
While there are certainly other places to keep JAR files, and aliasing and likely other better ways of managing them that I've not looked into, the full path will be used in the samples below.

## GraphViz

Next, install [Graphviz](https://graphviz.org).

```zsh {linenos=false}&nbsp;
brew install graphviz
```

## PDF Support

Installation of the two items above (PlantUML and GraphViz) are all that is necessary to start generating diagrams as PNG from PlantUML source.
To generate diagrams as PDF, some additional dependencies are required, and are listed at [plantuml.com/pdf](https://plantuml.com/pdf) along with links do download them.
**These files must be placed in the same directory as `plantuml*.jar`.**

* `avalon-framework-4.2.0.jar`
* `batik-all-1.7.jar`
* `commons-io-1.3.1.jar`
* `commons-logging-1.0.4.jar`
* `fop.jar`
* `xml-apis-ext-1.3.04.jar`
* `xmlgraphics-commons-1.4.jar`

## Math Support

For math support, see [plantuml.com/ascii-math](https://plantuml.com/ascii-math) for instructions to install [JLaTeXMath](https://github.com/opencollab/jlatexmath), a Java library for displaying mathematical formulas written in LaTeX.
Follow the link at the bottom of the page to download the following dependencies and **place them in the same directory as `plantuml*.jar`.**

* `batik-all-1.7.jar`
* `jlatexmath-minimal-1.0.3.jar`
* `jlm_cyrillic.jar`
* `jlm_greek.jar`

# The Basics: Generating a PlantUML Diagram

## Generate Diagram as PNG

Let's take the following sample from [planttext.com](https://www.planttext.com) and paste it into a file called `sample.puml`.

```plaintext
@startuml
Bob -> Alice: Hello!
@enduml
```

The minimum command to generate an image from the PlantUML source is below.

```zsh {linenos=false}&nbsp;
# Generate sample.png from sample.puml
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -tpng sample.puml
```

The resulting file, `sample.png` is shown below.

<img src="/img/posts/plantuml-diagrams/sample.png" width="200" />

PlantUML supports math like the example below from [plantuml.com/ascii-math](https://plantuml.com/ascii-math).

```plaintext
@startuml
:<latex>\int_0^1f(x)dx</latex>;
:<latex>x^2+y_1+z_{12}^{34}</latex>;
note right
Try also
<latex>\dfrac{d}{dx}f(x)=\lim\limits_{h \to 0}\dfrac{f(x+h)-f(x)}{h}</latex>
<latex>P(y|\mathbf{x}) \mbox{ or } f(\mathbf{x})+\epsilon</latex>
end note
@enduml
```

With math support installed as described above, and the sample PlantUML pasted into a file called `sample-math.puml`, the same basic command below can be used to generated the diagram as a PNG.

```zsh {linenos=false}&nbsp;
# Generate sample-math.png from sample-math.puml
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -tpng sample-math.puml
```

The resulting file, `sample-math.png` is shown below.

<img src="/img/posts/plantuml-diagrams/sample-math.png" width="600" />

## Generate Diagram as PDF

To generate diagrams as PDF, after installing the above dependencies, simply use the `-tpdf` option instead of `-tpng`.

```zsh {linenos=false}&nbsp;
# Generate diagram as PDF
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -tpdf sample.puml
```

A list of all the command line options can be displayed with

```zsh {linenos=false}&nbsp;
# Display help
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -help
```

## Font Support

To change fonts in a PlantUML diagram, it's helpful to first print the list of fonts that PlantUML knows, and verify the names it knows them by.
This can be accomplished by using `listfonts` in a PlantUML diagram and generating an output to a file.
Rather than creating a document with this one command, we can pass the contents of such a document to the PlantUML command.
A couple such options are shown below, first for `zsh`.
(Strangely, this seems to break when attempting to write to PDF, but given it doesn't really matter I did not investigate further.)

```zsh {linenos=false}&nbsp;
# Write list of available fonts to fonts.png (zsh option 1)
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -o $PWD -tpng =(echo "@startuml fonts\nlistfonts\n@enduml")

# Write list of available fonts to fonts.png (zsh option 2)
echo "@startuml\nlistfonts\n@enduml" | java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -o $PWD -tpng -pipe > fonts.png
```

These methods show some different capabilities of PlantUML -- in the first option the filename is specified by `fonts` which follows `@startuml`, and the output *directory* is specified by `-o`.
The second method shows the use of PlantUML's `-pipe` option.
Using `bash` it can be done with the following command.

```bash {linenos=false}&nbsp;
# Write list of available fonts to fonts.png (bash)
printf "@startuml\nlistfonts\n@enduml" | java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -o $PWD -tpng -pipe > fonts.png
```

The `fonts.png` image lists the fonts.
Note the image below has been cropped to only show the first two dozen or so fonts.

<img src="/img/posts/plantuml-diagrams/fonts.png" width="400" />

With the list of fonts now known, the desired font can be specified by `skinparam defaultFontName HelveticaNeue` in a PlantUML file called `sample-helveticaneue.puml` as shown below.

```plaintext
@startuml
skinparam defaultFontName HelveticaNeue
Bob -> Alice: Hello!
@enduml
```

This file is passed to the PlantUML command as before.

```zsh {linenos=false}&nbsp;
# Generate diagram as PNG
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -tpng sample-helveticaneue.puml
```

Now, the generated diagram uses `HelveticaNeue`:

<img src="/img/posts/plantuml-diagrams/sample-helveticaneue.png" width="200" />

There are many other ways to style a digram that are not covered here.
However, it is worth noting that passing `-language` to PlantUML will display, among other things, the various `skinparams` that can be used.

```zsh {linenos=false}&nbsp;
# Display available types, keywords, skinparams, etc.
java -jar /Library/Java/Extensions/plantuml-1.2023.12.jar -language
```

# Building in Docker

If the instructions described in [Installation]({{< relref "#installation" >}}) are too much trouble, or you would for any other reason like to run PlantUML from a container, [plantuml.com/command-line](https://plantuml.com/command-line) has some instructions.
There links to an image can be found on [Github Packages](https://github.com/plantuml/plantuml/pkgs/container/plantuml) and [Docker Hub](https://hub.docker.com/r/plantuml/plantuml).
You can pull one of these images and use the PlantUML installation within to generate your diagrams.

```zsh {linenos=false}&nbsp;
# Pull the image from Docker Hub
docker pull plantuml/plantuml
```

For example, the `sample.puml` diagram above can be built using this container with the following command.

```zsh {linenos=false}&nbsp;
# Generate sample.png from sample.puml using the plantuml/plantuml image
docker run --rm -v $PWD:/puml-diagrams plantuml/plantuml -o /puml-diagrams/out /puml-diagrams/src/sample/sample.puml
```

However, from the [Dockerfile](https://github.com/plantuml/plantuml/blob/master/Dockerfile) for this image, the additional dependencies to generate PDFs, or render LaTeX math are not installed.
So, a modified image is needed that contains all the dependencies.

## Option 1: Modify Container and Save a New Image

One option is to simply copy the necessary into the a container running from the `plantuml/plantuml` image, save that as a new image, and then run this new image.

```zsh {linenos=false}&nbsp;
# Run a new container from the plantuml/plantuml image
docker run --name puml_container plantuml/plantuml

# Copy all the .jar files into puml_container to the /opt directory (where PlantUML is installed)
docker cp jars/. puml_container:/opt

# Save the container as new image
docker commit puml_container dpwiese/plantuml

# Run the newly created dpwiese/plantuml image as a new container
docker run --rm -v $PWD:/puml-diagrams dpwiese/plantuml -o /puml-diagrams/out /puml-diagrams/src/sample/sample-math.puml
```

This was easy enough to do, might become a bit tedious if more configuration was required to a running container.
Another option is to perform this configuration with a Dockerfile and create the new image from that.

## Option 2: Building Our Own Docker Image from Dockerfile

We can leverage a Dockerfile from an existing OpenJDK Distribution such as Amazon Corretto ([Dockerfile](https://github.com/corretto/corretto-docker/blob/main/8/jdk/debian/Dockerfile)) and specify here the installation of dependencies.
In this case, that is `apt install -y graphviz` and `COPY ./jars /user/share/java`.
With those lines added the following commands can be walked through to generate the new image.

```zsh
# Build the new image from the Dockerfil
docker build -t buster-slim-plantuml:1-2023.12 -f Dockerfile .

# Run a container from this newly created image
docker run --rm -v $PWD:/puml-diagrams buster-slim-plantuml:1-2023.12 java -jar /usr/share/java/plantuml-1.2023.12.jar -o /puml-diagrams/out /puml-diagrams/src/sample/sample-math.puml
```

One noticable difference in the way this second image was created is that it doesn't use the PlantUML entry point defined in the first image.
That entrypoint could have been overridden, but it just makes calls to run PlantUML more verbose.
Without this entrypoint though, it makes it easier to pass arguments to `java`, should they be needed.

This image can be further modified as needed and uploaded to a repository where it may be more easily accessible.
As of the time of writing I have not done, and won't describe that process here.

# Using a Makefile

The above steps and commands, while not terribly difficult to run, are a bit cumbersome to remember and modify each time a new diagram is created.
To further simplify this process, it made sense to create a makefile, and adopt a project structure like the one I use at [github.com/dpwiese/checklists](https://github.com/dpwiese/checklists) to create aviation checklists.
With basic knowledge of Make, and the commands above, the makefile can be easily assembled.
There isn't too much to describe here, and the makefile can be found in the sample project repository at [github.com/dpwiese/puml-diagrams-sample](https://github.com/dpwiese/puml-diagrams-sample).
The objective achieved via the use of this makefile was to map the above commands to things like `make fonts`, `make png`, `make pdf`, and so on.
