#### 引入spring-boot
````
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.0.RELEASE</version>
		<relativePath />
	</parent>
````

#### 引入依赖，spring私有库
````
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.M8</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
			<!-- <artifactId>spring-cloud-netflix-eureka-server</artifactId> -->
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
	</dependencies>
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/libs-milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
````

#### 启动类
````
	@SpringBootApplication
	@EnableEurekaServer
	@ComponentScan
	@ImportAutoConfiguration
	public class Application {

		public static void main(String[] args) {
			SpringApplication.run(Application.class, args);
		}

		@Bean
		public Filter httpTraceFilter() {
			return new CommonsRequestLoggingFilter();
		}

	}
````

#### 配置
````
server:
  port: 8761
spring:
  application:
    name: eureka-server
eureka:
  server:
    enableSelfPreservation: true
  instance:
    hostname: peer1
    leaseRenewalIntervalInSeconds: 1
    leaseExpirationDurationInSeconds: 2
  client:
    register-with-eureka: false
    fetch-registry: false
    registry-fetch-interval-seconds: 30
    serviceUrl:
      defaultZone: http://192.168.2.1:8761/eureka/
	  
````

#### assembly打包
````
<build>
		<plugins>
			<!-- <plugin> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-maven-plugin</artifactId> 
				<executions> <execution> <goals> <goal>repackage</goal> </goals> </execution> 
				</executions> </plugin> -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<version>2.10</version>
				<executions>
					<execution>
						<id>copy-dependencies</id>
						<phase>package</phase>
						<goals>
							<goal>copy-dependencies</goal>
						</goals>
						<configuration>
							<type>jar</type>
							<includeScope>runtime</includeScope>
							<useBaseVersion>true</useBaseVersion>
							<outputDirectory>${project.build.directory}/lib</outputDirectory>
						</configuration>
					</execution>
				</executions>
			</plugin>
			<!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<classesDirectory>target/classes/</classesDirectory>
					<archive>
						<manifest>
							<mainClass>${main-class}</mainClass>
							<!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
							<useUniqueVersions>false</useUniqueVersions>
							<addClasspath>true</addClasspath>
							<classpathPrefix>lib/</classpathPrefix>
						</manifest>
						<manifestEntries>
							<Class-Path>.</Class-Path>
						</manifestEntries>
					</archive>
				</configuration>
			</plugin>

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<descriptors> <!--描述文件路径 -->
						<descriptor>src/assembly/zip.xml</descriptor>
					</descriptors>
				</configuration>
				<executions>  <!--执行器 mvn assembly:assembly -->
					<execution>
						<id>make-tar-x</id><!--名字任意 -->
						<phase>package</phase><!-- 绑定到package生命周期阶段上 -->
						<goals>
							<goal>single</goal><!-- 只运行一次 -->
						</goals>
					</execution>
				</executions>
			</plugin>

		</plugins>
	</build>
````

#### zip.xml
````
<assembly
        xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
    <id>bin</id>
    <formats>
        <format>tar.gz</format>
    </formats>
    <fileSets>
        <fileSet>
            <directory>${project.basedir}/target</directory>
            <includes>
                <include>*.jar</include>
            </includes>
            <outputDirectory>/</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.basedir}/target/lib</directory>
            <includes>
                <include>*.jar</include>
            </includes>
            <outputDirectory>/lib</outputDirectory>
        </fileSet>
    </fileSets>

</assembly>
````

#### mvn
````
	mvn spring-boot:run
	mvn assembly:assembly
	java -jar x.jar --spring.profiles.active=peer1
````

[例子:](https://github.com/CheukBinLi/original/tree/2.20/original-prototype/original-prototype.spring.cloud/original-prototype.spring.cloud.eureka-server)
	  