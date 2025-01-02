# Afina
Exercise in creating a relational database ready to be embedded into .NET process (SQLite like)

I once thought about the fact that on the one hand having a database in most even small projects is more often justified than not - on the other hand using a DBMS leads to unnecessary dependencies and infrastructure deployment (yes, it's fast with docker nowadays), which in principle you could do without.

Among other things, I missed the classical course on DBMS construction, as I was studying a speciality more related to hardware (solid state physics, semiconductors, chip design and simulations and all this stuff). It would be an interesting exercise, I thought, to make my own database. They say that there is no better way to understand something than to try to explain it to someone else. In the case of programming, there's no better way to understand something than to try to write it. :)

After that, I opened my sketchbook and sketched out some uncomplicated requirements:
1. Ability to build into the .NET process and the engine itself written in C# 
2. Availability of providers under EF and Dapper
3. Implementation of the basic pgsql syntax - the subset will definitely not be complete, but the point is to be able to perform classic CRUD operations in their entirety
4. ACID - transactionality
5. Optional ability to listen to the port and have binary compatibility on the interface with tools that are ready to interact with postgresql

Here and later in other md-files I'm going to log my thoughts and progress on the subject.

## High level design
Given the requirements I decided to go from the top level this time, gradually going down in description. So we are talking about the application layer using EF and Dapper, there will certainly be its own code with providers that will eventually operate on System.Data interfaces such as IDbConnection, IDbCommand, IDbTransaction and others. This will be the layer following the application layer - let's call it the interface layer. After that we should organise the work with queries directly and here we can't get away from several things that we can think about at once - firstly, a lexer and a parser for building a syntax tree, secondly, this syntax tree will need to be traversed with certain purposes - that's why it's better to make an abstract visitor on the basis of which to make a scheduler. The query cache and the data cache should be around the same place. And the level below should be used to organise work directly with disks.
