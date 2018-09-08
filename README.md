[![The Library Code GmbH](the_library_code_gmbh.png)](https://www.the-library-code.de)

[![ZHAW](zhaw_logo.png)](https://www.zhaw.ch)

[![DSpace Logo](dspace_logo.png)](http://www.dspace.org)

# Duplication Detection Service for DSpace 6.2 (JSPUI)

This extension to [DSpace](http://www.dspace.org) was developed by [The Library Code GmbH](https://www.the-library-code.de) with the support of [Zürcher Hochschule für Angewandte Wissenschaften](https://www.zhaw.ch). It extends DSpace 6.2 JSPUI by an duplication detection.

Enduring the submission, a user should get notified if an item with a similar title is already archived in the repository. The possible duplicates are presented to the user who can decide to remove his/her submission, to decide later about the possible duplicates or to continue the submission nevertheless. This duplication detection should run as early as possible to avoid a lot of work while entering metadata to the submission form.

Enduring the workflow step a reviewer is warned if items with a similar title are detected in the repository. The warning gets displayed on the page on which the reviewer decides whether to accept or decline a submission to the repository. A note will be displayed if no duplicates were found.

The Duplication Detection Service looks for items with similar titles. To find items that have similar but not totally equal names, fuzzy search is used. The Levenshtein algorithm performs these searches. It calculates a measure called edit difference. The edit difference counts how many characters have to be deleted, swapped or changed to transform one string into another one. By default, the Duplication Detection Service considers two items as similar if the edit distance between there title does not exceeds 8 changes. This is configurable (see below).

The Duplicates Detection Service does not show possible duplicates if a user does not own the permissions to see these items. Only the items that are readable for the current user will be considered as possible duplicates. Unfinished submissions (so called workspace items) and other versions of the item being submitted will never be considered a possible duplicate. If a user is able to review items in the workflow process also workflow items with similar titles will be presented as possible duplicates. If a user is able to administrate withdrawn items those will be considered too.

Possible duplicates are listed in a simple citation style containing the names of the first two authors, the title, the year, the publisher name and place and a link to the item. The citation style cannot be changed by configuration currently.

## Prerequisites

In its current form the Duplication Detection Service is developed for DSpace 6.2, JSPUI and PostgreSQL only. Neither XMLUI nor Oracle are supported currently. Why it was **not** tested, it probably will work on newer versions of DSpace 6 as well. It probably won't work without further development on DSpace 7 and above. If you need support to develop a similar solution for XMLUI, do need support for Oracle or any other version of DSpace, please don't hesitate to contact [The Library Code GmbH](https://www.the-library-code.de).

Besides the prerequisites of DSpace, the Duplication Detection Service requires the PostgreSQL Extension "fuzzystrmatch" which contains an implementation of the Levenshtein algorithm.

## Branches

We will add one branch per supported version. Currently DSpace 6.2 is the only supported version: [dspace-6.2-addition](https://github.com/the-library-code/deduplication/tree/dspace-6.2-addition).


## Installation

To install this add-on you will have to install the PostgreSQL extension "fuzzystrmatch" manually. As a postgres super user you have to run the sql query `CREATE EXTENSION fuzzystrmatch;`.  This can be done by using the psql command line client: `sudo -u postgres psql dspace -c "CREATE EXTENSION fuzzystrmatch;"`.

Furthermore, you will need to know how to compile and update DSpace. There are two ways to install this add-on's source code, both are documented below. After installing the add-on please change your configuration as described in the section "Configuration" below. The Library Code also offers support to install this and other add-ons.

### Install the add-on by changing two poms and the message catalog

Please remind to install the PostgreSQL extension "fuzzystrmatch", as described above.
To install this add-on using maven's capability to automatically download and include all necessary files, you need to change two pom files and the message catalog.

At the end of the file `[dspace-source]/dspace/modules/additions/pom.xml` you will find a section `<dependencies>...</dependencies>`. At the end of this section, right before the closing tag, you need to add the following lines:

<pre>
  &lt;dependency&gt;
    &lt;groupId&gt;de.the-library-code.dspace&lt;/groupId&gt;
    &lt;artifactId&gt;addon-duplication-detection-service-api&lt;/artifactId&gt;
    &lt;version&gt;[6.2.0,6.3.0)&lt;/version&gt;
    &lt;type&gt;jar&lt;/type&gt;
  &lt;/dependency&gt;
</pre>

You have to make three changes to the file `[dspace-source]/dspace/modules/jspui/pom.xml`. The xml file contains several sections, you will have to change the sections `<build><plugins>...</plugins></build>` and the section `<dependencies>...</dependencies>`. In the plugin section several plugins are defined and configured. The configuration of the maven-dependency-plugin contains the tags `<includeGroupIds>` and `<includeArtifactIds>`. You have to add `de.the-library-code.dspace` to the group ids and `addon-duplication-detection-service-api` to the artifact ids. The whole section should then look like the following:

```
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
      <execution>
        ...
        <configuration>
          <includeGroupIds>org.dspace.modules,de.the-library-code.dspace</includeGroupIds>
          <includeArtifactIds>additions,addon-duplication-detection-service-api</includeArtifactIds>
          ...
        </configuration>
      </execution>
    </executions>
  </plugin>
```

Then please change the configuration of the maven-war-plugin. It contains a section `<overlays>`, starting with an empty tag `<overlay />`, followed by another section `<overlay>...</overlay>`. You have to add another section between the empty tag and the existing `<overlay>`-section. Please add the following section:

<pre>
  &lt;overlay&gt;
    &lt;groupId&gt;de.the-library-code.dspace&lt;/groupId&gt;
    &lt;artifactId&gt;addon-duplication-detection-service-jspui&lt;/artifactId&gt;
    &lt;type&gt;war&lt;/type&gt;
  &lt;/overlay&gt;
</pre>

The whole `<overlays>`-section should then look like:

```
  <overlays>
    <overlay />
     <overlay>
       <groupId>de.the-library-code.dspace</groupId>
       <artifactId>addon-duplication-detection-service-jspui</artifactId>
       <type>war</type>
     </overlay>
     <overlay>
       <groupId>org.dspace</groupId>
       <artifactId>dspace-jspui</artifactId>
       <type>war</type>
     </overlay>
   </overlays>
```

At the end of the file `[dspace-source]/dspace/modules/jspui/pom.xml` you will find a section `<dependencies>...</dependencies>`. At the end of this section, right before the closing tag, you need to add the following lines:

<pre>
  &lt;dependency&gt;
      &lt;groupId&gt;de.the-library-code.dspace&lt;/groupId&gt;
      &lt;artifactId&gt;addon-duplication-detection-service-api&lt;/artifactId&gt;
      &lt;version&gt;[6.2.0,6.3.0)&lt;/version&gt;
      &lt;type&gt;jar&lt;/type&gt;
  &lt;/dependency&gt;
  &lt;dependency&gt;
      &lt;groupId&gt;de.the-library-code.dspace&lt;/groupId&gt;
      &lt;artifactId&gt;addon-duplication-detection-service-jspui&lt;/artifactId&gt;
      &lt;version&gt;[6.2.0,6.3.0)&lt;/version&gt;
      &lt;type&gt;war&lt;/type&gt;
  &lt;/dependency&gt;
</pre>

Add the content of the file [additional-Messages.properties](https://github.com/the-library-code/deduplication/blob/dspace-6.2-addition/additional-Messages.properties) to your message catalog. Your message catalog is located either under `[dspace-src]/dspace/modules/jspui/src/main/resources/Messages.properties` or `[dspace-src]/dspace-api/src/main/resources/Messages.properties`.

Then change your configuration as described below, re-compile and update DSpace, and restart Tomcat to finish the installtion.

### Installing the add-on by copying its code into your overlays

Please remind to install the PostgreSQL extension "fuzzystrmatch", as described above.

Copy the files within the directories [jspui/src](https://github.com/the-library-code/deduplication/tree/dspace-6.2-addition/jspui/src) and [api/src](https://github.com/the-library-code/deduplication/tree/dspace-6.2-addition/api/src) into your overlays (`[dspace-src]/dspace/modules/additions/src/...` and `[dspace-src]/dspace/modules/jspui/src/...`). Please pay attention not to overwrite any locally changed files. Add the content of the file [additional-Messages.properties](https://github.com/the-library-code/deduplication/blob/dspace-6.2-addition/additional-Messages.properties) to your message catalog. Your message catalog is located either under `[dspace-src]/dspace/modules/jspui/src/main/resources/Messages.properties` or `[dspace-src]/dspace-api/src/main/resources/Messages.properties`. Change your configuration as described in the following section of this readme, recompile and update DSpace, and restart Tomcat to finish the installation.

This add-on added the following files:

 * api/src/main/java/org/dspace/submit/step/DuplicateDetectionStep.java
 * api/src/main/java/org/dspace/content/duplication/DuplicateDetector.java
 * api/src/main/java/org/dspace/content/duplication/DuplicationDetectionServiceImpl.java
 * api/src/main/java/org/dspace/content/duplication/service/DuplicationDetectionService.java
 * api/src/main/java/org/dspace/content/duplication/factory/DuplicationDetectionServiceFactoryImpl.java
 * api/src/main/java/org/dspace/content/duplication/factory/DuplicationDetectionServiceFactory.java
 * jspui/src/main/webapp/submit/handle-duplicates.jsp
 * jspui/src/main/webapp/submit/request-title.jsp
 * jspui/src/main/java/org/dspace/app/webui/submit/step/JSPDuplicateDetectionStep.java
 * jspui/src/main/java/org/dspace/app/webui/jsptag/ReferenceItemTag.java

This add-on changed the following files:

 * api/src/main/java/org/dspace/content/dao/impl/ItemDAOImpl.java
 * api/src/main/java/org/dspace/content/dao/ItemDAO.java
 * api/src/main/java/org/dspace/storage/rdbms/hibernate/postgres/DSpacePostgreSQL82Dialect.java
 * jspui/src/main/webapp/WEB-INF/dspace-tags.tld
 * jspui/src/main/webapp/mydspace/perform-task.jsp
 * jspui/src/main/webapp/mydspace/preview-task.jsp
   
Then change your configuration as described below, re-compile and update DSpace, and restart Tomcat to finish the installtion.

## Configuration

### Submission Workflow

The Submission workflow needs some additional configuration. One part of the Duplication Detection Service is implemented as submission step for JSPUI. To activate this step add the following lines to your `dspace/config/item-submission.xml`:

```
<step-definitions>
[...]
     <!-- This configures the duplication detection step. Please notice that the duplication detection step
          currently works in JSPUI only and must be disabled if you use XMLUI. The duplication detection
          step is also referenced bellow. -->
     <step id="duplicate-detection">
         <heading>submit.progressbar.duplicate-detection</heading>
         <processing-class>org.dspace.submit.step.DuplicateDetectionStep</processing-class>
         <jspui-binding>org.dspace.app.webui.submit.step.JSPDuplicateDetectionStep</jspui-binding>
         <workflow-editable>true</workflow-editable>
     </step>
[...]
```

Reference the step as `<step id="duplicate-detection" />` in the step definitions, wherever you want to activate it. We suggest activating it before the `DescribeStep` as the duplication detection aims to prevent users from unnecessarily typing metadata of items that are already part of the repository. Following this advice it will probably be the first step or the step after the StartSubmissionLookupStep. See the [DSpace manual](https://wiki.duraspace.org/display/DSDOC6x/Submission+User+Interface) if you have further questions regarding the `item-submission.xml`.

As the Duplication Detection Step requires the user to enter the title the visibility of the title's metadata field should be set to workflow in `input-forms.xml`. This prevents the field from being shown twice in the submission process while it is still being editable in the workflow step. See the [DSpace manual](https://wiki.duraspace.org/display/DSDOC6x/Submission+User+Interface) if you need further guidance on how to achieve that.

### DSpace's configuration

The following configuration properties are used by the Duplication Detection Service. You need to add any of these properties only if you want to change the default value used for it as stated bellow. To set a property just add it to your local.cfg and change its value.

* *duplication.detection.edit-distance* (default: 8) Set the maximum edit distance between to titles to be considered similar. Regarding the edit distance, see the description of the Levenshtein algorithm above.
* *duplication.detection.maximum-duplicates* (default: 10) For performance reasons the maximum number of possible duplicates that are presented to a user should be limited. If you set this property to a value equal or less than 0 all possible duplicates will be presented. Be aware that disabling this limit may lead to timeouts if a lot of duplicates were found.
* *duplication.detection.field* (default: dc.title) Set the metadata field used for the comparison. Changing this property only changes the metadata field used to find possible duplicates. It does not change the field the Duplication Detection Submission Step stores the title value in.

## Architecture
The Duplication Detection Service was added to dspace-api as `org.dspace.content.duplication.service.DuplicationDetectionService`. It contains methods to do the fuzzy search, which let you define the metadata field and the maximum edit distance, as well as methods that use the configuration properties (see above) for the metadata field and the maximum edit distance. Furthermore the Duplication Detection Service provides methods that return all items found by the search as well as methods that return only the items a user can see. The methods that filter items filter all Workspace Items, all items that share a version History with the item they are compared to, all items that are withdrawn items and items that still in the workflow unless the user may administrate the item's collection or may review the item. All methods are preceded by JavaDoc comments that further explains there purpose.

To be able to use PostgreSQL's implementation of the Levenshtein algorithm, you need to install the fuzzystrmatch extension to your PostgreSQL installation (see above). As DSpace uses Hibernate Postgresql's function to use levenshtein have to be registered. This is done in `org.dspace.storage.rdbms.hibernate.postgres.DSpacePostgreSQL82Dialect`.

To reference the possible duplicates in the citation format described above, we added a JSPTag as `org.dspace.app.webui.jsptag.ItemReferenceTag`. This tag gets an item as attribute and creates the citation style reference as noted above: `<dspace:reference-item item="<%= Item %>" />`. It creates a button to open that item. If the item is withdrawn the button opens the edit item form, if the button is a Workflow Item the button opens the Workflow Task preview, and if the item is a normal item it opens the item view. The button does always open a new browser tab. You can use the `ItemReferenceTag` also for other items than possible duplicates.

To embed the duplication detection in the workflow process we copied `dspace-jspui/src/main/webapp/mydspace/perform-task.jsp` to the maven overlays and changed it. For the submission workflow `org.dspace.app.webui.submit.step.JSPDuplicateDetectionStep` were implemented. The Duplicate Detection Step uses two JSPs: `jspui/src/main/webapp/submit/request-title.jsp` and `jspui/src/main/webapp/submit/handle-duplicates.jsp`.

All changes were realized in the maven overlays, so that you can see easily which files were added or changed.

## License
This work is licensed under the [DSpace Source Code BSD License](http://www.dspace.org/license/).

## Further development
Further development of this extension is currently not planed. If you need help on changing or enhancing this extension in particular or DSpace in general, please don't hesitate to contact  [The Library Code GmbH](https://www.the-library-code.de).
