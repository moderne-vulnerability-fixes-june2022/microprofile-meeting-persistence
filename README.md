# microprofile-meeting-persistance

## Overview
This lab takes you through how to update the MicroProfile Meeting application to add persistence.

At the time this lab was written MicroProfile had not defined a persistence mechanism. This is not because MicroProfile does not view persistence as important, but indicates the many equally valid choices that could be made for persistence in microservices. In this lab, Cloudant is used but there are other equally valid options like JPA, MongoDB, or Cassandra.

## Prerequisites
 * Completed [Part 1: MicroProfile Meeting Application](https://github.com/IBM/microprofile-meeting)
 * [Eclipse](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/mars2)
 * WebSphere Developer Tools (WDT)
   * To install WDT, download and start Eclipse, then drag and drop [Install WDT](http://marketplace.eclipse.org/marketplace-client-intro?mpc_install=1778478) to your running Eclipse workspace to install WebSphere Developer Tools on to the toolbar to start the WDT installer.
 * [Git](https://git-scm.com/downloads)
 * [Bluemix Account](bluemix.net)
 
 ## Steps
 ### Step 1. Check out the source code

  #### From the command line
  Run the following commands:
  
  ```
  $    git clone https://github.com/IBM/microprofile-meeting-persistance.git
  ```

  #### In Eclipse, import the project as an existing project.
  1. In Eclipse, switch to the Git perspective.
  2. Click **Clone a Git repository** from the Git Repositories view.
  3. Enter URI `https://github.com/IBM/microprofile-meeting-persistance.git`
  4. Click **Next**, then click **Next** again accepting the defaults.
  5. From the **Initial branch** drop-down list, click **master**.
  6. Select **Import all existing Eclipse projects after clone finishes**, then click **Finish**.
  7. Switch to the Java EE perspective.
  8. The meetings project is automatically created in the Project Explorer view.
  
  ### Step 2. Deploy Cloudant Instance
  
  ### Step 3. Update application to compile against Cloudant
  
  ### Step 4. Update MeetingsUtil to convert between Cloudant and JSON-Processing objects
  
  ### Step 5. Updating the MeetingManager
  
  ### Step 6. Updating the Server Configuration
  
  ### Step 7. Updating the Maven POM
  
  ### Step 8. Running the application
There are two ways to get the application running from within WDT:

 * The first is to use Maven to build and run the project:
 1. Run the Maven `install` goal to build and test the project: Right-click **pom.xml** in the `meetings` project, click **Run As… > Maven Build…**, then in the **Goals** field type `install` and click **Run**. The first time you run this goal, it might take a few minutes to download the Liberty dependencies.
 2. Run a Maven build for the `liberty:start-server goal`: Right-click **pom.xml**, click **Run As… > Maven Build**, then in the **Goals** field, type `liberty:start-server` and click **Run**. This starts the server in the background.
 3. Open the application, which is available at `http://localhost:9080/meetings/`.
 4. To stop the server again, run the `liberty:stop-server` build goal.

 * The second way is to right-click the `meetings` project and select **Run As… > Run on Server** but there are a few things to note if you do this. WDT doesn’t automatically add the MicroProfile features as you would expect so you need to manually add those. Also, any changes to the configuration in `src/main/liberty/config` won’t be picked up unless you add an include.

Find out more about [MicroProfile and WebSphere Liberty](https://developer.ibm.com/wasdev/docs/microprofile/).

## Next Steps
Part 3: [MicroProfile Application - Using Java EE Concurrency](https://github.com/IBM/microprofile-meeting-concurrency)
