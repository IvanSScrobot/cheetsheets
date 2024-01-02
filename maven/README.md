## mvn package
builds the Maven project and packages it into a JAR

```
mvn -f dir/pom.xml package -o -q
```
-o is to build in offline mode
-q is for quiet mode
-X is to run in debug mode

To skip unit tests: 
```
mvn -DskipTests package
```

## mvn compiler:compile
compiles the Java source classes of the Maven project

To run the entire Maven lifecyclet up to compile, use:
```mvn compile```

## mvn install
builds the Maven project and installs the project files (e.g. JAR files) to the local repository

## mvn deploy
deploys the artifact to the remote repository

## mvn validate
validates the Maven project to ensure that everything is correc

## mvn dependency:tree
to generate the dependency tree

## mvn test
to run the project's tests

## mvn clean
to clean the Maven project by deleting the target directory