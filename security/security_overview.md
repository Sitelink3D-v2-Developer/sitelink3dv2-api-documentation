# Sitelink3D v2 Security Overview
The Sitelink3D v2 security architecture is simple and powerful. Very little knowledge is required to interact with the system. More advanced usage however requires a more detailed understanding of how Sitelink3D v2 implements authorization. This page introduces the basics of identity and authorization based access control and progresses into more advanced security topics such as trust hierarchies and resource naming.

# Introduction
Welcome! You are at the start of a step-by-step journey through the Sitelink3D v2 security implementation. The concepts described here are simple, yet piecing them together to form a complete understanding is something best done incrementally. The sections on this page achieve just that.

As a developer, there may be a number reasons for wanting to learn about the Sitelink3D v2 security model. This page caters to the simple case of using the system with supplied credentials up to the advanced case of generating security tokens, defining resources and delegating permissions.

Although this page can be read sequentially, readers may feel free to skip sections that are not relevant to the task at hand without compromising the flow of the content. Important information is highlighted as tips or summaries in block quotes.

This page covers the following concepts:
- Limitations of traditional security architectures.
- How Sitelink3D v2 uses possession rather than identity to solve these limitations.
- Defining a globally unique resource naming convention that Sitelink3D v2's security system can reference.
- Defining entities that are responsible for protecting these resources.
- How permission to access resources is issued without identity.
- How issued permissions are trusted to have come from the entity that claims to have issued them.
- Determining that an entity has issued permissions for a resource that it actually controls.
- Implementing fine grained access to resources.
- Delegating the ability to protect resources to other trusted entities.

# How Security Is Traditionally Implemented

## About This Section
In the introduction, we raised the concept of using _possession_ rather than _identity_ as a means of implementing resource protection within Sitelink3D v2. This is a good starting place for our security journey. Before addressing the reason for this design however, it is useful to introduce some context surrounding traditional security architectures.

## The Basics
We all know something about computer access control. It's everywhere. Providing a means by which users of a system can be permitted or denied access to resources is fundamental to the safe operation of any computer system.

When ever you log into your bank account online for example, you provide your identity credentials, and a password, that collectively determine who you are (also known as your identity) and what you can do (also known as your authorization). Providing an identity such as an email address or customer number and a password is typical of a mechanism call IBAC (or Identity Based Access Control). This approach works well for many situations and is a commonly accepted architecture.


> _The Typical IBAC Process_
> 1. A Client passes both Identity information and a request to a Service.
> 2. The Service gives the Identity information to an internal Policy Engine.
> 3. The Policy Engine authenticates the information and then computes and gives a set of Permissions for the Service.
> 4. The Service uses the permissions to allow or deny the Client access to resources and returns an appropriate response.

When we designed Sitelink3D v2 however, we found that being able to seamlessly operate in a B2B environment required more than IBAC could offer. The subsequent sections describe this journey and describe our solution with the help of analogies and examples. If you are already familiar with ZBAC, feel free to jump to subsequent sections.

> _Tip:_ 
> When reading the following sections, try to separate the concepts of identity (who you are) and authorization (what you can do).

# Some Problems With Traditional Security Implementations
## About This Section
In the introduction, we raised the concept of using _possession_ rather than _identity_ as a means of implementing resource protection within Sitelink3D v2. In the previous section we explored how identity can be used to implement security using a mechanism known as IBAC (Identity Based Access Control). This section describes some of the fundamental reasons why Sitelink3D v2 does not use identity for controlling access to resources. It is included for context.

## A Problem Scenario
What if the bank that we mentioned in the introduction (let's call them Giant Bank) wanted to change its business model and on-sell its banking services to other banks? This would mean that new banks could simply purchase the IT services they need to provide everything that an online bank requires to operate. Examples may include services that support account transactions, downloading monthly statements and applying for loans.

Initially this sounds like a great idea for everyone involved. Giant Bank can generate income by on-selling services that it already owns and maintains and any smaller bank, let's call one Tiny Bank, doesn't need to implement these systems themselves. Tiny Bank just needs to rebrand Giant Bank's services on their own, unique looking website.

There's a problem however. When Tiny Bank signs up its first customer, Bob, he goes to log in to his new account via the Tiny Bank website using his email and password. That's fine until Bob selects an operation on the Tiny Bank website that's actually implemented by Giant Bank, say, generate a transaction statement.

Remember, this service is only a rebranded version of Giant Bank's service so the customer's identity and password need to be forwarded on to Giant Bank's servers to be authenticated before the statement can be generated. Tiny Bank's handling of Bob's statement request may have looked something like the following.

```
api.giantbank.com/generate_statement "bob@tinybank.com" "password"
```

> _Example Representation of a Band Customer Using Identity_
> 
> | BOB                 |
> |---------------------|
> | **Bank**            |
> | Tiny Bank           |
> | **Identity**        |
> | bob@tinybank.com    |
> | **Capabilities**    |
> | Generate statements |


This process is required because the service that generates statements needs to first determine whether the requester is authorized to do so. Giant Bank's software hence determines whether the user account exists by looking up bob@tinybank.com in a database that Giant Bank maintains, it checks that the provided password is correct, runs the statement generation if so and then returns a result that is displayed on Tiny Bank's website. 

The same process occurs for all services that have been on-sold and for all customers of not only Tiny Bank, but all the other new institutions that have signed up to use Giant Bank's platform.

> _Some Useful Definitions_
> 
> **Authentication**
>
> The process of determining who you are.
> 
> **Authorization**
>
> The process of determining that you have the right to access a resource.


One can perhaps start to imagine some of the problems that rapidly arise from this approach. Because Giant Bank is required to authenticate the identities of users of its services, it necessarily needs to know about all the customers of all the banks that it on-sells to. Privacy issues aside, the mechanics of this become unmanageable.

The situation gets worse however. Tiny bank may require a customer to log on using an email address but another institution, Loans Are Us, requires their customers to log in with an 8 digit number. Giant Bank now needs to know about the different representations of customers along with the customers themselves.


## The Problems Compound
So far we've only explored one delegated business and one service, namely statement generation via Tiny Bank. In reality, things become much more complex. Let's look at a new on-sold service offered by Giant Bank called apply for loan.

Unlike Tiny Bank which only deals with statement generation, Loans Are Us as a business has purchased the use of the loan application service as it suits its business model to do so.

Loans Are Us are lucky enough to have two clients, Mark and John. John however has sadly fallen behind on his loan payments and has consequently had a restriction placed upon him. He is still able to log in to his Loans Are Us account using his unique 8 digit number and view his balance, but his ability to apply for new loans has been revoked. To enforce this, a modification needs to be made in the access control list (also known as ACL) within Giant Bank.

> _Example Representation of Multiple Bank Customers Using Identity_
> 
> | BOB                 | MARK                                    | JOHN                |
> |---------------------|-----------------------------------------|---------------------|
> | **Bank**            | **Bank**                                | **Bank**            |
> | Tiny Bank           | Loans Are Us                            | Loans Are Us        |
> | **Identity**        | **Identity**                            | **Identity**        |
> | bob@tinybank.com    | 12345678                                | 88881111            |
> | **Capabilities**    | **Capabilities**                        | **Capabilities**    |
> | Generate statements | Generate statements and apply for loans | Generate statements |


In this scenario, John's identity as represented by the number 88881111 needs to be disallowed access to the loan application service. What cost is involved in achieving that? Is a maintenance contract required between Giant Bank and Loans Are Us? How long will John still be able to apply for loans before the Giant Bank team has made the update in their ACL?

These examples are typical of the problems that arise when using identity for access control. They are born from the fact that IBAC (which you'll recall is Identity Based Access Control) is an aggregated model that requires users' identification, however that's represented, to cross domain boundaries (in this case from bank to bank).

> _Shortcomings of IBAC_
> 
> 1. **Identity information is leaked** over service and business boundaries because wide dissemination of identity data is required.
> 2. **It is difficult to manage** fine-grained authorization models because policy information is distributed over multiple Policy Engines in multiple services.
> 3. **Inconsistent resource naming** resulting from diversity in how data is defined and referenced between domains precludes the adoption of a universal naming > convention for resources and sharing of resources across services.
> 4. **No permission delegation** is possible in any practical way.
> 5. **There is vulnerability** to confused deputy attacks due to the use of identity in determining authorization.

Although we've used a simple financial example to illustrate these issues, such problems are broadly observable in computer services that protect resources across team or company boundaries. In order to on-sell services and share protected resources, we obviously need a new approach.

# Implementing Security Using Possession Rather Than Identity

## About This Section
This section describes how Sitelink3D v2 addresses the problems raised in the previous section. Replacing identity with possession is a core concept and is important for a thorough understanding of security within Sitelink3D v2.

## Separating Authentication And Authorization - Introducing ZBAC
So how can we address the shortcomings described previously? Let's consider a federated model which both provides the banks in our previous examples with a level of autonomy in how they deal with their clients and drops the need to track customer identity between their business units. 

Fundamentally, the problems we've discussed so far are the result of mixing the tasks of authentication (determining who you are) with authorization (determining what you can do). What if we separated these tasks out such that our cross domain systems only deal with authorization?

Consider a software service that allows access to resources based on the possession of an authorization token rather than an identity (let's call this service an Authorization Checker). In this paradigm, resource protection becomes much simpler and more secure because any request can be granted or denied based on whether the requester has the token required for the resource they are requesting access to, irrespective of who they are, how they obtained the token or how protected resources are represented.

The job of issuing these authorization tokens can then be delegated to an entity that is within the domain of operation and is hence better equipped and trusted to do so. Let's call this entity an Authorization Issuer. In this example, Tiny Bank and Loans Are Us both issue tokens in exchange for login credentials provided by their own clients. 

Because fine grained control no longer needs to be associated with identities at all levels, this also frees us up to use more general representations of resources (such as customer email addresses or client codes) which are then able to be specialized by the business or domain that knows how to do so. Let's not worry about how resources are represented for now. Resource management is a separate core concept that will be addressed a little later.

> _Dealing with Authorization Only_
>
> Only two roles are required.
> 
> **Authorization Issuer**
> Issues a token that allows the holder to perform an operation.
> 
> **Authorization Checker**
> Determines that a requested operation is permitted by the token accompanying the request.

## A Real World Example
It turns out that the concept of separating authentication and authorization isn't as obscure as perhaps it first appears. In fact, there are numerous examples of this paradigm operating in our day to day lives. Let's use a different real world example to explore this proposal a little.

Think of a policeman who pulls you over for speeding in your car. You pull over because you recognize that the policeman has flashing lights on his car and is wearing a blue uniform. You accept that the possession of these items authorizes him to stop you and issue you a ticket. You don't have a list of all the police officers who work in your town and compare the man's name badge to that list in order to identify him first; the man's identity is irrelevant. He carries with him credentials in the form of a car and a uniform that authorize him to issue you a ticket. How unlucky!

These credentials (namely the car and the uniform) were issued to the man some time ago, perhaps even years, when he joined the police force. The process that resulted in the provision of these items is widely trusted by the community. We accept that the government doesn't hand police credentials out to anyone, but rather runs a stringent screening and recruitment process and subjects police applicants to quality training before deeming them worthy of possessing the car and uniform.

This is an example of how the process of identifying an individual can be done in advance by an independent but trusted entity in order to issue commonly recognized authorization credentials in a distributed environment. In this case, the Authorization Issuer is the government and the Authorization Checker is yourself when you recognize the car with the flashing lights and the blue uniform. The scenario we've just described in fact is exactly the approach taken by an alternative to IBAC called ZBAC (or authoriZation Based Access Control).

> _The Typical ZBAC Process_
> 
> 1. A Client application (such as a website) sends Identity information (only) to a Policy Engine.
> 2. The Policy Engine computes a set of Permissions, signs them to avoid spoofing, and returns them to the Client as a token.
> 3. The Client sends the set of signed Permissions to the Server together with a request.
> 4. The Server uses the set of permissions as before.

In the previous section we listed five shortcomings of IBAC. Here we present the five solutions provided by ZBAC.

> _Benefits of ZBAC_
> 
> 1. **Identity information is contained** within domain boundaries as identity is authenticated locally in order to issue anonymous authorization tokens.
> 2. **It is easy to manage** fine-grained authorization models because policy information is local to the business or domain closest to the user.
> 3. **Consistent resource naming** is achieved because the token is an abstraction that allows separation of inter-business and intra-business concerns.
> 4. **Permission delegation is easy to achieve** by controlling policy and selectively distributing anonymous authorization tokens.
> 5. **There is no vulnerability** to confused deputy attacks as decisions are always based on the possession of valid tokens rather than identities requiring authentication.

## ZBAC In Action Within Sitelink3D v2
Already we have presented enough information to make some sense of the Sitelink3D v2 REST API defined on our [Swagger](https://api.sitelink.topcon.com/swagger/) page. Security tokens are provided to Sitelink3D v2 via the Authorization header parameter.

Sitelink3D v2 security tokens are implemented as a JWT (JSON Web Token) as defined by [RFC7519](https://tools.ietf.org/html/rfc7519).

### Recap
Let's recap what we've described to date. Sitelink3D v2 uses a security architecture called ZBAC which relies on the possession of security tokens rather than the submission of identity information to protect system resources. Two entities are required for this mechanism, an Authorization Issuer and an Authorization Checker. An Authorization Checker is not concerned with how a Client came into possession of security tokens. Security tokens are implemented within Sitelink3D v2 as JWTs (JSON Web Tokens).

The separation of authentication and authorization is a core Sitelink3D v2 security concept. We have yet to define how resources are represented and how an Authorization Issuer and Authorization Checker are implemented; for now, the fact that they exist is all that is required to understand the substitution of possession for identity. More detailed ZBAC information can be found [here](http://www.hpl.hp.com/techreports/2009/HPL-2009-30.pdf).

# Issuing Security Tokens With Sitelink3D v2 Security Nodes

## About This Section
In the last section, we identified that Sitelink3D v2's ZBAC based security model requires two types of entities, an Authorization Issuer and an Authorization Checker. The former generates JWTs that can be used with API requests and the latter accepts JWTs as part of API requests and uses them to determine whether the call is authorized. It is noteworthy that a knowledge of this implementation is not required to use Sitelink3D as a Topcon customer.

This section explores the implementation of the Authorization Issuer as the entity that creates JWTs. This content is not necessary for those who simply want to use JWTs issued to them but is essential for anyone wanting to create their own JWTs that protect resources they own or delegate the creation of JWTs to another Authorization Issuer.

## Purpose And Properties Of Authorization Issuers
Let's now define an Authorization Issuer with enough detail to understand how it works. In Sitelink3D v2, an Authorization Issuer is referred to as a Security Node, or just simply a Node. This terminology is born from the fact that nodes can be nested in a hierarchy.

A Node is simply anything that generates JWTs. A Node for example may just be a folder in a file system which contains the files and applications required to generate JWTs. Alternatively, a Node may be a database that is queried for JWTs. However it is implemented, Nodes generate JWTs and are typically comprised of the following components in order to do so.

> _Components of a Sitelink3D v2 Security Node_
> 
> | A Resource                                                                                                                                                      | A JWT Creation Tool                                                                                                                                               | An Identifier                                                                                                                                                     | An Operation List                                                                                                                     |
> |-----------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
> | Description                                                                                                                                                     | Description                                                                                                                                                       | Description                                                                                                                                                       | Description                                                                                                                           |
> | A JWT would be useless without some resource that it is protecting.                                                                                             | A software utility or tool is required to produce the string that defines a JWT for the resource that the Node controls.                                          | A software utility or tool is required to produce the string that defines a JWT for the resource that the Node controls.                                          | A resource is something that has operations performed on it when it is accessed. Multiple actions are possible.                       |
> | That resource is simply a string that could represent a customer, file, building, construction site or conceptually anything that we want to control access to. | As the format of a JWT is well defined, any executable or script that produces a correctly formatted JWT will suffice and developers are free to write their own. | As the format of a JWT is well defined, any executable or script that produces a correctly formatted JWT will suffice and developers are free to write their own. | A Node needs to contain a list of strings that simply specify operations that it knows about. It uses this list when generating JWTs. |
> | Sitelink3D v2 refers to a resource as a subject.                                                                                                                |                                                                                                                                                                   |                                                                                                                                                                   | Sitelink3D v2 uses an actions field for this purpose.                                                                                 |
> | Example                                                                                                                                                         | Example                                                                                                                                                           | Example                                                                                                                                                           | Example                                                                                                                               |
> | "BankAccount:12345678"                                                                                                                                          | create_jwt.exe                                                                                                                                                    | ad8d2c4...54fa                                                                                                                                                    | "generate statement", "apply for loan"                                                                                                |
## Let's Make A JWT
We now have all the information required to create a basic JWT. In order to demonstrate this, let's return to our bank example from the previous sections. Loans Are Us now has a Node that can issue JWTs. When Mark logs into the Loans Are Us website with his identity and password, the Loans Are Us website generates a JWT that it returns to Mark's web browser.

This process has accepted identity and produced a possession in accordance with the ZBAC approach. After verifying Mark's password, the Loans Are Us web server used its JWT creation tool to produce the JWT. This may have looked something like the following.

```
create_jwt.exe 12345678
```

Mark's web browser now possesses a JWT that contains the information required to define what operations can be performed while logged in. From this point on, the browser will only need to use the JWT when requesting operations. Mark's identity is irrelevant.

> _JWT Tip_
>
> When learning about JWTs, it's natural to expect them to be unreadable or encrypted in some way. This is not the case. Although JWTs are base 64 encoded, there is nothing inherently wrong with the contents of a JWT being public.
> 
> Although this sounds strange at first, recall that the possession of the JWT is all that is required to use the JWT. This means that anyone looking at the contents of a JWT will not obtain knowledge about anything that they don't have authorization for.
> 
> Sitelink3D v2 implements a feature that invalidates a JWT if the contents are altered in any way. This is achieved by the Node signing the JWT payload with a private key. For now however, it is enough to know that a JWT payload can be viewed without compromising security.

For the purpose of illustration, Mark's JWT may simply be the following string that sits in his browser's memory. This is not technically a valid JWT, but it's a visualization that uses the above terminology to solidify the concept. Here we have arbitrarily assigned Loans Are Us with an identifier of ```ad8d2c40-49b2-43ca-bffa-14968b5a54fa```.

```json
{
  "issuer":"ad8d2c4049b243cabffa14968b5a54fa",
  "subject":"BankAccount:12345678",
  "actions":["generate_statement", "apply_for_loan"]
}
```

When Bob now clicks on the statement generation button, the Tiny Bank website implements the request by forwarding on the JWT rather than sending Bob's identity to Giant Bank.

~~```api.giantbank.com/generate_statement "BankAccount:12345678" "bob@tinybank.com" "password"```~~

```api.giantbank.com/generate_statement "BankAccount:12345678" JWT```

# How Sitelink3D v2 Uses Security Tokens To Authorize API Requests

## About This Section
Our journey so far has identified that Sitelink3D v2's ZBAC based security model requires two types of entities, an Authorization Issuer (known as a Node) and an Authorization Checker. The former generates security tokens called JWTs that can be used with API requests. The latter accepts JWTs and uses them to determine whether API calls are authorized.

In the last section we discussed how JWTs are produced. This section explores how they are consumed. This knowledge is useful for a comprehensive understanding of Sitelink3D v2's security architecture but is not necessary for anyone intending to simply use JWTs they have been issued.

## Purpose And Properties Of An Authorization Checker
Let's now define an Authorization Checker with enough detail to understand how it works. In Sitelink3D v2, there is only one Authorization Checker. It is a micro service called Trust Manager. Trust Manager is aware of all Nodes within Sitelink3D v2 and there is a process by which it is made aware of new Nodes.

Trust Manager is not directly queried by an API user. Users will simply make use of the Sitelink3D v2 API services that they need and supply each call with their JWT. Each micro service then passes that JWT on to Trust Manager and asks whether the requested operation is permitted by the supplied JWT. The purpose of Trust Manager is to provide a yes or no answer to two simple questions.

> _Trust Manager Summary_
>
> Trust Manager is Sitelink3D v2's Authorization Checker. It exists to answer two question. Given a JWT and a request:
>
> 1. Is the JWT valid?
> 2. Does the JWT allow the requested operation for the specified resource?

## Let's Check A JWT
We now have all the information required to check a basic JWT. In order to demonstrate this, let's return to our bank example from the previous sections. You'll recall that Mark is in possession of a JWT. As a website user he is unaware of this, but his browser knows to use the JWT with all API requests. To recap, the JWT payload from the previous section is provided again below.

```json
{
 "issuer":"ad8d2c4049b243cabffa14968b5a54fa",
 "subject":"BankAccount:12345678",
 "actions":["generate_statement", "apply_for_loan"]
}
```

You'll also recall that our example API call for generating statements looked something like the following.

```
api.giantbank.com/generate_statement "BankAccount:12345678" JWT
```

When the statement generating code receives this request, it does the same as every other service.

1. Send Trust Manager the supplied JWT and ask whether it allows statement generation on the specified bank account resource.
2. If yes, then proceed with statement generation and return the data.
3. If no, then return unauthorized.

To rephrase, the service handling the request makes a call to Trust Manager in order to ask it the following question. _Does this JWT allow "statements to be generated" on the resource called "BankAccount:12345678"?_

Note that Trust Manager doesn't need to know what it means to generate a statement. Nor does it need to understand anything about bank accounts. Sitelink3D v2's micro services operate in exactly the same manner. The Site Owner service for example uses Trust Manager when it receives a call to create_site to determine whether it should do so based on the JWT supplied by the caller.

Trust Manager simply accepts strings for the requested verb and noun and pulls apart the JWT (or JWTs) to answer the two required questions. Let's look at how it achieves this.

| Is the JWT valid?                                                                                                                                                                                            | Does the JWT allow the requested operation?                                                                                                                                                                  |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Process                                                                                                                                                                                                      | Process                                                                                                                                                                                                      |
| Trust Manager interrogates the issuer field in the JWT. You'll recall that these are globally unique.                                                                                                        | This step guarantees that the Node that produced the JWT was actually entitled to do so. We would reject a JWT created by the "Loans Are Us" Node if it referenced a customer from another bank for example. |
| Trust Manager looks at its list of Nodes to determine whether there is indeed one with that UUID. If so, the JWT has been issued by a known Node and is verified by using that Node's public encryption key. | Firstly we check that the resource reference by the JWT matches the resource property of the Node.                                                                                                           |
| JWT:                                                                                                                                                                                                         | JWT:                                                                                                                                                                                                         |
| ```"issuer":"ad8d2c4049b243cabffa14968b5a54fa"```                                                                                                                                                            | ```"subject":"BankAccount:12345678"```                                                                                                                                                                       |
| Node:                                                                                                                                                                                                        | Node:                                                                                                                                                                                                        |
| ```"ad8d2c4049b243cabffa14968b5a54fa"```                                                                                                                                                                     | ```"BankAccount:12345678"```                                                                                                                                                                                 |
|                                                                                                                                                                                                              | Secondly we check that the requested verb provided to Trust Manager appears in the the Node's Operation List.                                                                                                |
|                                                                                                                                                                                                              | Request:                                                                                                                                                                                                     |
|                                                                                                                                                                                                              | "generate_statement"                                                                                                                                                                                         |
|                                                                                                                                                                                                              | Node:                                                                                                                                                                                                        |
|                                                                                                                                                                                                              | "actions":["generate_statement", "apply_for_loan"]                                                                                                                                                           |
| Answer                                                                                                                                                                                                       | Answer                                                                                                                                                                                                       |
| Yes                                                                                                                                                                                                          | Yes                                                                                                                                                                                                          |

As the answer to both of these questions is Yes, Trust Manager responds to the query with approval and the service can then proceed to complete that operation. It does so without knowing anything about the identity of the Tiny Bank user.

# Defining A Resource Naming Convention

## About This Section
The previous sections presented a very limited yet working example of a JWT being used to authorize use of a service. Astute readers will have already likely noticed some problems or limitations with the information presented so far. In order to introduce some Sitelink3D v2 security concepts, such limitations have been required. This section starts to address these by refining some of the concepts already presented. In particular, this section explains the way Sitelink3D v2 references resources.

## Some Problems With Our Example
As currently defined, the "Loans Are Us" Node is only capable of generating JWTs for a single resource "BankAccount:12345678". That would make for a fairly limited bank. Sitelink3D v2 addresses this limitation by permitting wildcards at the end of resource names. Let's update the Node's resource property with this new refinement.

```"BankAccount*"```

That certainly solves the problem of limitation but does so in a way that introduces the opposite problem. A Node with this resource property would be able to write JWTs that allow statement generation and loan applications for all accounts at all banks. This is easily solved by including a unique identifier as part of the Node's resource property. This is a unique resource prefix and is unrelated to the UUID describing the Node itself. Let's update the Node with this new refinement.

```"9b178e64-322c-4f23-9252-bd3b6c96823cBankAccount*"```

Given that the architecture guarantees uniqueness of the prefixed UUID, we have achieved a functional solution. The Loans Are Us Node can now generate JWTs for any account number but the fact that the resource string has been prefixed with an identifier unique to the resource, it can't issue JWTs for accounts controlled by other Nodes. You'll recall that this is enforced when Trust Manager compares the resource string in the JWT with the resource string in the Node. Although functional, the provided syntax is somewhat untidy so let's introduce some separators and key:value pair syntax

```"9b178e64-322c-4f23-9252-bd3b6c96823c:BankAccount:*"```

## Updated Example
Now that the Node is able to issue authorization for all accounts that Tiny Bank controls, it is free to be selective when creating JWTs. The Node can limit the capabilities of a JWT simply by restricting the resource string to a subset of the Node's resource property. When Mark logs into his bank account, the bank will now issue him with the following JWT and the process follows as previously described.

```json
{
  "issuer":"ad8d2c4049b243cabffa14968b5a54fa",
  "subject":"9b178e64-322c-4f23-9252-bd3b6c96823c:BankAccount:12345678",
  "actions":["generate_statement", "apply_for_loan"]
}
```

> _Resource Strings in Sitelink3D v2_
> 
> This example employs a UUID to demonstrate how Node resources are made unique. In reality, the unique resource prefix is a human readable string that is assigned to the Node. The important point is that this string is meaningless to the Node; it is simply a unique prefix and depending on the Node, looks something like the following.
> 
> ```urn:X-topcon:le:222ff3c3-f82a-421e-8b0e-ae6fd618a348```

## One Final Refinement To Resource Representation

There's one more modification we can make to the way we represent resources within the Sitelink3D v2 security architecture. The reason for this modification is to separate the unique Node resource prefix string and the wildcard concept. By separating these two concepts, we can introduce great flexibility in terms of controlling resource access with JWTs. The cleanest way to achieve this separation is to split the current resource property into two fields. The first is the Node's resource prefix and the second specifies how resources can be used. Let's call the former a "subject" and the latter a "scope".

> Resource Related Components of a Sitelink3D v2 Security Node
> 
> | Subject. Referenced as "sub".                                                                                                                                                                                                        | Scope. Referenced as "scp".                                                                                                                       |
> |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
> | Description                                                                                                                                                                                                                          | Description                                                                                                                                       |
> | A JWT would be useless without some resource that it is protecting.                                                                                                                                                                  | Rather than just using a "*" wildcard as previously discussed, the Scope is an enumeration that specifies how subject strings can be interpreted. |
> | Rather that specifying a single resource string, the Subject property is simply the Node's unique resource prefix. When the Node generates JWTs, it uses this string and appends any domain or business specific content after that. | The Scope field is one of "*","node","children" or "desc" (for descendant).                                                                       |
> | This means that the Node can control exactly what resource strings appear within a JWT while guaranteeing that the full string will be globally unique.                                                                              | Detailed information on how Scope fields work within Sitelink3D v2 is available in the Swagger documentation.                                     |
> | Example                                                                                                                                                                                                                              | Example                                                                                                                                           |
> | ```"sub":"urn:X-topcon:le:222ff3c3-f82a-421e-8b0e-ae6fd618a348"```                                                                                                                                                                   | ```"scp":"*"```                                                                                                                                   |


## Resource Scopes
The resource qualifier introduced above as the Scope property relies on the fact that Resources can be viewed as a chain of key:value pairs as already intimated. The following resource string illustrates chaining by utilizing the node and child terms in situ.

```"sub": "node:node_value:child:child_value"```

We are now able to precisely restrict resource access in JWTs issued by a Node. Returning to our Loans Are Us example, let's explore one of our "problem scenarios" discussed in a preceding section. You'll recall that Loans Are Us had two clients, Mark and John. John had fallen behind on his loan payments and Loans Are Us may wish to create a resource representation that categorizes so called bad clients. This can be done by chaining terms onto the subject string.

In this case, Loans Are Us may be looking to issue a JWT that restricts access to only accounts that have fallen behind on payments; perhaps intended for issuance to debt collectors. It would after all be inappropriate to issue access to accounts that are not in arrears. Actions appropriate to a debt collector would include generation of statements but not loan applications. The Loans Are Us Node may be configured as follows.

```json
{
  "sub":"urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c",
  "scope": "*",
  "actions":["generate_statement", "apply_for_loan"]
}
```

A JWT issued by Loans Are Us for the debt collectors may look like the following. Noteworthy is the fact that the JWT is a subset of both the subject and operations fields available to the Node. Note also the issuing Node's UUID is referenced by the iss field.

```json
{
  "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c:clients:bad",
  "scp": "*",
  "actions":["generate_statements"],
  "iss":"ad8d2c4049b243cabffa14968b5a54fa"
}
```

Of course when using this new structure, the Loans Are Us Node is free to issue JWTs with any degree of resource chaining as appropriate to their business model. Trust manager will determine that the chained terms are simply appendages to the "sub" field configured within the Node and permitted by the "*" scope.

This illustrates the flexibility that resource chaining provides. This flexibility is born from the fact that it is the Node itself that decides how its own data is structured and represented when it generates JWTs. Trust Manager simply intersects the subject and scope in the JWT with the subject and scope properties that the Node issuing the JWT is configured with to make authorization decisions.

# Defining A Resource Naming Convention
## About This Section
The previous section described a fully refined naming convention that uses key:value pair chaining in conjunction with a scope field to implement fine grained control over resources.

This section further refines the properties of a Node and a JWT and should provide a complete picture of how these entities operate in the production Sitelink3D v2 system.

## Actions
One area to refine before we can look at Node policies is how actions are defined. Previously we have discussed that a Node contains a property called an "action list". In reality, this way of mapping what operations can be performed on specific resources is too limited. A single resource may be operated on by different services and each service will potentially require different access to a single resource. To date we have simplified the discussion by blurring the concepts of services and what those services can do.

This limitation can be addressed with a modification to how operations are defined within Nodes and within JWTs. What we have previously referred to as an action list needs to become a list of service to action mappings. This design allows individual services to be specified and for each service, the specific actions that can be performed on the resource. This is the purpose of the _"Actions"_ field which is abbreviated to _"act"_. Naturally, the "*" wildcard can be used to introduce flexibility in how actions are expressed.

Returning to our banking example, it makes sense that one of the banking services is responsible for managing clients in the bank database. Let's call this service _"client_manager"_. The client_manager service should be able to call the _"add_client"_ and _"rename_client"_ functions on the bank's client resources. It may be the case that all services are able to list the bank's clients. Such an action set would be expressed using our new syntax as follows.

```json
"act": {
  "client_manager":["add_client", "rename_client"],
  "*":["list_clients"]
}
```

Let's add a modified subject and collectively define the actions that the bearer of the token can expect a service to perform on their behalf. This provides a complete picture of what this permission is specifying. A possible JWT may look like the following.

```json
{
  "iss":"ad8d2c4049b243cabffa14968b5a54fa",
  "sub":"urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c:members:clients",
  "scope": "*",
  "act": {
    "client_manager":["add_client", "rename_client"],
    "*":["list_clients"]
  }
}
```

The power of specifying actions in this way becomes more clear when considering how other resources within a bank may be associated with services. A bank may have a representation for their staff that is distinct from their clients for example. Each staff member may be represented by a UUID as part of the subject string.

Let's also assume that there is a human resources service capable of a number of operations. The JWTs that are issued when a staff identity logs on to the bank then depends on the type of employee and hence the human resource operations available in their JWT. The truncated JWT below specifies that everyone can view and edit their own details.

```json
{
  "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c:members:staff:entry:f451ce5e-3726-4067-b3e7-be111b35d00d",
  "act" : {
    "hr-service": ["view", "edit-details"]
  }
}
```

Different people will hold different JWTs. When a manager logs in, the system recognizes his role and issues him with perhaps the following JWT with additional capabilities. Note the broader subject and the extended action list.

```json
{
  "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c:members:staff",
  "act" : {
    "hr-service": ["hire", "fire", "interview", "pay", "work"]
  }
}
```

## Node Policy
The resource scope examples in the previous section along with the action examples just covered have hopefully made one thing obvious. The way that Node properties are represented are very similar to the way that JWTs are structured. In fact, this symmetry is part of the beauty of this design. Let's introduce two new terms that help us better refer to the properties that a Node is configured with.

> _Improved Definitions for Sitelink3D v2 Security Node Properties_
> 
> **Permission**
> 
> Permissions are defined by a collection of common fields present in both a Node and in the payload of a signed JWT. The fields that collectively comprise a permission are the subject "sub", a scope that identifies whether the subject is restricted in some way "scp" and a set of service, action pairings "act".
> 
> A JWT contains only a single permission. Multiple permissions are achieved by issuing multiple JWTs. A Node will by default contain a permission that allows it to issue unrestricted JWTs for its unique subject. Detailed information on how permission fields work within Sitelink3D v2 is available in the Swagger documentation.
> 
> **Policy**
> 
> A policy is a set of permissions. As JWTs can only contain a single permission, policies are exclusive to Nodes. A Node is likely to be responsible for multiple resources, or even different scopes of the same resource chain. This is achieved by simply storing a collection of one or more permissions in a list called a policy. > A node must have at least one permission for it to generate JWTs against.

Armed with these new definitions, we are able to express the Loans Are Us Node policy described above as the following policy comprised of two permissions. The first permission states that the Node is free to chain any number of resource identifier strings to the subject that represents its own resource space and generate JWTs for any services and actions to operate on that space. The second is the permission delegated by Giant Bank that allows for JWTs to be created that control access to the subset of Giant Bank's resource space as defined by the B2B relationship between the banks.

```json
[
  {
    "sub": "urn:X-topcon:le:5524571e-3c95-4f75-a116-7e138436d1a8",
    "scp": "*",
    "act": { "*":["*"] }
  },
  {
    "sub":"urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:bank:9b178e64-322c-4f23-9252-bd3b6c96823c",
    "scp": "*",
    "act":["generate_statement", "apply_for_loan"]
  }
]
```

## The Node Complexity Versus JWT Complexity Spectrum
A Node will generate JWTs that are either more restrictive or equal to the capability of its policy. Obviously, any JWT generated that specifies capabilities that exceed those of the generating Node's policy will be rejected when verified with Trust Manager.

When designing the creation of Nodes and Node policies then, there is a balance that can be struck. When looking to restrict access, at one end of the spectrum a Node policy can be very broad and the JWTs issued by that Node then need to contain the appropriate restrictions in terms of explicit narrowing of the subject, scope and action fields or some combination thereof.

Alternatively, a Node may be created with quite a specific policy that grants it only the ability to issue JWTs for specific services, subjects and scopes. In this case, the Node may then issue unrestricted JWTs that have the effect of resolving to the policies of the Node when intersected by Trust Manager. Of course, any combination of these extremes is also possible. Such scenarios are visualized below. The contents of the policy and JWT can be reversed and still result in the same effective permissions.

**Policy**

```json
[
  {
    "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214",
    "scp": "*",
    "act": { "*":["*"] }
  }
]
```

**JWT**

```json
{
  "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:members:clients:account:12345678",
  "scp": "node",
  "act": {
    "account_service":["view_balance", "deposit", "transfer"]
  }
}
```

**Effective Permissions**

```json
{
  "name":"Loans Are Us",
  "sub": "urn:X-topcon:le:564529a7-3774-4e12-a414-27efb60b8214:members:clients:account:12345678",
  "scope": "node",
  "act": {
    "account_service":["view_balance", "deposit", "transfer"]
  }
}
```

# Conclusion
This walkthrough has explained the fundamental concepts behind the security model used within Sitelink3D v2. The benefits of separating the tasks of authentication and authorization were introduced and the means by which signed JWTs facilitate fine-grained control over resources was explored. The particular permissions required by the services that comprise Sitelink3D v2 are provided in the Swagger documentation.