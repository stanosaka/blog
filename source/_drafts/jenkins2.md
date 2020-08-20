---
title: jenkins2
tags: jenkins2
---
# Technologies used
- Jenkins -workflow management
- Git - source management
- Gradele - build engine and automation
- SonarQube - code analysis and metrices
- Jacoco -code coverage
- Artifactory - binary artifact storage & management
- Docker - container and image creation

## Jenkins Domain Specific Language (DSL)
- Groovey-based
- Allows for orchestrating process steps
- Can be wriiten and strored a pipeline script in job itself or as an external Jenkinsfile stored in the repository

## Working with the pipeline script editor
- Syntax error highlighting - not always valid
- Hover over X to see explanation
- Automatic pair generation for () and {}
  - Nice, but can trip you up
  - Type "stage ("and you get "stage()"
  - Type node {, hit return, and closing} is auto-generated
- Quotes are significant; Double check after copy and paste from Word or PDF
- Rule of thumb: "If the text isn't green, the quotes aren't clean"
- Blocks can be collapsed/expanded

## Terminology- Node
- Binds code to a particular system (agent) ro run via a label
- Has defined number of executors - slot for execution
- As soon as an executor slot comes open on that agent, the task is run 
- Code between {} forms program to run  
- A particular agent can be specified in node(agent)
- Creates an associated workspace (working directory) for duration of the task
- Schedules node steps ro run in build queue
- Best practice: Do any significant work in a specific node 
   - Why? The default is for a pipeline script to run on the master with only a lightweight executor-expecting limited use of resources.
```
node {
  // stages
}

node ('agent_1'){
  // stages
}

####
parallel (
    win: { node ('win64'){
      ...
    }},
    linux: { node ('ubuntu'){
      ...
     }},
)
```
## Git
- Distributed version control system
- Open source
- Available for multiple platforms
- Roots in source control efforts for Linux Kernel (2005)
- Primary developer- Linux Torvalds
![git](https://i.imgur.com/oEtkGOc.jpg)
**workflow**
git init or git clone
<create or edit content>
git add .
git commit -m "<msg>"
git push origin master  

## Gradle   
- General purpose build automation system
- Middle ground between ANT and Maven
  - ANT -maximum flexibility, no conventions, no dependency management  
  - Maven - strong standards, good dependency management, hard to customize, not flexible
- Offers usedful "build-in" conventions, but also offers ease of extension
- Useful for developing build standards
- Rich, descriptive language (DSL - Groovy) - Java core (or Kotlin)
- Built-in support for Groovy, Java, Scala, OSGI, Web, etc.
- Multiple dependency management models 
  - Transitive - can declare only top-level and interface with Maven or Ivy
  - Manual - can manage by hand
- Toolkit for domain-specific build standards
- Offers useful features - upstream builds, builds that ignore non-significant changes, etc.
- Derives a lot of functionality from plugins

**Gradle structure**
- Two basic concepts - projects and tasks
- Projects
  - Every Gradle build is made up of one or more projects
  - A project is some component of software that can be built (examples: library JAR, web app, zip assembled from jars of other projects)
  - May represent something to be done, rather than something to be built. (examples: deploying app to staging area or production)
- Task
  - Represents some atomic piece of work which a build performs (examples: compiling classes, creating a jar, generating javadoc, publishing archives to a repository)
  - List of actions to be performed
  - May include configuration
![gradle build](https://i.imgur.com/cPAtxOu.png)
```
def destDirs = new File('target3/results')

task createTarget(type: DefaultTask){
  doLast {
        destDirs.mkdirs()
        }
}

task createTarget << {
        destDirs.mkdirs()
}
```
## Global tools and pipeline
- Tools defined in global configuration
- DSL keyword 'tool'
- In pipeline script
  - def variable for tool location
  - assign variable to "tool <toolname>" from global configuration
  - Use variable in place of location in script
![tools](https://i.imgur.com/9VadT3I.jpg)

## Pipeline DSL Syntax
- Many functions (per Groovy) expect closures (code blocks enclosed in {})
- May steps can only run inside of a node block (on an agent)
- Steps in DSL always expect mapped (named) parameters
  - Example: git branch: 'test', url: 'http://github.com/brentalster/gradle-greetings.git'
  - Two parameters: "branch" and "url"
- Shorthand for git([branch: 'test', url: 'http://github.com/brentalster/gradle-greetings.git'])
  - Groovy permits not putting parentheses for parameters to functions
  - Named parameter syntax is shorthand for Groovy mapping syntax of [key:value, key:vaule]
- Default object is "script", so if only passing one parameter or required parameter, can omit that.
  - bat([script: 'echo hi'])= bat 'echo hi'

## Snippet Generator
- Facilitates generating groovy code for typical actions
- Select the operation
- Fill in the arguments/parameters 
- Push the button
- Copy and paste

## Shared Pipeline library
- Allows you to create a shared SCM repository of pipeline script code and functions
- Two modes:
  - General mode: Any SCM repository ("external library")
  - Legacy mode: Git repository hosted by Jenkins ("internal library")
![repository structre](https://i.imgur.com/v8l0D0p.jpg)

## Defining Global Shared External Pipeline Libraries in Jenkins 
- Defined in Manage Jenkins-> Configure System
- Usable by any script 
- Trusted - can call any methods
  - Could put potentially unsafe methods in libraries here to encapsulate them
  - Limit push access
- Name - will be used in script to pull in library
- Version = any version identifier understood by SCM
  - example: Git: branch, tag, commmit SHA1
- Modern SCM - plugin with newer api to pull specific identifier/version of library
- Legacy SCM -existing plugins
  - To pull specific identifier version, include ${library.LibraryName.version} in SCM configuration 

## Using Libraries and resources
- External Libraries
  - Default version: @Library('LibraryName')
   >> @Library('Utilities')
  - Override version: @Library('LibraryName@version')
   >> @Library('Utilities@1.4')
- Resources for External Libraries
  - use libraryResource to load from /resources directory in shared library structure
  - def input = libraryResource 'org/foo/bar/input.json'
- Third-party Libraries
  - @Grab
  

