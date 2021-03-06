
# Introduction 

## Who is this article for?

In this article, I will show you how I built **a full stack architecture using PHP, Symfony, Doctrine, GraphQL, ApolloJS and React**. If you are considering to set up an architecture with GraphQL in PHP, I encourage you to read this article because I talk about each component and interaction between them. 

I also take some time to show how I solved the most pressing challenges I found while integrating these technologies. 

The code we are going to explore is an open source App built with this architecture. When I was studying, I felt that there were missing code examples on the community, so **I hope that the repositories we are using will be usefull to your own study**.

Here they are: 

- [PHP backend with Symfony, Doctrine and Overblog GraphQL](https://gitlab.com/bruno.p.reis/nosso-jardim)

- [JS frontend with Apollo Client, React Apollo, React and Semantic UI React](https://gitlab.com/bruno.p.reis/nosso-jardim-client)

As they say, **a repo is worth a thousand words**!

Don't take the example here as "Best Practices". It's impossible to talk about "Best Practices" in a so fast evolving technology scenario, especially for GraphQL based libs, since it was released two year ago. 

So please do as you were eating a fish, keep the meat and spit the fishbone.  

## Driving Forces

Please, allow me a more abstract and high level talk before getting into the code. 

React is a very nice frontend technology and has some paradigm shifts for a frontend tech. **It's beauty lies in the fact that you always have a predictable view for a specific model state.** The hadaches with a lot of binds and listeneres are mostly gone and the code is a lot more clean and therefore maintainable. 

Talking in other terms, a React frontend is a pure function where you pass a model and get a view. Together with Redux, that has a similar approach for state management, they make a very cool tool for the frontend apps, but something was missing.  

**The R & R couple (React and Redux) still lack a good way to integrate with assynchronous data**. So, I started a quest on what to use there. I researched a lot of redux patterns and libs to do this job, but they all seemed to require to much to do the job. 

Looking for solutions, I was hearing a lot of buzz about Relay, and I decided to give it a try. Specially because I like the facebook policy of releasing code they actually use. So I started studying relay.  

Relay docs were too cryptic to me and I gave up on it on my first try. A little later, I met Apollo, backed by the meteor guys. I found they had a better documentation and community, so I decided to give it a try.

I started my first app using a GraphQL client. Using it **I discovered that Apollo was able to do a lot more than just fetching data from the server. It's cache layer is very well managed, it is able to normalize and denormalize data into there, it can manage subscriptions and also can agregate queries to save requests.**  

Using it I was able to mantain a very nice and organised codebase, backed by a very powerfull tool to integrate assynchronous data with my view. **To me, the declarative way of composing the views with HOCs wrapping the views fitted like a glove in my hands.**

To use Apollo, I had to learn **GraphQL, that is a superb language and technology to expose and query an API**. Created inside Facebook and used into real life hard core projects in an intensive way before being released, GraphQL really improves a lot of things over old rest communication. ( Don't take me for granted. [Take a look here.](http://graphql.org/learn/)

I'm already a **PHP/Symfony** developer for a long time, so that was my choice for backend. I confess I was very inclined to work with the JS implementation at first, but adding too much new stuff to a new project is never good in my experience. 

I found mature libs in PHP and the app organization, after a lot of refining, came out to be very good and clean in the backend too. Indeed, **GraphQL helped me a lot on that Backend organisation.** 

Another decision taken was running most tests over the API layer, what also proved to be a very good decision. I like a lot **TDD**, but finding the correct amount of tests to move fast and also finding the correct layer to test is a fine art to me. 

Well, this stuff is hard to talk about, so please take a look at the code([1](https://gitlab.com/bruno.p.reis/nosso-jardim)[2](https://gitlab.com/bruno.p.reis/nosso-jardim-client)) to get an overall feeling befor we start. Or, if you are not in a mood, just follow along.

## Our Roadmap

    Introduction
        Who is this article for?
        Driving Forces
        Our Roadmap (<-you are here)
    Backend
        The App we are gonna look at
        Installation
        The API Schema
        Overblog GraphQL
        The Schema Declaration - The entrance to the backend
        Following a complete backend query
        A little Recap
        Cors
        Error Handling
        Testing your code
    Frontend
        Installing the frontend
        Overview
        Our next destination: MessageStatus View Component
        Integrating data with containers
        Diving deepen into the PEERS query
        Integration between Apollo containers and also Redux
        Verification Point


# Backend

On the backend side, we are going to explore a [PHP backend with Symfony, Doctrine and Overblog GraphQL](https://gitlab.com/bruno.p.reis/nosso-jardim) 

## The App we are gonna look at.

The view is usually the easiest part to understand an application domain. So let's do it before we install the backend and look at it's API. 

We are gonna work on a **Knowledge Management App**. The main purpose of this app is to organize knowledge that is shared through Telegram and other channels in the future. 

The mais screen, as shown below, is where you organize messages in specific threads or conversations (Conversas in Brazilian Portuguese) and put tags on those threads. 

![Create a Thread and Tag it!](./images/createThread.gif)

This App gives us interesting data structures to serve us as an example:

1. A paginated list - Messages
2. A non paginated list - Threads
3. A tree - Subtopics (not shown in the gif)
4. Lots of simple views, menus and buttons.
5. Forms and data edition - Add Tag, Edit Subtopic, Add Subtopic, and so on...

**These examples are all available on the repository code** and can ilustrate how to handle these data structures on both front and backend. 

Ok, now that you understand what we will be working on, you might want to grab the backend code.

## Installation

1. [Clone the backend repo.](https://gitlab.com/bruno.p.reis/nosso-jardim)
2. Run composer install and configure your parameters
3. Import Fixtures and Data
4. Start the php server
5. Read the next section so that you can look at GraphiQL knowing the app


## The API Schema

You can run [queries](https://dev-blog.apollodata.com/the-anatomy-of-a-graphql-query-6dffa9e9e747) using GraphiQL and see the results right there, with the docs on the right where you can check for the types and formats. 

![Running Queries](./images/runningQueries.gif)
![Looking at the Documentation](./images/documentation.gif)

Go ahead, take some time now, navigate through GraphiQL and look at the schema and queries available to understand our domain. I should be under /graphiql in the address your php server is runnig. Probably on 127.0.0.1:8000/graphiql

## Overblog GraphQL

To build the GraphQL server over symfony, we are using the overblog/GraphQLBundle lib. (https://github.com/overblog/GraphQLBundle) 

It's a very good lib that integrate straight into symfony creating the GraphQL endpoint and adding nice features to the Webonys lib. One very special feature it has is the [Expression Language|https://github.com/overblog/GraphQLBundle/blob/master/Resources/doc/definitions/expression-language.md]

As you can see on it's [requirements|https://github.com/overblog/GraphQLBundle/blob/master/composer.json], it is built over cool libs. I want to especially note these two: 

	1 - overblog/graphql-php-generator - This is responsible for reading a nice and clean yml and convert it into the GraphQL type objects. It will all happen under the hood and all you will need to touch is the yml and the resolvers. 

	2 - webonyx/graphql-php - This is the real engine of our car. A PHP port of GraphQL reference implementation. Very stable and ready to use. Please take a look at the docs and I call special attention to the 

## The Schema Declaration - The entrance to the backend

Our schema declaration is under src/AppBundle/Resources/config/graphql/
Queries are defined in the Query.types.yml. The fields there are the system queries, possible args are defined there and the resolvers are also set there. 

So we can understand that this file is the entrance on our system. It defines the API interface with the outter word. 

## Following a complete backend query

So, as we have seen, the schema is the entrance to the backend. From there, it direct the calls to specific resolvers, using the nice [expression language](https://github.com/overblog/GraphQLBundle/blob/master/Resources/doc/definitions/expression-language.md) ofered by the overblog bundle. 

Take a look at the item field or query: 

```yml 
#Queries.types.yml
#...
        fields:
            item:
                type: "Item"
                args:
                    id:
                        type: "ID!"
                resolve: "@=service('app.resolver.items').findOne(args)"
#...
```

It defines the id argument, set it's type as ID and also inform that it's required (!). It also declares the resolver as bing the service defined as "app.resolver.items", that is then requested from symfony dependency injection container. It also passes the received args object straight to the resolver method "findOne". Let's look at "findOne": 

```php 
#AppBundle\Resolver\ItemsResolver.php
#...
	public function findOne($args)
    {   
        $id = $args['id'];
        $item = $this->repo()->find( $id );
        if(!$item) throw new UserErrorException("Item not found for id " . $id, 1);
        return $item;
    }
#...
```

That's a normal code you would expect in a controller, but as you can see, the GraphQL bundle adds some very nice goodies, saving our time on repetitive tasks we usually need to do on controllers or to build as a layer by ourselves.

First, it defines the expected input. Before passing data to the resolver it will check for the 'id' field requirement and type. So, you can be sure it's there when you write your code.

Second, it will transform the object we return in the desired response. I say desired response because this query can be called in different ways, depending on the client needs and will return different fields and even "join fields" according to to what the client has requested. 

After building the response on the server, what is done by the resolver, the GraphQL layer there will check to see if the values are correct according to the return type of that query. 

In that case, the declared type for that query is "Item". And, being so, it will look at Item.types.yml to understand that type and it's field declarations. If the response field types are ok, validated against that, then it will send that response to the client as a 'JsonResponse'. 

```yml
Item:
    type: object
    config:
        description: "A media type"
        fields:
            id:
                type: "ID!"
                description: "The id of the item."
            subtopic:
                type: "Subtopic"
                description: "The subtopic refined by this item"
            name:
                type: "String"
                description: 'The name of this item'
```

If you are like me and like to take a look at the source to understand what you are using, you can start looking at [the controller endpoint action](https://github.com/overblog/GraphQLBundle/blob/master/Controller/GraphController.php#L20). That's the controller that establishes the graphql endpoint for all the requests. From there it will call the executor and make all this fun to happen. And from there you can explore a little more how the bundle integrate the webonyx graphql lib into symfony. 


## A little Recap

Well, this is a lot of information so far, so I suggest a little recap now. I know I explained a lot about GraphQL and it's inner workings, and also about the libs we are using. So, I think that it's important to us to see, after setting up all this army, all this power, what was really left to us: 

1 - define a single field entry on the Query.types.yml
2 - writting the resolver, completely focused on my business logic. 
3 - nothing else. 

Hey, this is beautifull! Take a look: 
    - no need to write a lot of controllers
    - no need to think about routes since you have only one endpoint
        - at least to me thinking in terms of fields and objects is a lot easier than comming up with some restful logic to map uris to business
    - good documentation being generated 
        - most part of it by introspection
    - a tool to interact and test your api (graphiQL). 
    - The best: focus on your business logic. 

As I've said before, while building the backend my first objective was only to have a clean and nice schema to be accessed from outside. But, I got surprised on how the GraphQL approach made my write less boilerplate code on the server. And also satisfied by the resulting code organization. 

Let's talk a little now about some other issues I had to deal in the server development. 

## Cors

This app is not on a prod server yet, but I intend to keep the server in a different env, so I installed the [NelmioCorsBundle|https://github.com/nelmio/NelmioCorsBundle]

The configurations today are very open and will need to be a lot more string on a prod server. But, I just wanted you to note that it's running and will help you to avoid a lot of errors seen on the frontend client. 

It's also worth noticing that I had to add 'content-type' as an allowed header in it's config. 

If you don't know this bundle yet, it's worth taking a look at it. It will manage the headers sent and specially the OPTIONS pre-flight requests to enable cross origin resource sharing. In other words, will enable you to call your API from a different domain. 

## Error Handling

Error handling is a very open topic on GraphQL world. The specs don't say a lot ([1](https://facebook.github.io/graphql/#sec-Errors),[2](https://facebook.github.io/graphql/#sec-Executing-Operations)) and are open to a lot of interpretations. And, as we can see by the community, there are a lot of different opinions on how to handle them ([1](https://voice.kadira.io/masking-graphql-errors-b1b9f15900c1),[2](https://medium.com/@tarkus/validation-and-user-errors-in-graphql-mutations-39ca79cd00bf))

Being GraphQL so recent, it's expected that you don't have well established best practices on it yet. And this is something nice to take into consideration when designing your system. Maybe you see a better way of doing things. So, please do it, test, and share with us. 

Overblog deals with errors in a manner that is good enough me. First, it will add normal validation errors when it encounter them on the data validation of input or output types. 

Second, it will handle the exceptions thrown in the resolvers. When it catches an exception, it will add an error to the response. Almost all exceptions are added as an "internal server error" generic message. The only two Exception types (and subtypes) that are not translated to this generic message are: 

- ErrorHandler::DEFAULT_USER_WARNING_CLASS
- ErrorHandler::DEFAULT_USER_ERROR_CLASS

These can be [configured](https://github.com/overblog/GraphQLBundle/blob/master/DependencyInjection/Configuration.php#L101) on your config.yml with your own exceptions: 

```yml
overblog_graphql:
    definitions:
        exceptions:
            types:
                errors: "\\AppBundle\\Exceptions\\UserErrorException"
```

So in the code we have seen before:

```php 
		#AppBundle\Resolver\ItemsResolver.php
		...
		public function findOne($args)
	    {   
	        $id = $args['id'];
	        $item = $this->repo()->find( $id );
	        if(!$item) throw new UserErrorException("Item not found for id " . $id, 1);
	        return $item;
	    }
        ...
```

if you pass a non existent id to that query, it will return a nice user friendly message:

[Sreen Capture of this happening on GraphiQL]


To understand it a little deeper, please put your diving mask and look at the class that implements this handling: "Overblog\GraphQLBundle\Error\ErrorHandler"

The ErrorHandles is also responsible to put the stack trace on the response. But, it will only do it if the symfony debug is turned on. That is normally done on the "$kernel = new AppKernel('dev', true);" call. I encourage you to test sending a non existent id to that query with debug==true and with debug==false and seing the response. You can do that on GraphiQL. 

To finalize our explanation on this subject, it's also worth noticing that exceptions are also logged on dev.log.  


## Testing your code

The most radical XP coders would say that legacy code is any code that is not tested. Even if it was written today. I'm not so hard on that, but I really like to have a good test suite over my apps. In fact, we started this app with a very different architecture, using other libs and layers, and we would probably not have being able to refactor it to it's actual state of art without it's test suite. 

I did not only refactor the app, but also refactored a lot the test suite itself. It now has a nice utility belt, with tools that enable a good test to be written in the top of the API layer, with calls being made to the GraphQL api. 

To speed up a little, these calls are not simulated as being http calls, but are indeed calls to the GraphQL processor. Given that we have only one endpoint, it does not make sense to test it everytime. 

The response of those calls will allways be a json like structure. A graph of objects, represented and validated by specific GraphQL types. So, I decided to put [peekmo/jsonpath|https://github.com/Peekmo/JsonPath] to work to be able to query those responses in a cleaner syntax. 

At least to me querying for: 

```php
'0.subtopics.0.id'
```

is easier than for: 

```php
$data[0]['subtopics'][0]['id']
```

And readability is a topic that I like to specially enforce on test suites. 

After fighting with it without real arguments for years and years, I now admit without any pain or hurt that PHP code looses a lot to other popular languages in the readability aspect of it's syntax. So, any help on that is welcome. 

Let's take a look at a complete test. Comments are on the code to help your understanding:

```php
	
	function helper() {
        return new SubtopicsTestHelper($this);
    }

	/** @test */
	public function shouldDeleteASubtopic()
	{
		// main responsible for executing queries against graphql layer
		// return a function that enable a json path query on the results
	    $h = $this->helper();

	    //register two subtopics in the first level (children of root node)
	    // ['name'=>'sub1'] is the variables part of the query
	    $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>'sub1']);
	    $h->SUBTOPICS_REGISTER_FIRST_LEVEL(['name'=>'sub2']);

	    // this calls for the root subtopics query. It's executed on the first call that ends just after (['root']=>true)
	    // that will respond with another function that will enable querying the resonse
	    // that function is called with two arguments:  '0.subtopics', '0.subtopics.0.id'
	    // and it returns an array with two results, one for each argument, that are processed by list and assigned to $subtopics and $subId_1
	    list(
	        $subtopics,
	        $subId_1
	    ) = $h->SUBTOPICS_QUERY(
	        ['root'=>true]
	    )(
	        '0.subtopics',
	        '0.subtopics.0.id'
	    );

	    // safety check to verify if both were really inserted
	    $this->assertCount(2,$subtopics);
		
		//here we remove the first added subtopic
	    $ret = $h->SUBTOPIC_DELETE(['id'=>$subId_1])();

	    list(
	        $subtopics,
	        $name
	    ) = $h->SUBTOPICS_QUERY(['root'=>true])(
	        '0.subtopics',
	        '0.subtopics.0.name'
	    );

	    // and finally we check to see if only the second subtopic is present
	    // it's worth noticing that there can be more than one root subtopic. 
	    // that's why we search with 0 (root) . subtopics (it's children) . 0 (first child) . name
	    $this->assertCount(1,$subtopics);
	    $this->assertEquals('sub2',$name);
	}
```

To run tests without errors, I need to clean the cache. I've not inspected the reasons on that yet, but it feels like a bug on the generation of the cached graphql types. Anyway, I just use a command like this everytime: 

```
bin/console cache:clear --env=test;phpunit tests/AppBundle/GraphQL/Subtopics/Mutations/DeleteTest.php
```

I encourage you to open SubtopicsTestHelper and follow and understand the 'proccessResponse' method (Reis\GraphQLTestRunner\Runner\Runner::processGraphQL). There you will be able to see the GraphQL call happening and the json path component being wrapped in the returning funcion. 





Some words about Optimization
=============================
			
	- Optimização
		Optimize human time, not machine time
			Premature optimization is one of the cardinal sins of programming
			I realized that sticking to one query per resolver actually optimized a far more important parameter: 
				how many hours I spent writing and rewriting code every time the API changed.
			É mais barato ter menos mão de obra e mais infra. 
				Principalmente se temos clareza dos gargalos. 

	Depois disso pense em otimizar performance
		Mensure constantemente. 
		Melhor do que otimizar queries a bancos é usar cache e batching. 
			Há dúvidas se muitas queries simples são realmente melhores que uma grande.
				Mas, independentemente disso, o ganho em ter um código limpo é 
					poder reduzir mão de obra, e a 
					facilidade e velocidade de manutenção e evolução. 
			De qualquer forma não tome isso por verdade. Meça, teste e escolha seu caminho. 

	Performance: 

		https://github.com/overblog/GraphQLBundle/blob/master/Resources/doc/definitions/debug/index.md

		---------
			overblog_graphql:
			    definitions:
			        show_debug_info: true
		---------
			A toolbar do Symfony funciona melhor

	https://www.youtube.com/watch?v=c35bj1AT3X8 - Excelente, mostra o funcionamento de JS assíncrono e Promisses para então explicar o DataLoader.
						
	https://www.youtube.com/watch?v=OQTnXNCDywA - Lee Byron explicando o código fonte do DataLoader.js

	Ferramentas de monitoramento: 
		http://www.apollodata.com/optics

	Referências
		https://scaphold.io/community/blog/apollo-optics-and-your-graphql-server/
		https://dev-blog.apollodata.com/optimizing-your-graphql-request-waterfalls-7c3f3360b051

	Bela Referência: 
		https://dev-blog.apollodata.com/how-to-build-graphql-servers-87587591ded5

	Backend as a Service
		https://scaphold.io/

		
