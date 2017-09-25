# Part 2: MicroProfile Meeting Application - Adding Persistance

## Overview
This lab takes you through how to update the MicroProfile Meeting application to add persistence.

At the time this lab was written MicroProfile had not defined a persistence mechanism. This is not because MicroProfile does not view persistence as important, but indicates the many equally valid choices that could be made for persistence in microservices. In this lab, Cloudant is used but there are other equally valid options like JPA, MongoDB, or Cassandra.

Adapted from the blog post: [Writing a simple MicroProfile application: Adding persistence](https://developer.ibm.com/wasdev/docs/writing-simple-microprofile-application-2-adding-persistence/)

## Prerequisites
 * Completed [Part 1: MicroProfile Meeting Application](https://github.com/IBM/microprofile-meeting)
 * [Eclipse Java EE IDE for Web Developers](http://www.eclipse.org/downloads/)
 * IBM Websphere Application Liberty Developer Tools (WDT)
   1. Start Eclipse
   2. Launch the Eclipse Marketplace: **Help** -> **Eclipse Marketplace**
   3. Search for **IBM Websphere Application Liberty Developer Tools**, and click **Install** with the defaults configuration selected
 * [Git](https://git-scm.com/downloads)
 
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
  
### Step 2. Installing MongoDB
If you completed the previous labs and installed MongoDB, make sure MongoDB is running. If you are starting fresh, make sure you install MongoDB. Depending on what platform you are on the installation instructions may be different. For this exercise you should get the community version of MongoDB from the [mongoDB download-center](https://www.mongodb.com/download-center#community).

1. Once installed you can run the MongoDB database daemon using:
```bash
mongod -dbpath <path to database>
```

The database needs to be running for the application to work. If it is not running there will be a lot of noise in the server logs.
  
### Step 3. Update application to compile against MongoDB API
To start writing code, the Maven pom.xml needs to be updated to indicate the dependency on MongoDB:
1. Open the `pom.xml`.
2. On the editor select the **Dependencies** tab.
3. On the Dependencies tab there are two sections, one for Dependencies, and the other for Dependency Management. Just to the right of the Dependencies box there is a **Add** button. Click the **Add** button.
4. Enter a **groupdId** of `org.mongodb`.
5. Enter a **artifactId** of `mongo-java-driver`.
6. Enter a **version** of `2.14.3`.
7. In the **scope** drop-down, select **provided**. This will allow the application to compile, but will prevent the Maven WAR packager putting the API in the WAR file. Later the build will be configured to make it available to the server.
8. Click **OK**.
9. Save the `pom.xml`.
  
### Step 4. Update MeetingsUtil to convert between MongoDB and JSON-Processing objects
When updating the application, we need to convert between the MongoDB representation of data and the JSON-Processing one. Rather than scatter this around this can be placed in the MeetingsUtil class:

1. Open the `MeetingsUtil` class from **meetings > Java Resources > src/main/java > net.wasdev.samples.microProfile.meetings > MeetingsUtil.java**

2. The first method that is needed takes a MongoDB `DBObject` and returns a `JsonObject`. At the beginning of the file, after the class definition, add the method declaration:
```java
public static JsonObject meetingAsJsonObject(DBObject obj) {
    // method body will go here
}
```

3. This introduces one new class, the `DBObject` class is in the `com.mongodb` package:
```java
import com.mongodb.DBObject;
```

4. The first step in creating a new `JsonObject` is to create a `JsonObjectBuilder` using the `JSON` class. All of these are already used in the class so no new imports will be required. Add the following line to the start of the method:
```java
public static JsonObject meetingAsJsonObject(DBObject obj) {
    JsonObjectBuilder builder = Json.createObjectBuilder();
```

5. The JSON object for a meeting uses a field called `id`. MongoDB uses `_id` for the same function, so `id` in JSON will map to `_id` in MongoDB. To extract the `id` field from the `DBObject` and add it to the JSON, add this line to the method:
 ```java
builder.add("id", (String) obj.get("_id"));
```

6. The `title` field can be directly mapped from JSON to the MongoDB object. The `title` is a String but, to ensure the right add method is called, cast it to a String. Add this line:
```java
builder.add("title", (String) obj.get("title"));
```

7. The `duration` field is a Long. As before it needs to be cast to a Long to ensure the right `add` method is called:
```java
builder.add("duration", (Long) obj.get("duration"));
```

8. If a meeting is running it’ll have a `meetingURL` field. If it isn’t running it’ll return null. If there is no `meetingUR`L field, we don’t want to add it to the meeting returned so we want to only add if `meetingURL` is non-null. Add this:
```java
String meetingURL = (String) obj.get("meetingURL");
if (meetingURL != null)
    builder.add("meetingURL", meetingURL);
```

9. Finally we need to return a `JsonObject`. This can be obtained by calling the `build` method on the `JsonObjectBuilder`. Add this:
```java
return builder.build();
```

10. Next, we need a method that does the opposite, mapping from a `JsonObject` to a `DBObject`. This method serves two purposes: It creates a new `DBObject` and it merges a `JsonObject` to an existing `DBObject`, so that it has two parameters rather than one. After the end of the previous method add this:
```java
public static DBObject meetingAsMongo(JsonObject json, DBObject mongo) {
    // method body will go here
}
```

11. The first thing to do in this new method is check if the `DBObject` passed in is null. If it is null, a new `DBObject` is created. `DBObject` is abstract so a `BasicDBObject` is what will be instantiated. At the beginning of the method add this (there will be a compile error after this but don’t worry, it’ll be fixed soon):
```java
if (mongo == null) {
    mongo = new BasicDBObject();
```

12. Next, the `id` needs to be moved from the `JsonObject` to the new `DBObject`. This should only be done when a new `DBObject` is created because, otherwise, a disconnect between the `URL` and the `JsonObject` could result in an `id` being incorrectly overwritten. Add this:
```java
    mongo.put("_id", json.getString("id"));
}
```

13. This introduced a new class, the `BasicDBObject` which is in the `com.mongodb` package but needs to be imported:
```java
import com.mongodb.BasicDBObject;
```

14. The `title` field is a direct mapping from the `JsonObject` to the `DBObject` but the `JsonObject` contains a `JsonString` which needs to be converted to a String. The `toString` method can’t be used for this because it wraps the String literal with quotes, which isn’t required here. Fortunately `JsonObject` provides a convenient `getString` method for this:
```java
mongo.put("title", json.getString("title"));
```

15. The `duration` field is also a direct mapping but it’s a `JsonNumber` in the `JsonObject`, which needs to be converted to a Long to go into the `DBObject`:
```java
mongo.put("duration", ((JsonNumber) json.get("duration")).longValue());
```

16. This introduced the `JsonNumber`, which needs to be imported:
```java
import javax.json.JsonNumber;
```

17. We want to get the `meetingURL` but, since it might not be there, you can’t use the `getString` method because it’ll throw a `NullPointerException` if there is no field with that name. To get around this, use the `get` method, which returns a `JsonString`. A null check can then be performed and only if it is non-null will it be added to the JSON. The `getString` method must be used since toString wraps the string in quotes:
```java
JsonString jsonString = json.getJsonString("meetingURL");
 if (jsonString != null) {
     mongo.put("meetingURL", jsonString.getString());
 }
 ```
 
18. This introduced the `jsonString`, which needs to be imported:
```java
import javax.json.JsonString;
```

19. Finally return the mongo object and save the file.
```java
return mongo;
```
  
### Step 5. Updating the MeetingManager
The `MeetingManager` currently makes use of a `ConcurrentMap` to store the meetings. All the code that integrates with this needs to be updated. In this section this is done one step at a time; as a result there will be compilation errors until you get to the end:
  
1. Open the `MeetingManager` class from **meetings > Java Resources > src/main/java > net.wasdev.samples.microProfile.meetings > MeetingManager.java**.

2. After the class definition delete the following line from the file:
```java
private ConcurrentMap<String, JsonObject> meetings = new ConcurrentHashMap<>();
```

3. To interact with MongoDB, an instance of `DB` needs to be injected. This can be done using the `@Resource` annotation which can identify which resource from `JNDI` to inject. Add the code where the `ConcurrentMap` was removed (in the previous step):
```java
@Resource(name="mongo/sampledb")
private DB meetings;
```

4. This pulls in two new classes. The MongoDB `DB` class and the Java EE `@Resource` annotation. These need to be imported:
```java
import javax.annotation.Resource;
import com.mongodb.DB;
```

5. MongoDB stores entries in a collection in the database. The first thing to do when writing or reading is to select the collection. To simplify code later on, let’s create a convenience method to get the collection:
```java
public DBCollection getColl() {
    return meetings.getCollection("meetings");
}
```

6. This introduces a new class, the `DBCollection` class, which is in the com.mongodb package. This needs to be imported:
```java
import com.mongodb.DBCollection;
```

7. Find and edit the `add` method:
* Remove the method body from the `add` method. This will be replaced to update the database.
```java
public void add(JsonObject meeting) {
    // code will be added here
}
```

* First get the collection.
```java
DBCollection coll = getColl();
```

* The method is given a `JsonObject` and we need a `DBObject` for MongoDB, so we need to convert. The API can take an existing entry from the database, so first lets see if we can find something from the database using the findOne method:
```java
DBObject existing = coll.findOne(meeting.getString("id"));
```

* This introduces a new class, the `DBObject` class which is in the `com.mongodb` package. This needs to be imported:
```java
import com.mongodb.DBObject;
```

* Next call the `MeetingsUtil` convenience method to convert from `JsonObject` to `DBObject`:
```java
DBObject obj = MeetingsUtil.meetingAsMongo(meeting,  existing);
```

* Finally save the new or changed `DBObject` back into the database:
```java
coll.save(obj);
```

8. Find and update the `ge`t method:
* Remove the method body from the `get` method (this will be replaced to fetch from the database):
```java
public JsonObject get(String id) {
    // code will be added here
}
```

* To get a single entry the collection needs to be obtained, an entry found by id, and then converted to a `JsonObject` using the utility method created earlier. Add the following line to the `get` method:
```java
return MeetingsUtil.meetingAsJsonObject(getColl().findOne(id));
```

9. Find and update the `list` method. The `list` method is slightly more complicated to update. The general structure stays the same but for loop will change. To iterate over entries in a collection a `DBCursor` is used and that returns a `DBObject`. The `DBObject` then needs to be converted to a `JsonObject`. Replace the existing loop that looks like this:
```java
for (JsonObject meeting : meetings.values()) {
    results.add(meeting);
}
```
  with this one
```java
for (DBObject meeting : getColl().find()) {
    results.add(MeetingsUtil.meetingAsJsonObject(meeting));
}
```

10. Find and update the `startMeeting` method:
* This change will radically simplify the code because there is no need to create and merge multiple `JsonObject`s. Instead, you can simply add the `meetingURL` to the existing `DBObjec`t. The `id` and `url` will still need to be fetched from the `JsonObject`. Remove the following lines from the `startMeeting` method:
```java
JsonObject existingMeeting = meetings.get(id);
JsonObject updatedMeeting = MeetingsUtil.createJsonFrom(existingMeeting).add("meetingURL", url).build();
meetings.replace(id, existingMeeting, updatedMeeting);
```

* After the `id` and `url` are fetched, replace the removed code with the following four lines to get a collection, find the meeting entry, set the `meetingURL`, and then save it back to the database:
```java
DBCollection coll = getColl();
DBObject obj = coll.findOne(id);
obj.put("meetingURL", url);
coll.save(obj);
```

11. Save the file.

The` MeetingManager` is now able to persist to a MongoDB database and back. However, the server still needs to be configured to enable it. This consists of two parts: First, server configuration and, second, getting the server runtime set up.
  
### Step 6. Updating the Server Configuration
The server configuration is part of the project so first let’s configure that:

1. Open the `server.xml` from **src > main > liberty > config > server.xml**.

2. There are several lines commented out. These need to be uncommented and then modified. The following lines should have the surrounding comment markers removed:
```java
<featureManager>
    <feature></feature>
</featureManager>
```

3. Between the open and close feature tag add `mongodb-2.0`. It should look like this:
```java
<featureManager>
    <feature>mongodb-2.0</feature>
</featureManager>
```

4. A shared library needs to be defined to be used by the application and the runtime for defining the MongoDB resources:
```java
<library id="mongodriver">
  <file name="${shared.resource.dir}/mongo-java-driver.jar"/>
</library>
```

5. Next the `mongo` needs to be defined. This tells the server where the `mongo` server instance is running:
```java
<mongo id="mongo" libraryRef="mongodriver">
  <ports>27017</ports>
  <hostNames>localhost</hostNames>
</mongo>
```

6. Next, define the database:
```java
<mongoDB databaseName="meetings" jndiName="mongo/sampledb" mongoRef="mongo"/>
```

7. Finally, configure the application so it can see the MongoDB classes:
```java
<webApplication location="meetings-${project.version}.war">
  <classloader commonLibraryRef="mongodriver"/>
</webApplication>
```

8. Save the file.

The next step is to configure the Maven build to make sure all the resources end up in the right place.
  
### Step 7. Updating the Maven POM
The Maven POM needs to be updated to do a few things: It needs to copy the MongoDB Java driver into the Liberty server, define an additional bootstrap property, copy the application to a different location, and ensure that the `mongodb-2.0` feature is installed:

1. Open the `pom.xml` in the root of the project.

2. Select the `pom.xml` tab in the editor.

#### Copy the MongoDB Java driver
1. Search the file for the string `maven-dependency-plugin` you should see this in the file:
```xml
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-dependency-plugin</artifactId>
<version>2.10</version>
<executions>
    <execution>
        <id>copy-server-files</id>
        <phase>package</phase>
        <goals>
            <goal>copy-dependencies</goal>
        </goals>
        <configuration>
            <includeArtifactIds>server-snippet</includeArtifactIds>
            <prependGroupId>true</prependGroupId>
            <outputDirectory>${project.build.directory}/wlp/usr/servers/${wlpServerName}/configDropins/defaults</outputDirectory>
        </configuration>
    </execution>
</executions>
```

This is copying server snippets from dependencies into the server configuration directory. We are going to add to it instructions to download the `mongo-java-driver`, copy it to the `usr/shared/resources` folder, and strip the version off the JAR file name. This last part means we don’t have to remember to update the `server.xm` every time the dependency version is upgraded.

2. To add these additional instructions, add the following lines after the `</execution>` closing tag:
```xml
<execution>
    <id>copy-mongodb-dependency</id>
    <phase>package</phase>
    <goals>
        <goal>copy-dependencies</goal>
    </goals>
    <configuration>
        <includeArtifactIds>mongo-java-driver</includeArtifactIds>
        <outputDirectory>${project.build.directory}/wlp/usr/shared/resources/</outputDirectory>
        <stripVersion>true</stripVersion>
    </configuration>
</execution>
```

#### Place `project.version` into `bootstrap.properties`
The `server.xml` references the WAR by artifact name and version. The version is referenced using a variable which needs to be provided to the server. This can easily be done using the `bootstrap.properties`:

1. Search the `pom.xml` file for the string `<bootstrapProperties>`. You should see this in the file:
```xml
<bootstrapProperties>
    <default.http.port>${testServerHttpPort}</default.http.port>
    <default.https.port>${testServerHttpsPort}</default.https.port>
</bootstrapProperties>
```

2. Before the closing </bootstrapProperties> add the following line:
```xml
<project.version>${project.version}</project.version>
```

#### Copy the application to the `apps` folder
The Maven POM deploys the application into the `dropins` folder of the Liberty server but this doesn’t allow a shared library to be used. So, instead, the application needs to be copied to the `apps` folder:

1. Search in the `pom.xml` for the string ‘dropins’, you should see this:
```xml
<goals>
    <goal>copy-resources</goal>
</goals>
<configuration>
  <outputDirectory>${project.build.directory}/wlp/usr/servers/${wlpServerName}/dropins</outputDirectory>
```

2. Update the word dropins to be apps. It should look like this:
```xml
<configuration>
  <outputDirectory>${project.build.directory}/wlp/usr/servers/${wlpServerName}/apps</outputDirectory>
```

#### Install the `mongodb-2.0` feature from the Liberty repository
The `mongodb-2.0` feature is not in the Liberty server installations that are stored in the Maven repository so it needs to be downloaded from the Liberty repository at build time:

1. Search the `pom.xml` for the string `package-server`, you should see this:
```xml
  <execution>
      <id>package-app</id>
      <phase>package</phase>
      <goals>
          <goal>package-server</goal>
      </goals>
  </execution>
</executions>
```

2. Just before the `</executions>` tag, add the following lines to cause the `mongodb-2.0` feature to be installed:
```xml
<execution>
    <id>install-feature</id>
    <phase>package</phase>
    <goals>
        <goal>install-feature</goal>
    </goals>
    <configuration>
        <features>
            <acceptLicense>true</acceptLicense>
            <feature>mongodb-2.0</feature>
        </features>
    </configuration>
</execution>
```

3. Save the `pom.xml`.
  
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
