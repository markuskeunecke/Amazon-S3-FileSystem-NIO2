An **Amazon AWS S3** FileSystem Provider **JSR-203** for Java 8 (NIO2) using the AWS SDK v2.

This is a forked version of [Upplication's S3 FileSystem](https://github.com/upplication/Amazon-S3-FileSystem-NIO2) based on the AWS SDK v2.

Amazon Simple Storage Service provides a fully redundant data storage infrastructure for storing and retrieving any amount of data, at any time.
NIO2 is the new file management API, introduced in Java version 7.
This project provides a first API implementation, little optimized, but "complete" to manage files and folders directly on Amazon S3.

[![Build Status](https://travis-ci.org/elerch/Amazon-S3-FileSystem-NIO2.svg?branch=sdkv2)](https://travis-ci.org/elerch/Amazon-S3-FileSystem-NIO2/builds) [![Coverage Status](https://coveralls.io/repos/elerch/Amazon-S3-FileSystem-NIO2/badge.png?branch=sdkv2)](https://coveralls.io/r/elerch/Amazon-S3-FileSystem-NIO2?branch=sdkv2) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.lerch/s3fs/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.lerch/s3fs)

#### How to use

##### Download from Maven Central

```XML
<dependency>
	<groupId>org.lerch</groupId>
	<artifactId>s3fs-sdk2</artifactId>
	<version>1.0</version>
</dependency>
```


And add to your META-INF/services/java.nio.file.spi.FileSystemProvider (create if not exists yet) a new line like this: org.lerch.s3fs.S3FileSystemProvider.

##### S3FileSystem and AmazonS3 settings

All settings for S3FileSystem and for the underlying AmazonS3 connector library can be set through System properties or environment variables.
Possible settings can be found in org.lerch.s3fs.AmazonS3Factory.

#### Using service locator and system vars

Check that s3fs_access_key and s3fs_secret_key system vars are present with the correct values to have full access to your amazon s3 bucket.

Use this code to create the fileSystem and set to a concrete endpoint.

```java
FileSystems.newFileSystem("s3:///", new HashMap<String,Object>(), Thread.currentThread().getContextClassLoader());
```

##### Using service locator and amazon.properties in the classpath

Add to your resources folder the file amazon.properties with the content:
s3fs_access_key=access-key
s3fs_secret_key=secret-key

Use this code to create the fileSystem and set to a concrete endpoint.

```java
FileSystems.newFileSystem("s3:///", new HashMap<String,Object>(), Thread.currentThread().getContextClassLoader());
```

##### Using service locator and programatically authentication

Create a map with the authentication and use the fileSystem to create the fileSystem and set to a concrete endpoint.

```java
Map<String, ?> env = ImmutableMap.<String, Object> builder()
				.put(org.lerch.s3fs.AmazonS3Factory.ACCESS_KEY, "access key")
				.put(org.lerch.s3fs.AmazonS3Factory.SECRET_KEY, "secret key").build()
FileSystems.newFileSystem("s3:///", env, Thread.currentThread().getContextClassLoader());
```

Complete settings lists:

* s3fs_access_key
* s3fs_secret_key
* s3fs_request_metric_collector_class
* s3fs_connection_timeout
* s3fs_max_connections
* s3fs_max_retry_error
* s3fs_protocol
* s3fs_proxy_domain
* s3fs_proxy_host
* s3fs_proxy_password
* s3fs_proxy_port
* s3fs_proxy_username
* s3fs_proxy_workstation
* s3fs_socket_send_buffer_size_hint
* s3fs_socket_receive_buffer_size_hint
* s3fs_socket_timeout
* s3fs_user_agent
* s3fs_amazon_s3_factory
* s3fs_signer_override
* s3fs_path_style_access

##### Set endpoint to reduce data latency in your applications

```java
// Northern Virginia or Pacific Northwest
FileSystems.newFileSystem("s3://s3.amazonaws.com/", env, Thread.currentThread().getContextClassLoader());
// Northern Virginia only
FileSystems.newFileSystem("s3://s3-external-1.amazonaws.com/", env, Thread.currentThread().getContextClassLoader());
// US West (Oregon) Region
FileSystems.newFileSystem("s3://s3-us-west-2.amazonaws.com/", env, Thread.currentThread().getContextClassLoader());
// US West (Northern California) Region
FileSystems.newFileSystem("s3://s3-us-west-1.amazonaws.com/", env, Thread.currentThread().getContextClassLoader());
// EU (Ireland) Region
FileSystems.newFileSystem("s3://s3-eu-west-1.amazonaws.com/", env, Thread.currentThread().getContextClassLoader());
```

For a complete list of available regions look at: http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region

##### How to use in Apache MINA

```java
public FileSystemFactory createFileSystemFactory(String bucketName) throws IOException, URISyntaxException {
    FileSystem fileSystem = FileSystems.newFileSystem(new URI("s3:///"), env, Thread.currentThread().getContextClassLoader());
    String bucketPath = fileSystem.getPath("/" + bucketName);

    return new VirtualFileSystemFactory(bucketPath);
}
```

##### How to use in Spring

Add to classpath and configure:

```java
@Configuration
public class AwsConfig {

    @Value("${lerch.aws.accessKey}")
    private String accessKey;

    @Value("${lerch.aws.secretKey}")
    private String secretKey;

    @Bean
    public FileSystem s3FileSystem() throws IOException {
        Map<String, String> env = new HashMap<>();
        env.put(org.lerch.s3fs.AmazonS3Factory.ACCESS_KEY, accessKey);
        env.put(org.lerch.s3fs.AmazonS3Factory.SECRET_KEY, secretKey);

        return FileSystems.newFileSystem(URI.create("s3:///"), env, Thread.currentThread().getContextClassLoader());
    }
}
```

Now you can inject in any spring component:

```java
@Autowired
private FileSystem s3FileSystem;
```

#### Features:

* Copy and create folders and files
* Delete folders and files
* Copy paths between different providers
* Walk file tree
* Works with virtual s3 folders (not really exists and are element's subkeys)
* List buckets for the client
* Multi endpoint fileSystem

#### Roadmap:

* Add unit tests (current tests depend on a mock object based on v1)

#### Out of Roadmap:

* Watchers

#### How to contribute

Clone the github repository:

```java
git clone https://github.com/elerch/Amazon-S3-FileSystem-NIO2.git
```

To run the tests:

First, you must copy the file `src/test/resources/amazon-test-sample.properties` and paste in the same directory with the name amazon-test.properties. In your copy you must edit all the keys:

```
bucket_name=/your-bucket-name for test
# http://docs.aws.amazon.com/general/latest/gr/rande.html 
s3fs_secret_key= your secret key for test
s3fs_access_key=your access key for test
```

Thats all, now you can run the test with the command: `mvn test` or `mvn integration-test -Pintegration-tests`

#### LICENSE:

Amazon S3 FileSystem NIO2 is released under the MIT License.
