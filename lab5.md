# Lab 5 Report - Lian Welch

![5.1](images/5.1.png)
![5.2part1](images/5.2part1.png)
![5.2part2](images/5.2part2.png)
![5.3](images/5.3.png)
![5.4](images/5.4.png)
![5.5](images/5.5.png)
![5.6part1](images/5.6part1.png)
![5.6part2](images/5.6part2.png)
![5.7](images/5.7.png)
![5.8part1](images/5.8part1.png)
![5.8part2](images/5.8part2.png)
![5.9](images/5.9.png)
![5.10](images/5.10.png)
![5.11](images/5.11.png)
![5.12](images/5.12.png)
![5.13](images/5.13.png)
![5.14](images/5.14.png)
![5.15](images/5.15.png)
![5.16](images/5.16.png)
![5.17](images/5.17.png)
![5.18](images/5.18.png)
![5.19](images/5.19.png)
![5.20](images/5.20.png)
![5.21](images/5.21.png)
![5.22](images/5.22.png)

## Task 1
The commands executed in task one was done to set up the environment and start the lab. The cloud shell was opened and the PROJECT_ID environment variable was configured. The required GCP APIs were enabled; Cloud Run, Artifact Registry, Cloud Build, and Resource Manager. The ART_REG variable points were verified to the correct Artifact Registry repository. The lab 5 working directory was created and cloned to the lab repository by using a trailing dot. The python virtual environment was set up and activated using ~/lab5/venv. The gRPC tools were installed as pinned versions; grpcio, grpcio-tools, protobuf. The restore.sh script was created and ran to recover any environment variables in case the Cloud Shell reset. 

## Task 2
The commands executed in this task were done with the goal of generating gRPC Stubs. Proto/conversion.proto was inspected to show that it defines the Convert RPC method, ConversionRequest, and ConversionResult. Setup.sh was ran to compile the proto file using protoc into Python stubs. This was verified by showing that both conversion_pb2.py and conversion_pb2.grpc.py exist in the service directories. They both share the same generated code, meaning that both sides are bound since the engine uses its server side, and the API uses it as a client. This was confirmed when the Convert() method was shown to exist in conversion-engine/server.py. The UNIT_CATEGORY logic was confirmed in converter-api/server.py before any gRPC call was made. 

## Task 3
The commands in this task were done to build and deploy the conversion engine. The Dockerfile was reviewed and shown to define python's runtime, dependencies, and port 50051. The container image was built and pushed to Artifact Registry using gcloud build submit. Then --use-http2 and the private service --no-allow-unauthenticated was used to deploy to Cloud Run. The service URL was saved to the CONVERSION_ENGINE_URL variable and .env file. The service was verified as private when the curl command returned a 403 Forbidden response from Cloud Run, meaning that the container never received the request. 

## Task 4
The commands in this task were done to build and deploy the converter API. The Dockerfile was reviewed and shown to listen to port 8080 and runs server.py. The container image was built and pushed using gcloud builds to submit. Using public --allow-unauthenticated, it was deployed to Cloud Run and injected CONVERSION_ENGINE_URL as the environment variable. The only public facing endpoint was saved using CONVERTER_API_URL. 

## Task 5
The commands in this task were done to grant service to service permissions. The Converter API's default compute service account was retrieved using the project number. Roles/run.invoker was granted to the service account on the conversion-engine Cloud Run service. The IAM binding was verified using get-iam-policy. When calling the engine, the Converter API shows its identity token, where Cloud Run verifies the token before going through with the request so that the container does not manage any credentials. 

## Task 6
The commands in this task were done to create end to end testing. There was a health check to make sure that the status was ok. When running /units all five categories were shown; length, weight, temperature, speed, and volume. The length conversion returned the proper information showing that the internal gRPC call happened, while the temperature conversion showed that the non-linear conversion path was working. The weight, speed, and volume conversions were tested to show success. The cross category rejection returned HTTP/2 400 which means that validation had stopped the request before the engine was called. During the cold call the first call was supposed to take about 3-6 seconds starting from zero while the second call was under 1 second since it was already warm. 

## Task 7
The commands in this task were done to explore the Google Cloud Console. The converter-api showed allow unauthentication while the conversion-engine showed required authentication. The logs of converter-api shows HTTP requests and validation rejections, while the conversion-engine logs shows computation events, proving the single-responsibility design. The Artifact Registry showed the converter-api:v1 and conversion-engine:v1 images digest, size, and push date.

## Reflection Questions
### 1
Unit conversion is a genuinely stateless workload because the output depends on the input, meaning no database, session, or history is needed while the same input always creates the same output at any instance. The scale to zero model works because any instance can handle any request identically. A workload that would not be suitable for this model would be a shopping cart because it has to remember what is in the cart in between requests.

### 2
A public REST interface is defined by URL paths such as /convert?value=100&from=km&to=miles, the intended audience is people and the HTTP client. The internal gRPC interface is defined by conversion.proto, the intended audience is machines. It is useful for microservice systems to separate these two contracts because it lets each side evolve independently, the REST interface can adjust it's parameters without interacting with proto while the internal contract can add fields without affecting the external clients.

### 3
Validation is performed in the API layer instead of inside the engine because of the system boundary that protects the engine from receiving invalid input, meaning that the engine only does math without worrying about the HTTP error or category rules. This shows that the responsibilities should be divided between microservices by each service.

### 4
The Converter API proves to the Conversion Engine that it is allowed to call it by requesting a short lived identity token from the Cloud Run metadata server. The identity token represents who the caller is while specifying the audience, which is the Conversion Engine's URL. Cloud Run verifies the token at the platform edge before forwarding the request to the container. This model is considered zero trust because no services are trusted by default, meaning that every call must show its identity to get to its specific target.

### 5
A cold start is the delay when Cloud Run starts a container from zero after inactivity. This system may require two cold starts for the first request because Cloud Run starts the Converter API calling the Conversion Engine, which both start from zero meaning that the delays are sequential. Cloud Run accepts this trade off instead of keeping the containers always running because the benefit of paying nothing while idle outweighs the downside of an occasional latency experience. 

### 6
The .proto file is stored in the repository while the generated _pb2.py files are not because the .proto file is the source of truth and is what people write and maintain and the _pb2.py files are derived artifacts meaning that they are regenerated from the proto and shouldn't be saved as duplicates that could go out of sync. By storing only the proto both services regenerate stubs from the same definition but if the contract changes compile time errors show up right away before deployment.

### 7
A change belongs in the public API if it affects how the external clients interact. A change belongs in the internal engine if it is about math or computation logic. A change belongs in the service contract if the data exchanged between services needs to be changed. This decision process helps teams extend systems without breaking existing functionality by keeping changes isolated.

## Diagram
![Diagram](images/5labd.png)
