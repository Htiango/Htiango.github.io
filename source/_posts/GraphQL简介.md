---
title: GraphQL Introduction
date: 2019-11-12 22:51:41
tags: [GraphQL,Django,backend]
categories: 
- CS Knowledge

---

有幸在工作中使用并熟悉了GraphQL，将在本文对其进行简要的介绍，并附上如何构建一个基于Django的GraphQL API 的 tutorial。

<!-- more -->

后端程序猿们都知道REST，这种API的定义方式一直广为流行。不过想必熟悉REST的朋友也知道，在REST下，每个接口对应着一个URL。随着接口的增多，URL的命名和维护的成本也逐步升高，而对文档的维护成本也很高。

熟悉Django的朋友想必也曾经历过维护REST API的痛苦，尤其是看着`urls.py`文件越来越长，文档更新速度永远赶不上需求的变动。

今年年初的时候，有幸全权负责一个internal website的后端部分，在和前端定义接口的时候，我们决定采用GraphQL而非REST。在整个项目进行的过程中，我深刻感受到了GraphQL带来的便利。当时也整理了不少相关的知识点(in English OF COURSE!)，懒得做翻译的我决定就直接把当时的笔记贴过来，希望能够有所帮助！

## Summary

REST APIs requires loading from multiple URLs, while GraphQL APIs can get all the data your app needs in a single request.

There are multiple server libraries for GraphQL. And I mainly use the `Graphene`. And `graphene-django` is also a very good library binding with Django. 

Though I haven't tried its client libraries, but GraphQL does provide a bunch of client libraries. (It's also very easy to implement since only involves HTTP requests)

By reading through the [official documentation](https://graphql.org/learn/), you should be able to know about GraphQL. 

## How to query a GraphQL server
When querying, GraphQL asks for specific fields on objects. We can also pass arguments to fields to get more specific field results. In GraphQL, every field and nested object can get its own set of arguments, making GraphQL a complete replacement for making multiple API fetches.

Since the result object fields may not include the arguments we pass, you can't directly query for the same field with different arguments. So it lets you rename the result (aliase). 

If we requires a fileds multiple times, we can use GraphQL's reusable units called fragments. (construct sets of fields and we can include them in queries wherever we want)

GraphQL queries always end at scalar values.

Each field on each type is backed by a function called the resolver which is provided by the GraphQL server developer. 

A resolver function receives four arguments:
1. **obj**: The previous object, which for a field on the root Query type is often not used.
2. **args**: The arguments provided to the field in the GraphQL query.
3. **context**: A value which is provided to every resolver and holds important contextual information like the currently logged in user, or access to a database.
4. **info**: A value which holds field-specific information relevant to the current query as well as the schema details, also refer to type GraphQLResolveInfo for more details

Resolve function can be asynchronous and needs to be aware of [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). 

Different fields can be resolved concurrently.

## How to mutate server-side data
By convention, any operation involving writes should be sent via mutation. 

While query fields are executed in parallel, mutation fields run in series, one after the other.

GraphQL also supports interfaces. An Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface.

## How to build GraphQL APIs using Django
In Python, we use `Graphene` as the library to implement GraphQL APIs. But when I implemented my Django based GraphQL APIs, I choose the `Graphene-Django` library. It is built upon `Graphene` and has some Django related features. The goal of it is to connect models from Django ORM to graphene object types.

In order to have GraphQL APIs in Django, we have to:
### Put following in project settings: 
    ```python
    INSTALLED_APPS = [
        ...
        # This will also make the `graphql_schema` management command available
        'graphene_django',
        ...
    ]
    ```
### Put following in project url:
    ```python
    urlpatterns = [
        url(r'^admin/', admin.site.urls),
        ...
        url(r'^graphql$', GraphQLView.as_view(graphiql=True, schema=schema})),
    ]
    ```
### Define a global schema (combining all apps schema) in `project.schema.py`:
    ```python
    class Query(app1.schema.Query,
                app2.schema.Query,
                ...,
                graphene.ObjectType):
        pass
    
    class Mutation(app1.schema.Mutation, 
                   app2.schema.Mutation,
                   ...,
                   graphene.ObjectType):
        pass
    
    schema = graphene.Schema(query=Query, mutation=Mutation)
    ```
### Define each Query/Mutation in each relative app. 
This is for the modularization purpose

## GraphQL good and bad
Compared with REST, GraphQL does has a bunch of advantages. But it also has some short-comes.

### The good:
+ Client can get exact what they need and receive data in expected formats. 
+ Users can know what data is available since GraphQL service defines a set of types which completely describe the set of possible data you can query on that service
+ Can fetch all required data with one single request. 
+ ...

### The bad:
+ Caching control. Caching is built into in the HTTP specification which RESTful APIs are able to leverage. But in GraphQL since there is only one endpoint, it's complex to control caching. 
+ Complex to do error handle since queries always return a HTTP status code of 200
+ More engineer work. Developers has to define a GraphQL schema

