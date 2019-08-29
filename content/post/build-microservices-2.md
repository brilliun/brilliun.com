---
title: "Building Microservices: Designing Fine-Grained Systems - Part II"
description: ""
keywords: []
tags: ['microservice', 'distributed system']
categories: ['Reading']
date: 2019-08-27T23:36:35+09:00

draft: false
isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

# Chapter 5. Splitting the Monolith

## Example: Breaking Foreign Key Relationships
Our reporting code in the finance package will reach into the line item table to pull out the title for the SKU. It may also have a foreign key constraint from the ledger to the line item table, as we see in Figure 5-2.

![Figure 5-2. Foreign key relationship](/images/build-microservices/5-2.png)
*Figure 5-2. Foreign key relationship*

So how do we fix things here? Well, we need to make a change in two places. First, we need to stop the finance code from reaching into the line item table, as this table really belongs to the catalog code, and we don’t want database integration happening once catalog and finance are services in their own rights. The quickest way to address this is rather than having the code in finance reach into the line item table, we’ll expose the data via an API call in the catalog package that the finance code can call. This API call will be the forerunner of a call we will make over the wire, as we see in Figure 5-3.

![Figure 5-3](/images/build-microservices/5-3.png)
*Figure 5-3. Post removal of the foreign key relationship*

At this point it becomes clear that we may well end up having to make two database calls to generate the report. This is correct. Typically concerns around performance are now raised. I have a fairly easy answer to those: how fast does your system need to be? And how fast is it now? If you can test its current performance and know what good performance looks like, then you should feel confident in making a change. Sometimes making one thing slower in exchange for other things is the right thing to do, especially if slower is still perfectly acceptable.

But what about the foreign key relationship? Well, we lose this altogether. This becomes a constraint we need to now manage in our resulting services rather than in the database level. This may mean that we need to implement our own consistency check across services, or else trigger actions to clean up related data. Whether or not this is needed is often not a technologist’s choice to make. For example, if our order service contains a list of IDs for catalog items, what happens if a catalog item is removed and an order now refers to an invalid catalog ID? Should we allow it? If we do, then how is this represented in the order when it is displayed? If we don’t, then how can we check that this isn’t violated? These are questions you’ll need to get answered by the people who define how your system should behave for its users.

## Example: Shared Static Data

I have seen perhaps as many country codes stored in databases (shown in Figure 5-4) as I have written StringUtils classes for in-house Java projects. So what do we do in our music shop if all our potential services read from the same table like this?

![Figure 5-4](/images/build-microservices/5-4.png)
*Figure 5-4. Country codes in the database*

Well, we have a few options. One is to duplicate this table for each of our packages, with the long-term view that it will be duplicated within each service also. This leads to a potential consistency challenge, of course: what happens if I update one table to reflect the creation of Newmantopia off the east coast of Australia, but not another?

A second option is to instead treat this shared, static data as code. Perhaps it could be in a property file deployed as part of the service, or perhaps just as an enumeration. The problems around the consistency of data remain, although experience has shown that it is far easier to push out changes to configuration files than alter live database tables.

A third option, which may well be extreme, is to push this static data into a service of its own right. In a couple of situations I have encountered, the volume, complexity, and rules associated with the static reference data were sufficient that this approach was warranted, but it’s probably overkill if we are just talking about country codes!

## Example: Shared Data

Now let’s dive into a more complex example, but one that can be a common problem when you’re trying to tease apart systems: shared mutable data. Our finance code tracks payments made by customers for their orders, and also tracks refunds given to them when they return items. Meanwhile, the warehouse code updates records to show that orders for customers have been dispatched or received. All of this data is displayed in one convenient place on the website so that customers can see what is going on with their account. To keep things simple, we have stored all this information in a fairly generic customer record table, as shown in Figure 5-5. 

![Figure 5-5](/images/build-microservices/5-5.png)
*Figure 5-5. Accessing customer data: are we missing something?*

So both the finance and the warehouse code are writing to, and probably occasionally reading from, the same table. How can we tease this apart? What we actually have here is something you’ll see often — a domain concept that isn’t modeled in the code, and is in fact implicitly modeled in the database. Here, the domain concept that is missing is that of Customer.

We need to make the current abstract concept of the customer concrete. As a transient step, we create a new package called Customer. We can then use an API to expose Customer code to other packages, such as finance or warehouse. Rolling this all the way forward, we may now end up with a distinct customer service (Figure 5-6). 

![Figure 5-6](/images/build-microservices/5-6.png)
*Figure 5-6. Recognizing the bounded context of the customer*

## Example: Shared Tables

Figure 5-7 shows our last example. Our catalog needs to store the name and price of the records we sell, and the warehouse needs to keep an electronic record of inventory. We decide to keep these two things in the same place in a generic line item table. Before, with all the code merged in together, it wasn’t clear that we are actually conflating concerns, but now we can see that in fact we have two separate concepts that could be stored differently. 

![Figure 5-7](/images/build-microservices/5-7.png)
*Figure 5-7. Tables being shared between different contexts*

The answer here is to split the table in two as we have in Figure 5-8, perhaps creating a stock list table for the warehouse, and a catalog entry table for the catalog details. 

![Figure 5-8](/images/build-microservices/5-8.png)
*Figure 5-8. Pulling apart the shared table*

## Refactoring Databases

### Staging the Break

So we’ve found seams in our application code, grouping it around bounded contexts. We’ve used this to identify seams in the database, and we’ve done our best to split those out. What next? Do you do a big-bang release, going from one monolithic service with a single schema to two services, each with its own schema? I would actually recommend that you split out the schema but keep the service together before splitting the application code out into separate microservices, as shown in Figure 5-9. 

![Figure 5-9](/images/build-microservices/5-9.png)
*Figure 5-9. Staging a service separation*

With a separate schema, we’ll be potentially increasing the number of database calls to perform a single action. Where before we might have been able to have all the data we wanted in a single SELECT statement, now we may need to pull the data back from two locations and join in memory. Also, we end up breaking transactional integrity when we move to two schemas, which could have significant impact on our applications; we’ll be discussing this next. By splitting the schemas out but keeping the application code together, we give ourselves the ability to revert our changes or continue to tweak things without impacting any consumers of our service. Once we are satisfied that the DB separation makes sense, we can then think about splitting out the application code into two services.

## Transactional Boundaries 

Transactions are useful things. They allow us to say these events either all happen together, or none of them happen. They are very useful when we’re inserting data into a database; they let us update multiple tables at once, knowing that if anything fails, everything gets rolled back, ensuring our data doesn’t get into an inconsistent state. Simply put, a transaction allows us to group together multiple different activities that take our system from one consistent state to another — everything works, or nothing changes.

With a monolithic schema, all our create or updates will probably be done within a single transactional boundary. When we split apart our databases, we lose the safety afforded to us by having a single transaction.

### Try Again Later 

The fact that the order was captured and placed might be enough for us, and we may decide to retry the insertion into the warehouse’s picking table at a later date. We could queue up this part of the operation in a queue or logfile, and try again later. For some sorts of operations this makes sense, but we have to assume that a retry would fix it. 

In many ways, this is another form of what is called eventual consistency. Rather than using a transactional boundary to ensure that the system is in a consistent state when the transaction completes, instead we accept that the system will get itself into a consistent state at some point in the future. This approach is especially useful with business operations that might be long-lived.

### Abort the Entire Operation 

Another option is to reject the entire operation. In this case, we have to put the system back into a consistent state. The picking table is easy, as that insert failed, but we have a committed transaction in the order table. We need to unwind this. What we have to do is issue a compensating transaction, kicking off a new transaction to wind back what just happened. For us, that could be something as simple as issuing a DELETE statement to remove the order from the database. Then we’d also need to report back via the UI that the operation failed. Our application could handle both aspects within a monolithic system, but we’d have to consider what we could do when we split up the application code. Does the logic to handle the compensating transaction live in the customer service, the order service, or somewhere else?

But what happens if our compensating transaction fails? It’s certainly possible. Then we’d have an order in the order table with no matching pick instruction. In this situation, you’d either need to retry the compensating transaction, or allow some backend process to clean up the inconsistency later on. This could be something as simple as a maintenance screen that admin staff had access to, or an automated process. 

Now think about what happens if we have not one or two operations we want to be consistent, but three, four, or five. Handling compensating transactions for each failure mode becomes quite challenging to comprehend, let alone implement.

### Distributed Transactions 

An alternative to manually orchestrating compensating transactions is to use a distributed transaction. Distributed transactions try to span multiple transactions within them, using some overall governing process called a transaction manager to orchestrate the various transactions being done by underlying systems. Just as with a normal transaction, a distributed transaction tries to ensure that everything remains in a consistent state, only in this case it tries to do so across multiple different systems running in different processes, often communicating across network boundaries. 

The most common algorithm for handling distributed transactions — especially short-lived transactions, as in the case of handling our customer order — is to use a two-phase commit. With a two-phase commit, first comes the voting phase. This is where each participant (also called a cohort in this context) in the distributed transaction tells the transaction manager whether it thinks its local transaction can go ahead. If the transaction manager gets a yes vote from all participants, then it tells them all to go ahead and perform their commits. A single no vote is enough for the transaction manager to send out a rollback to all parties. 

This approach relies on all parties halting until the central coordinating process tells them to proceed. This means we are vulnerable to outages. If the transaction manager goes down, the pending transactions never complete. If a cohort fails to respond during voting, everything blocks. And there is also the case of what happens if a commit fails after voting. There is an assumption implicit in this algorithm that this cannot happen: if a cohort says yes during the voting period, then we have to assume it will commit. Cohorts need a way of making this commit work at some point. This means this algorithm isn’t foolproof — rather, it just tries to catch most failure cases.

This coordination process also mean locks; that is, pending transactions can hold locks on resources. Locks on resources can lead to contention, making scaling systems much more difficult, especially in the context of distributed systems.

### So What to Do?

If you really need to go ahead with the split, think about moving from a purely technical view of the process (e.g., a database transaction) and actually create a concrete concept to represent the transaction itself. This gives you a handle, or a hook, on which to run other operations like compensating transactions, and a way to monitor and manage these more complex concepts in your system. For example, you might create the idea of an “in-process-order” that gives you a natural place to focus all logic around processing the order end to end (and dealing with exceptions).
