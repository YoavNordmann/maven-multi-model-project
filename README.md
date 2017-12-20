# maven-multi-model-project

or: "The Parent, The Project and the holy BOM"

> ***
> "In the beginning there was a pom. Then sub-modules 
>  came and defined it as their parent and all was good.
>  Then dependencies came along and all hell broke loose..."
> ***

In recent years I am seeing more and more maven multi-module projects with a separation of the encompassing project and the definition of the parent project. The encompassing project would define `<modules>` while the parent project would define the `<dependencyManagement>` and the `<pluginManagement>` and `<plugins>`.

This setup makes a lot of sense. There is true seperation of concern here. The encompassing project is responsible for governing the build process of the product while the parent project is responsible for the build plan of each individual project and which libraries it includes.

## BOM - Bill of Materials
Although I was familiar  with the concept of *BOM* in maven, I did not really realize the power it holds. For this understanding I have to thank the spring boot libraries and setup.
A BOM ("Bill of Materials") is simply a pom.xml file that contains a list of dependencies and their corresponding versions declared under `<dependencyManagement>`. In essence, it holds the *technology stack* of the project. Adding this seperate pom.xml file to the above mentioned *Multi Module Project Setup* we get an even better separation of concern by removing the dual responsibility of the parent project. We separate the technology stack from the build plan. 

### Maven Project Setup
So the Project setup would look like this:

1. Project
2. Project BOM
3. Project Parent

And the project structure would look something like this:

```
project
	├── ci
	├── cd
	├── pom.xml
	├── project-A
	│   └── pom.xml
	├── project-B
	│   └── pom.xml
	├── project-bom
	│   └── pom.xml
	└── project-parent
	    └── pom.xml
```

#### Project
+ This project has a pom.xml where we define the `<modules>`
+ In this project folder we will also have all the sub modules of this maven project.
+ Here one would add CI and CD folders for saving files of that nature
+ The Project BOM will be defined in here as a sub-module
+ The Project BOM will be defined in here as a sub-module
+ The Project Parent could be defined in here as a sub-module but can also be defined as a global parent somewhere else.

#### Project BOM
+ This project has a pom.xml where we define the `<dependencyManagement>` and all dependencies of the project.

#### Project Parent
+ This project has a pom.xml where we define the `<pluginManagement>` and `<plugins>`. 
+ Furthermore, we will define here the import of the BOM project as follows:

```xml
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>com.tikal</groupId>
				<artifactId>project-bom</artifactId>
				<version>${project-bom.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>		
	</dependencyManagement>
```

### project-A and project-B
+ These projects will generate actual artifacts (jar, war)
+ We define the parent in the pom.xml as follows:

```xml
	<parent>
		<groupId>com.tikal</groupId>
		<artifactId>project-parent</artifactId>
		<version>current-SNAPSHOT</version>
		<relativePath>../project-parent/pom.xml</relativePath>
	</parent>
```
+ In case the parent project is a global one we will define it without `<relativePath>`

The files can be viewed in their entirety in [github](https://github.com/YoavNordmann/maven-multi-model-project)

## Benefits

### Multiple Different Build Possibilities

One can change which modules are being built without changing anything else withing the project either by way of profiles or by way of another sub-module. 
Furthermore, one can change the way each individual module is being built as well. A lot of companies define one parent project for all projects to controll the build process. The build process **must** be the same for all projects as it holds plugins with tools which run to ensure code quality.
### Maven Dependency Hell Resolution

Too many times we struggle to get the Maven Dependency Hell uner controll. Transitive dependency conflicts are mediated by what is called the "nearest definition" which means that it will use the version of the closest dependency to your project. In case you are not satisfied with maven's decision you can change it by including the dependency in your pom which is not necessarily a good approach. This list can suddenly grow quite big.
Having a BOM define all the dependecies handles this for all of your projects avoiding duplication of dependancy duplication. A very good explanation on this issue can be seen at [Maven – Bill Of Materials](http://blog.inflinx.com/2013/12/29/maven-bill-of-materials/)

### Technology Stack Conformity
This BOM can be used by different parents and different projects just the same. In such a way we ensure the technology of all projects in a company are on the same level.

## The One Issue that remains
The only issue which remains is the build plan. As of now there is no way to import the `<pluginManagement>` and `<plugins>` as we do for the `<dependencyManagement>`. This is why when using [spring boot](https://projects.spring.io/spring-boot/) one has to define `spring-boot-starter-parent` as parent of he project otherwise you will not benefit from the plugin management of spring boot as described in their [documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html#using-boot-maven-parent-pom).

I do believe that if maven would make this adjustement then it would usher in a new era of configurability and complexity.

## Conclusion
If you are facing a maven multi-module project with not just two but several sub-modules I suggest you try this model as it helped me in the long run manageing all of the projects and the whole build process.

Contact Details
[Github](https://github.com/YoavNordmann)
[LinkedIn](https://www.linkedin.com/in/yoavnordmann/)
[Twitter](https://twitter.com/YoavNordmann)

