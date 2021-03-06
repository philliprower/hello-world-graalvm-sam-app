# Hello World Example AWS Serverless Application Model Project which uses GraalVM

Goal of this project is to demo using GraalVM in AWS SAM using both GraalVM JVM and as a native executable.
Hopefully help others use this as a template on how one can collect the profile data needed to build a native
executable using GraalVM native image agent.

[GraalVM](https://www.graalvm.org/docs/getting-started/) is a JVM which can run JVM applications with GraalVM JVM or compile JVM byte codes down to a [native executable](https://www.graalvm.org/reference-manual/native-image/).

This SAM application deploys Lambda same small hello-world application using GraalVM JVM and native executable.
To compile the JVM byte codes down to native executables, GraalVM must know about all the Reflection in your app.
GraalVM contains an agentlib which can gather this data for you.

Two docker images are built when running `sam build`.  First using GraalVM and second using a native executable.
The `Dockerfile-native` build an image that use GraalVM `native-image` command with executable jar which has agent profile data in `META-INF/native-image` to build a native executable.  You'll need a `native-image.properties` file if you are creating your
own app from this demo.  You can copy one included here in `src/main/resources/META-INF/native-image`.  Further information
on `native-image.properties` see [GraalVM Build Configuration](https://www.graalvm.org/reference-manual/native-image/BuildConfiguration/).

As is this repo can be used with `sam build` and then `sam deploy --guided`.  You will also need an AWS ECR repository.

The required profile data was gathered by running the `Dockerfile-graalvm` locally with the AWS Lambda Runtime Interface Emulator.
AWS Lambda Runtime Interface Emulator could be package in the docker image but instead I've just installed it, locally 
so it can be used to test other AWS Lambdas locally and this emulator isn't package in the image used by AWS Lambda.

### Install AWS Runtime Interface Emulator On Mac:
```shell
mkdir -p ~/.aws-lambda-rie && curl -Lo ~/.aws-lambda-rie/aws-lambda-rie https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie && chmod +x ~/.aws-lambda-rie/aws-lambda-rie
```

## Test Using Local Lambda Emulator

```shell
cd HelloWorldFunction
```

```shell
docker-compose up -d --build graalvm
```

```shell
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```

This will generate the required profile json files in `./target/native-image`.
Copy those files to your `./src/main/resources/META-INF/native-image`.

## Shutdown

```shell
docker-compose down
```

## Startup Native Image locally

```shell
docker-compose up -d --build native
```

Test again:

```shell
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'
```

## Deploy to AWS 
Run these from top directory of this repo.

```shell
sam build
sam deploy --guided
```

Output should display the HTTP endpoints.

If you don't have an ECR repository then you can run this command:
```shell
aws ecr create-repository --repository-name <your name>
```

You'll need an ECR repository for the `sam depoy --guided` command.

## [Emulation Configuration](https://github.com/aws/aws-lambda-runtime-interface-emulator#how-to-configure)

Running locally with AWS Lambda Runtime Interface Emulator is done via environment variables.
You can provide your SAM app with credentials needed to call AWS services such as S3 and SQS.

### Notes on using this project as a template
You'd want to execute all paths in your code to ensure all the profile data required would be generated.

### Intellij Users
The `hello-world.http` and `http-client.env.json` can be used with Intellij to make REST http calls.
After deploying to AWS using `sam deploy --guided` copy the API Gateway Id output into http-client.env.json.

# References
* https://github.com/aws/aws-lambda-runtime-interface-emulator
* https://github.com/aws/aws-lambda-java-libs/tree/master/aws-lambda-java-runtime-interface-client
* https://www.graalvm.org/reference-manual/native-image/