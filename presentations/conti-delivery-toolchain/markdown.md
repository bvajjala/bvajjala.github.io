# Continuous Delivery Toolchain

### Created by [Balaji Vajjala](https://bvajjala.github.io) / [@BVajjala](https://twitter.com/Bvajjala)



## Principal DevOps Consultant/Solution Architect

  Hi, my name is Balaji Vajjala and I'm the Chief DevOps Solution Architect here at Angie's List. Today  Here I will be talking about how we implemented a Continuous Deployment pipelines, DevOps Culture and other Misc Topics.
  Hope you will enjoy this presentation.


## Who Am I?

* A Full Stack DevOps Engineer/Solution Architect for last 10+ years!
* Original !sysadmin! and !Developer! since late 90's 
* Now working at Angie's List and Certainly Not a professional speaker



# Continuous Delivery Phases

* Orchestration and Deployment Pipeline Visualization
* Version Control
* Continuous Integration
* Artifact Management
* Test and Environment Automation
* Server Configuration and Deployment
* Monitoring and Reporting



### Orchestration and Deployment Pipeline Visualization

* Orchestration tools are the backbone of any CD system. They allow teams to build an effective sequence of deployment pipeline steps by integrating with their entire toolchain. These tools can also provide visualization utilities, which are important for enabling the full involvement of stakeholders from all departments. 
* For pipeline orchestration and visualization, you can use a dedicated deployment pipeline tool or you can use an application release automation (ARA) solution. 
* Whichever direction you take for your orchestration tool, you should be sure that it helps your team detect and expose delays at each stage of the pipeline, including wait times between stages. 
* Visualization and orchestration working in tandem allow teams to quickly identify the places they should optimize first. 




### Orchestration and Deployment Pipeline Visualization 

* Dedicated Deployment Pipeline Tools  : Jenkins (with plugins or 
through Cloudbees), ThoughtWorks Go, Atlassian Bamboo

* ARA                                   : ElectricCommander, CA Lisa 
IBM UrbanCode, XebiaLabs XL 

* Orchestration Engine                  : MaestroDev, CollabNet
  



### Version Control

*  Most software development teams use a version control system for the files they consider source code. however, many organizations forget to include configuration files, such as the configuration that defines the build and release system. 
*  All text-based assets should be stored in a version control system that everyone can easily access. 
*  The code changes should be very easy to review, line-by-line (ideally in a web browser), with a pull or merge request.

### Version Control

Version Control: Git, Mercurial, Perforce, Subversion, TFS



   
### Continuous Integration
* CI tools can support orchestration and visualization, but their core functionality is to integrate new code with the stable main release line and alert stakeholders if any new code would cause issues with the final product. 
* This makes it easy for teams to combine work on different features while keeping a master code branch ready for release. 
* It should feel natural to integrate many times a day. Teams should also connect a code metrics and profiling utility that can stop integrations if certain metrics reach an undesirable threshold.  



   
### Continuous Integration

* CI: Jenkins, Travis CI, ThoughtWorks GO, CircleCI, JetBrains TeamCity, Atlassian Bamboo

* Code Metrics: SonarQube, SLOC (and variants), SciTools Understand




### Artifact Management
* Packaged artifacts, rather than the application’s raw source code, are the focus of deployment pipelines. 
* Artifacts are assembled pieces of an application that include packaged application code, application assets, infrastructure code, virtual machine images, and (typically) configuration data. 

* Artifacts are identifiable (unique name), versioned (preferably semantic versioning), and immutable (we never edit them). Together, these artifacts allow developers to build a bill of materials (BOM) package describing the exact versions of all the artifacts in a particular version of their software system. Package metadata identifies when and how the package was tested or deployed into a particular environment. 

* Artifact management is most effective with an artifact repository manager. Artifact repositories contain a complete artifact usage history, similar to the way version control systems track source code changes. 
* They use dependency resolution between the package versions and allow the system to build a dependency graph from the hardware all the way up to the user interface of the application. The ability to verify dependencies through an entire system is powerful for tracking exactly what was (or will be) tested or deployed.



   
### Artifact Management

* Version Control Systems: Git, Subversion, Mercurial

* Artifact Repository Managers: Archiva, Artifactory, Nexus, OR roll-your-own with zip files, metadata, shared storage, and access controls

* Language-specific Package Managers: Composer (PHP), Ruby Gems, npm (Node.js), Python PIP

* OS-level Package Managers: APT, Chocolatey, RPM



 
### Test and Environment Automation
* The only manual testing in a deployment pipeline should be for tests that are tough for a computer to handle, such as exploratory testing, inspection of user interface designs, and user acceptance tests. The rest should be automated

* Tools for automated testing should operate in a completely headless (non-interactive) manner and be lightweight enough to run across many test servers simultaneously. 

* Teams also need to create testing environments on-demand by using environment automation tools that can provision a VM and configure an environment template.



 
### Test and Environment Automation

* Test Automation: JMeter, Selenium/WebDriver, Cucumber (BDD), RSpec (BDD), 
SpecFlow (BDD), LoadUI (Performance), PageSpeed (Performance), Netem(Network Emulation), SoapUI (Web Services), Test Kitchen (Infrastructure)

* Environment Automation: Vagrant, Docker, Packer



 
## Server Configuration and Deployment

Current deployment tools support three models:
* Push model: Manages the distribution and installation of packages to 
multiple remote machines. It’s a good choice for smaller systems 
because its usually simple and quick.

* Pull model: Requires an infrastructure configuration tool such as Chef or Puppet.  Supporters say it scales better than push and like that it treats the 
deployment of application code as another step in configuring the infrastructure.

* Hybrid model: Uses a push tool to trigger a pull client on target servers.  
For any of the three models, your team must ensure that the process is fully automated, provides detailed information with standard output and error messages, and allows easy and rapid rollback to a stable state.




### Server Configuration and Deployment

* Push deployment: Capistrano, Fabric, ThoughtWorks Go, MSdeploy, Octopus, RunDeck, various CI and build tools, various ARA tools

* Pull deployment: Ansible, Chef, CFEngine, Puppet, Salt.




### Monitoring and Reporting

* Monitoring your system logs is essential for spotting problems and halting 
the deployment pipeline. 
* Rather than manually collecting logs from each machine in an environment, logs should be shipped to a central store that indexes them and makes them available for searching via a web browser. 
* This is a crucial capability for a Continuous Delivery environment. 
* The log store should be connected to all environments (including the developers system) to speed up problem diagnosis and resolution. 
* Most monitoring tools should work in a dynamic infrastructure and integrate with yours through scripted configuration. 




### Monitoring and Reporting

* Log Aggregation & Search: Fluentd, Graylog2, LogStash, nxlog, Splunk

* Metrics, Monitoring, Audit: Collectd, Ganglia, Graphite, Icinga, Sensu, ScriptRock




## My Social Media Presence

  * [My LinkedIn Profile](https://www.linkedin.com/in/bvajjala)
  * [My Twitter Handle](https://twitter.com/Bvajjala)
  * [My Blog](https://bvajjala.github.io/)
  * [My Facebook Profile](https://www.facebook.com/bvajjala)
  * [My Resume](https://bvajjala.github.io/about/resume/)
  * [Contact me](mailto:bvajjala@gmail.com)


## The End