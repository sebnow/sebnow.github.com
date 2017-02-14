---
layout: post
categories: [programming, nodejs, graphql]
---

[GraphQL][GraphQL] is a new technology developed by Facebook, and fairly
recently [adopted by GitHub][GitHubGraphQL]. It's a query language and
runtime for building APIs, supporting queries, mutations and
[notifications][GraphQLSubscriptions].

GraphQL exposes a graph representation of the API under a single URL,
rather than using multiple URLs as in the case of RESTful APIs. Nodes in
the graph can have edges to other nodes using "fields" on the node.
A field may be given parameters, e.g. to restrict the query. A client
can make arbitrary queries on a graph, potentially projecting large
amounts of data. Nodes can even have edges to themselves, whether
directly or indirectly.


## Let's Go Shopping!

Let's take an online book store as an example. Such a website would keep
track of a list of authors and books. As most stores do these
days, it would probably also have a list of recommended/similar books
under a book listing. The graph would look something like the
following:

![ExampleGraph](http://g.gravizo.com/g?digraph G {
	graph [pad="0.25", nodesep="0.75", ranksep="0.75"]
	Author -> Book [label="Authored"]
	Book -> Author [label="Written by"]
	Book -> Book [label="Similar"]
})

In GraphQL this could be represented as follows;

	type Query {
		book(id: Int): Book
		author(id: Int): Author
	}

	type Book {
		isbn: ID
		author: Author
		similar: [Book]
	}

	type Author {
		id: ID
		name: String
		books: [Book]
	}

If we were to query for a list of recommended books, like so;

	query RecommendedBooks($isbn: ID) {
		book(isbn: $id) {
			isbn,
			author {
				name
			}
			similar {
				isbn
				author {
					name
				}
			}
		}
	}

We might get a response similar to;

	{% highlight json %}
	{
		"isbn": "978-3-16-148410-0",
		"author": {
			"name": "John Smith"
		},
		"similar": [{
			"isbn": "978-3-16-148410-1",
			"author": {
				"name": "Jane Doe"
			}
		}]
	}
	{% endhighlight %}

As you can see the data structures are mutually recursive. GraphQL
implementations, such as [graphql-js][GraphQLJS], handle this perfectly
fine. However, when faced with building a GraphQL API in NodeJS
I started modularizing my code, and separated the API layer from the
[business logic][ServicePattern] and [data access
layers][RepositoryPattern].

I was forced to construct these recursive data structures myself, which
meant one service needed to reference another service, which in turn
referenced the first one. Applying dependency injection wasn't easy.

My first attempt was to use lazy initialization by way of a service
proxy. The `Author` service was provided a `BookProxy` service, which
implemented the same service as the `Book` service. The `Book` service
was provided the `Author` service directly. All the `BookProxy` service
did was initialize the `Book` service and then proxy all calls to it.
This worked fine but it was a really ugly solution, and each recursive
data structure would require a similar hack.


## Break The Mold

GraphQL already solves this issue by way of the `context` parameter
passed to each [type's `resolve` function][GraphQLObjectType].

[GitHubGraphQL]: https://githubengineering.com/the-github-graphql-api
[GraphQLJS]: http://github.com/graphql/graphql-js 
[GraphQLObjectType]: http://graphql.org/graphql-js/type/#graphqlobjecttype
[GraphQLSubscriptions]: http://graphql.org/blog/subscriptions-in-graphql-and-relay
[GraphQL]: http://graphql.org
[RepositoryPattern]: https://martinfowler.com/eaaCatalog/repository.html
[ServicePattern]: https://en.wikipedia.org/wiki/Service_layers_pattern
