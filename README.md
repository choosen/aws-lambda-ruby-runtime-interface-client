## AWS Lambda Ruby Runtime Interface Client

We have open-sourced a set of software packages, Runtime Interface Clients (RIC), that implement the Lambda
 [Runtime API](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html), allowing you to seamlessly extend your preferred
  base images to be Lambda compatible.
The Lambda Runtime Interface Client is a lightweight interface that allows your runtime to receive requests from and send requests to the Lambda service.

The Lambda Ruby Runtime Interface Client is vended through [rubygems](https://rubygems.org/gems/aws_lambda_ric). 
You can include this package in your preferred base image to make that base image Lambda compatible.

## Requirements
The Ruby Runtime Interface Client package currently supports Ruby versions:
 - 2.5.x up to and including 2.7.x
 
## Usage

### Creating a Docker Image for Lambda with the Runtime Interface Client
First step is to choose the base image to be used. The supported Linux OS distributions are:

 - Amazon Linux 2
 - Alpine
 - CentOS
 - Debian
 - Ubuntu

In order to install the Runtime Interface Client, either add this line to your application's Gemfile:

```ruby
gem 'aws_lambda_ric'
```

And then execute:

    $ bundle

Or install it manually as:

    $ gem install aws_lambda_ric


Next step would be to copy your Lambda function code into the image's working directory.
```dockerfile
# Copy function code
RUN mkdir -p ${FUNCTION_DIR}
COPY app.rb ${FUNCTION_DIR}

WORKDIR ${FUNCTION_DIR}
```

The next step would be to set the `ENTRYPOINT` property of the Docker image to invoke the Runtime Interface Client and then set the `CMD` argument to specify the desired handler.

Example Dockerfile:
```dockerfile
FROM amazonlinux:latest

# Define custom function directory
ARG FUNCTION_DIR="/function"

# Install ruby
RUN amazon-linux-extras install -y ruby2.6

# Install the Runtime Interface Client
RUN gem install aws_lambda_ric

# Copy function code
RUN mkdir -p ${FUNCTION_DIR}
COPY app.rb ${FUNCTION_DIR}

WORKDIR ${FUNCTION_DIR}

ENTRYPOINT ["aws_lambda_ric"]
CMD ["app.App::Handler.process"]
```

Example Ruby handler `app.rb`:
```ruby
module App
  class Handler
    def self.process(event:, context:)
      "Hello World!"
    end
  end
end
```

### Local Testing

To make it easy to locally test Lambda functions packaged as container images we open-sourced a lightweight web-server, Lambda Runtime Interface Emulator (RIE), which allows your function packaged as a container image to accept HTTP requests. You can install the [AWS Lambda Runtime Interface Emulator](https://github.com/aws/aws-lambda-runtime-interface-emulator) on your local machine to test your function. Then when you run the image function, you set the entrypoint to be the emulator. 

*To install the emulator and test your Lambda function*

1) From your project directory, run the following command to download the RIE from GitHub and install it on your local machine. 

```shell script
mkdir -p ~/.aws-lambda-rie && \
    curl -Lo ~/.aws-lambda-rie/aws-lambda-rie https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie && \
    chmod +x ~/.aws-lambda-rie/aws-lambda-rie
```
2) Run your Lambda image function using the docker run command. 

```shell script
docker run -d -v ~/.aws-lambda-rie:/aws-lambda -p 9000:8080 \
    --entrypoint /aws-lambda/aws-lambda-rie \
    myfunction:latest \
        aws_lambda_ric app.App::Handler.process
```

This runs the image as a container and starts up an endpoint locally at `http://localhost:9000/2015-03-31/functions/function/invocations`. 

3) Post an event to the following endpoint using a curl command: 

```shell script
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'.
```

This command invokes the function running in the container image and returns a response.

*Alternately, you can also include RIE as a part of your base image. See the AWS documentation on how to [Build RIE into your base image](https://docs.aws.amazon.com/lambda/latest/dg/images-test.html#images-test-alternative).*

## Development

### Building the package
Clone this repository and run:

```shell script
make init
make build
```

### Running tests

Make sure the project is built:
```shell script
make init build
```
Then,
* to run unit tests: `make test`
* to run integration tests: `make test-integ`
* to run smoke tests: `make test-smoke`

## Security

If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security via our [vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public github issue.

## License

This project is licensed under the Apache-2.0 License.