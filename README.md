# Flair

Let's call it a framework.

Please note, flair is very new and not battle tested, it should not be a problem for most, as the libraries that flair uses are very stable, and you will easily be able to convert a flair project into it's contituent parts.

## What is flair?

It's a series of codegen and convience wrappers to write web applications in rust. At this point it only does GRPC, but the project is planned and structured to support other schema'd and schemaless protocols. I'd like to add some form of JSON, websockets and avro in soon.

## Why is flair?

Flair wanted to solve the following specific problems:

- Have a CLI that get's you up and running with a rust microservice instantly.
- Schema based protocols allow you to codegen your request handlers, so that you don't have to write "controllers" that implement your service's trait, which you already did in your proto files. Just write your proto file, run a command and start writing your code.
- Transport should not be intertwined with application logic, except when the application is for transport. A flair project tries to get you to write your web interface layer separate from your application logic. From there we aim to support multiple ways to communicate (sockets, streams, queues, etc.). There is a place where you handle your transport, and there's a place where you handle your application logic. It's not always trivial/possible to switch from synchronous to async transport, but you shouldn't be blocked by your framework.
- Rust compile times are notorious, and not every team has the expertise/time/option to improve their crates' complition time. The last resort usually ends up being crate splitting. So what if we just already seperated our applications boundries with crates? Flair makes it easier to split your application into different crates from the get-go, by providing a structure that makes sense. This is a opt in feature, and you are welcome to ignore it if you feel that you don't need it, or embrace it when the time comes.


## How does it work?

At this stage, flair has two main points:

### Codegen:
Most of the macro's are really just codegen for a series of dependencies

- Tokio: provides a fantastic async runtime
- Tonic: provides all the heavy lifting for handling GRPC.
- Tower: provides some necessary abstractions to facilitate transport independent middleware.
- Clap: provides a CLI interface for your web binary so that you can pass in args.
- Serde: Config file parsing, because you'll probably need to pass in some runtime secrets/config.

### Task locals
In your application, you'll probably want to have access to some global resources:

- A database connection pool
- Tracing headers like correlation ID's
- External services with their own connection pools, or single clients for rate limiting, etc.

You might have a few of these and feel comfortable passing them in explicitly, but this can quickly become messy. At the cost of losing some explicitness, flair introduces wrappers to create and wrap your tokio taks with a flair context variable. This is simply a struct that holds a series of Arc's to your global instances and/or hold your request headers so that you can access them multiple layers deep, without having functions that have nothing to do with requests or database calls pass around variables just so that you can inject it in subsequent calls.

Other frameworks have built their own version of this concept, but what nice about flair's is that it's completely Tokio. Which means this mechanism isn't specific to the JSON or GRPC library that you use. You can reuse this with any other library, provided it can run on Tokio.

While this is really convienient, there is a cost associated with task switching, and task locals need to be moved around when you suspend your task. Holding onto a few references won't impact your applications performance significantly. Be aware that if you choose to crack this open for your own usecase, that you ideally don't want to store too much in here.
