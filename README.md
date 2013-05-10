# Restdoc Maven Plugin

This plugin aggregates and transforms XML metadata gleaned from annotation processing with [org.rhq.helper:rest-docs-generator](https://github.com/rhq-project/helpers), which uses annotations from the JAX-RS spec and from the [swagger](https://developers.helloreverb.com/swagger/) project to generate documentation about REST endpoints and datatypes. By default, it generates HTML output using a built-in stylesheet that was originally from [here](https://git.fedorahosted.org/cgit/rhq/rhq.git/tree/modules/enterprise/server/jar/src/main/xsl/apiout2html.xsl).

Currently, this is **A VERY RUDIMENTARY IMPLEMENTATION**. The best I can currently say about the generated HTML is that it's better than nothing. Marginally. However, the aggregated XML offers more promise, especially when coupled with a jQuery-driven UI.

## Build.

Building is simple: `mvn clean install`

## Use.

In your project's top-level POM, add this:

    <dependencies>
      <dependency>
        <groupId>com.wordnik</groupId>
        <artifactId>swagger-annotations_2.9.1</artifactId>
        <version>1.2.3</version>
        <scope>provided</scope>
      </dependency>
      <dependency>
        <groupId>org.rhq.helpers</groupId>
        <artifactId>rest-docs-generator</artifactId>
        <version>4.6.0</version>
        <scope>provided</scope>
      </dependency>
    </dependencies>
    [...]
    <build>
      <pluginManagement>
        <plugins>
          <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <executions>
              <execution>
                <id>generate-restdoc</id>
                <phase>generate-resources</phase>
                <goals>
                  <goal>compile</goal>
                </goals>
                <configuration>
                  <annotationProcessors>
                    <processor>org.rhq.helpers.rest_docs_generator.ClassLevelProcessor</processor>
                  </annotationProcessors>
                  <proc>only</proc>
                  <compilerArguments>
                    <AtargetDirectory>${project.build.directory}/classes/META-INF/rest-docs</AtargetDirectory>
                    <AmodelPkg>org.commonjava.aprox.model</AmodelPkg>
                    <!-- <AskipPkg>org.rhq.enterprise.server.rest.reporting</AskipPkg> -->
                    <!-- enable the next line to have the output of the processor shown on console -->
                    <Averbose>true</Averbose>
                  </compilerArguments>
                  <!-- set the next to true to enable verbose output of the compiler plugin; default from the evn is true, but this is too noisy -->
                  <verbose>false</verbose>
                </configuration>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <groupId>org.commonjava.maven</groupId>
            <artifactId>restdoc-maven-plugin</artifactId>
            <version>0.1-SNAPSHOT</version>
            <executions>
              <execution>
                <id>generate-html</id>
                <goals>
                  <goal>generate</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </pluginManagement>
    </build>


This will automatically attempt to process any classes in your project tree that have JAX-RS or Swagger annotations. Next, in your WAR module, add:

    <build>
      <pluginManagement>
        <plugins>
          <plugin>
            <artifactId>maven-war-plugin</artifactId>
            <configuration>
              <webResources>
                <webResource>
                  <directory>${project.build.directory}/restdocs</directory>
                </webResource>
              </webResources>
            </configuration>
          </plugin>
        </plugins>
      </pluginManagement>
      <plugins>
        <plugin>
          <groupId>org.commonjava.maven</groupId>
          <artifactId>restdoc-maven-plugin</artifactId>
        </plugin>
      </plugins>
    </build>

When your build runs, your WAR should contain two new documents:

    /restdocs.html
    /restdocs.xml

The first is aggregated and transformed using the HTML XSLT stylesheet. The second is simply aggregated output from the annotation processor, which may be used from javascript or other UIs.

## Other Options.

Check `mvn org.commonjava.maven:restdoc-maven-plugin:0.1-SNAPSHOT:help` for more usage information.

