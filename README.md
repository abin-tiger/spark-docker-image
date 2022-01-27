# Spark Docker Images

The Dockerfiles in this repository are based on the ones from [Spark source repository](https://github.com/apache/spark/tree/master/resource-managers/kubernetes/docker/src/main/dockerfiles/spark). It was refactored to make it flexible.

The images built are stored in the [GitHub Packages docker repository](https://github.com/users/abin-tiger/packages/container/package/spark-docker-image). Available tags: 3.1.2, 3.2.0.

The image contains:
1. Spark distribution
2. Python 3.9
3. Pyspark library.
4. Dependancies for AWS S3.


## Using the images

The images can be executed directly with docker, [Spark submit](https://spark.apache.org/docs/latest/running-on-kubernetes.html#cluster-mode) or [Kubernetes Spark operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator).

Executing with docker:

```bash
docker run --env SPARK_USER=spark ghcr.io/abin-tiger/spark-docker-image:3.1.2 spark-submit --class org.apache.spark.examples.SparkPi local:///opt/spark/examples/jars/spark-examples_2.12-3.1.2.jar
```

## More details

The image build is divided into two stages: 
1. Download - The downloader stage is just used to download and extract any files required. The circle ci image used in here has all the necessary libs pre-installed.. This stage downloads the spark package from [Downloads | Apache Spark](https://spark.apache.org/downloads.html).
2. Build - This stage builds the final image from the downloaded artifacts.


## Using the image to build for a new Spark version

1. Duplicate and rename any of the Dockerfile (for eg. Dockerfile.spark-3.2.0) to the format Dockerfile.spark-<version>. For e.g. for building a image for Spark v2.3 create a new Dockerfile name Dockerfile.spark-2.4.1.
2. Update the docker build arguments:
    1. ARG SPARK_VERSION: This should be the version number for Spark. This is later set as an environment variable in final image.
    2. ARG PACKAGE_NAME: This is the name of the spark distribution file. Usually in the format spark-<spark version>-bin-hadoop<hadoop version>.
    3. ARG SPARK_DIST_URL: URL to the Spark tar.gz file. It can be found in [Downloads | Apache Spark](https://spark.apache.org/downloads.html). The link seen here is usually mirror link which could expire after some time. For older images find the link from Spark archives: https://archive.apache.org/dist/spark/
3. Update the jar versions under the comment "# All the extra jars needed". Here hadoop-aws version should match the hadoop version bundled in Spark. And the aws-bundle-sdk version should match the version used as compile dependancy in hadoop-aws. Checkout [maven](https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-aws) to get the correct versions.
4. Update the `py4j` zip path under "# Add # py4j in PYTHONPATH". py4j is a dependancy for Pyspark which should be added to PYTHONPATH. Version of the py4j included in each Spark version is different. Find the name of the zip file from the `spark-config.sh` file in Spark repository. For e.g for Spark version 3.2.0 spark-config.sh is [here](https://github.com/abin-tiger/spark/blob/v3.2.0/sbin/spark-config.sh). Switch the GitHub tag to see the file for another Spark version.
