##Cognitive Graph Database Structure Draft Proposal

Outlining the graph data model, based on the [Cognitive Network Protocol](http://noduslabs.com/research/cognitive-network-protocol/).

The core of this model are the notions of Concept, Statement, Context, User and Narrative, which are employed to provide a robust way of connecting disjointed pieces of data and knowledge.

=================

###1. Objective
Describe a general framework that could be used to abstract cognitive processes into a network-graph model.
Based on this framework, design a scalable data model that could be used to store, retrieve, and discover new knowledge: in short, which could emulate all the facets of thinking process.

=================

###2. Framework and Conceptual Model

* Tha basic building block of the model is a **concept**
* The **concepts** appear within **statements**
* The **statements** appear within **contexts**
* The **concepts**, the **statements**, and the **contexts** are all made by **users**
* The **narrative** provides a way of representing the **statements** and **concepts** in a sequential order
* The **user** is either the one who makes the statements or the one who receives them.

A **concept** is an entity, which represents a certain consistency of relations over time. [Origin: from Latin concipere, from com- ‘together’ + capere ‘take’]
For example, the "moon" is a concept, because the relations that it evokes have been persistent enough over time within a certain context to certain people, and thus produce an entity.

A **statement** is an utterance, or an expression where **concept** appears.
This can be a short snippet of text, a URL, an image, a video, a gesture etc.
A statement is different from a concept, because it is much less stable and belongs to the moment.
For example, "the moon is full" is a statement which has a contingent meaning, depending on who has uttered it, when, in which context.

A **context** is the "glue" that connects different concepts and statements together, relating them to the circumstances, environment, and individual perception. [Origin: from Latin contextus, from con- ‘together’ + texere ‘to weave’].
This can be a text, a website, a project, an interest list. A context has shorter life span than a concept, but a longer life span than a statement.
For example, "the moon is full" is a statement that appears in the context of this draft proposal.

A **narrative** can be used to represent a story or evolution of knowledge. It is important, because it creates different paths through the concepts and various contexts.
For example, we might want to place the statement "the moon is in the sky" before the statement "the moon is full", to make the latter statement verifiable to those who don't know what the moon is.

A **user** represents the person who's making the statement (perceiver) or the one who receives it (receiver). [See Murakami's 1Q84 novel].
For example, the user-perceive in this case are all the users who wrote this text, and the user-receiver are all those who read it.

**The model attempts to describe as a graph everything that can be expressed through language or any other semiotic system.**

=================

###3. Data Model

The following is the translation of the above framework into the graph database model.
It is specifically designed to allow for any types of requests and to remain flexible enough for future modifications.


There are 5 types of **nodes** , labelled accordingly:

* :Concept
* :Statement
* :Context
* :User
* :Narrative

Each node has the following **node properties**
* .name (which records its name as displayed to the user),
* .uid (a unique node ID),
* .timestamp (the time it was added into the database first).

Some nodes may also have additional properties, such as
* .text for the :Statement nodes, storing the text body,
* .login and .password fields for the :User nodes to allow authentication.

![](/images/basic-neo4j-graph-data-model.png "Basic Neo4J graph data model")

The Figure 1 above shows a basic outline of this data model implemented in Neo4J interface.



There are 6 types of **edges** within the database, labelled as:

* :TO
* :OF
* :AT
* :IN
* :BY
* :THRU

All edges are directed and have several properties to ensure fast information retrieval.

The **:TO** edge type connects the concepts that appear in the same statement together.
Those edges have the following properties:

* .uid (a unique edge ID)
* .timestamp (when the edge was created)
* .context (in which context the edge was created)
* .statement (in which statement the edge occurred)
* .user (which user added the statement where the edge occurred)
* additional properties indicating indicating the .weight and other characteristics of the relationship

The **:OF** edge type connects the concepts to the statements where they occur.
Directed from :Concept to :Statement nodes.
Those edges have the following properties:

* .uid (a unique edge ID)
* .context (the context ID in which the statement occurred)
* .user (the ID of the user who made the statement)

The **:IN** edge type connects the statements to contexts where they occur.
Directed from :Statement to :Concept.
Have the following properties:

* .user
* .timestamp

The **:AT** type of edge connects the concepts to contexts where they occur.
Directed from :Concept to :Context
Have the following properties:

* .timestamp
* .user
* .statement

The **:BY** type of edge connects all the nodes to the user that created them.
Directed towards the :User node.
Have the following properties:

* .timestamp
* .statement (if originating from :Concepts)
* .context (if originating from :Statement)

The **:THRU** type of edge connects nodes in a sequential narrative order.
They are similar to the :TO type of connections, except that their properties are:

* .uid
* .timestamp
* .user
* .narrative (indicates the unique ID of the Narrative node)


### 4. Examples

The Figure 2 below shows two :Statement nodes: "the moon is full" made in the "private" context (think of it as the user's private notes) and "the moon is round" made in the "idea" context.
As you can see, we now also have two :Context nodes ("private" and "idea"), and the total of 3 :Concept nodes (which were selected by user using a hashtag to be added into the system).
The :User node is linked to all of them via a :BY connection.

![](/images/context-concept-statement-graph-model.png "Context, Statement, Concept Graph Data Model in Neo4J")


When there are several users within the system who make the **statements that intersect on conceptual level**, the relation between them is represented within the graph:
Figure 3 below shows that a new :Statement "the moon is the satellite of the earth" was added by another :User (admin) in their own "private" context:

![](/images/multiple-user-neo4j-graph-data-model.png "Multiple user graph in Neo4J")



### 5. Neo4J Graph Database Implementation

The Cypher request to add a short statement, the concepts, the contexts, and link them up to the user in Neo4J:

> MATCH (u:User {name:"infranodus") MERGE (c_private:Context {name:"private",by:"user_uid",uid:"context_uid"}) ON CREATE SET c_private.timestamp="_" MERGE c_private-[:BY{timestamp:"_"}]->u CREATE (s:Statement {name:"#moon #full", text:"the #moon is #full @private", uid:"statement_uid", timestamp:"_"}) CREATE s-[:BY {context:"context_uid",timestamp:"_"}]->u CREATE s-[:IN {user:"user_uid",timestamp:"_"}]->c_private MERGE (moon:Concept {name:"moon"}) ON CREATE SET moon.timestamp="_", moon.uid="concept_uid" MERGE (full:Concept {name:"full"}) ON CREATE SET full.timestamp="_", full.uid="concept_uid" CREATE moon-[:BY {timestamp:"_",statement:"statement_id"}]->u CREATE moon-[:OF {context:"context_id",user:"user_id",timestamp:"_"}]->s  CREATE moon-[:AT {user:"user_uid",timestamp:"_",statement:"statement_uid"}]->c_private CREATE moon-[:TO {context:"context_uid",statement:"statement_uid",user:"user_uid",timestamp:"_",uid:"edge_uid",gapscan:"2",weight:"3"}]->full CREATE full-[:BY {timestamp:"_",statement:"statement_uid"}]->u CREATE full-[:OF {context:"context_uid",user:"user_uid",timestamp:"_"}]->s CREATE full-[:AT {user:"user_uid",timestamp:"_",statement:"statement_uid"}]->c_private;

This request will match the :User node (it should already exist in the system), then find the context named "private" created by that user (if it does not exist already, it will be created), then use the MERGE clause to look up the concepts and add them if they do not yet exist in the system, adding the statement, and the relations between all those nodes.


The statement to retrieve this information (as shown on Figure 1 above):

> MATCH (u:User{name:"infranodus"}), (c:Concept), (s:Statement), (ctx:Context), c-[:BY]->u, s-[:BY]->u, ctx-[:BY]->u RETURN c,s,ctx,u;



### 6. Bypassing Conceptual Layer, Connecting Resources

There may be cases where the users need to **connect different statements together, bypassing conceptual layer**, either in the way the data is displayed or recorded.

In case we are dealing with :Statement nodes that already contain conceptual data, it can be as simple as making a request connecting those statements through shared concepts:

> MATCH (u1:User{name:"user_name"}), (s1:Statement), s1-[:BY]->u1 WITH DISTINCT s1,u1 MATCH (s2:Statement), s2-[:BY]->u1, p=s1<-[:OF]-c-[:OF]->s2 WHERE s1 <> s2 WITH DISTINCT count(p) AS paths, s1, s2 RETURN s1,s2,paths ORDER BY paths DESC;

The Cypher query above will get all the :Statement nodes added by a :User, find which :Concept nodes connect them, and sort those :Statement nodes by the number of relations reach of them has.

In this case the "rich edge" description of the connections between the statements is defined by the concepts that connect them.

Another case, however, is when the user does not want to have any conceptual layer connecting the statements.

For example, there may be 2 or more videos / photographs / statements and other pieces of content that are related (in user's point of view) directly and do not have a conceptual layer describing them.

We will refer to such pieces of content as **Resources**.

However, to maintain the integrity of the database model we propose to represent this additional type of content and behavior in terms of the already existing ones.

Specifically, we propose the following model:

* A **:Statement** is the type of node used to represent a **Resource**;
* Avoiding to introduce a new node type for resources also means that :Statements can easily be converted into Resources;
* The Resource-Statements have a specific syntax, allowing them to store metadata;
* The Resource-Statements maintain the :BY (towards :User) and :AT (towards :Context) link types;
* The Resource-Statements may have Concepts attached to them too, if any connection to the conceptual layer is required;
* The Resource-Statements can be linked to one another directly using the :TO type of connection

It is important to note that as this feature above has not yet been thoroughly tested, it is suggested to add a connection of type :AND with the same properties as the type :TO.
This may make database writing / retrieval easier.

*Example: Linking Resources with No Conceptual Layer*

A user adds a statement "A" of the following kind: "http://youtube.com/video_id_1".
The system automatically recognizes that there are no concepts within this statement.
The statement is added into the system, linking it to the :User and :Context (default).
This statement can be retrieved by the user in relation to that context.

The user then adds another statement "B": "http://somesite.com/file.pdf" and decides to link it to the statement "A".
This statement will then be linked to the statement "A" by the :AND type of connection, taking into account the :Context where it occurred (may be "default"), the User ID, and the linking statement itself.

It is important to note that the linking statement is in fact the rich edge, describing connection between those resources.
It may have a default content of the kind: "http://youtube.com/video_id_1 and http://somesite.com/file.pdf", which user may edit to make it more meaningful.
However, the additional information about the user who actually made this connection as well as the context in which the connection was made already provides sufficient background info to make this connection more meaningful than just a link between two resources.


### 7. Demo Version

A demo version of this database model in Neo4J and Node.Js is implemented in InfraNodus project:
http://github.com/noduslabs/infranodus



### 8. To-Do

* Implement resource metadata model
* Program more interfaces (for OrientDB, TitanDB)
* Create impelemntations in different languages (PHP, Python, Node.Js/Javascript, Java)



=================


####CC BY-SA 4.0 License####

Licensed under Creative Commons, Attribution-ShareAlike  License.
You can remix, tweak, and build upon this work for non-commercial or commercial purposes, as long as you credit the previous contributors and license your new creations under the identical terms, making them open source and available to everybody for free.
https://creativecommons.org/licenses/by-sa/4.0/

Developed by:
[Dmitry Paranyushkin](http://github.com/deemeetree) | [Nodus Labs](http://www.noduslabs.com) | info@noduslabs.com
Your Name | Affiliation | Contact here
