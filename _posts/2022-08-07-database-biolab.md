---
layout: post
title: "Using databases to organize data in an experimental lab"
slug: "db-wet-lab"
categories:
    - database
    - wet-lab
    - rna-seq
    - statistics
description: How I am using databases to organize and fetch in an experimental lab
---

# Data is everywhere, and nowhere

Working as a bioinformatician[^1] in an academic experimental
lab means dealing with data most of the time. All types of data are available,
ranging from RNA-seq to longitudinal measurements of in vivo experiments. 
They all have their characteristics, but most of them are available in the
same data structure: tables. Usually my colleagues perform experiments,
data is generated and summarized into tables. They usually save these results
in their own folder in a shared server, so if one wants to check it out and 
analyse, it is directly available. There is a caveat there, where is the 
data exactly? Data is there, we all save it on the server, but at the same
time nowhere, we don't know exactly where it is, from which experiment
the data is coming and their specific details.

[^1]: And also as a mathematician, biostatistician and data scientist

What is the solution for this problem? How can we organize the data
so it is easily accessible and has the context that it was generated?

In the next sections I will explain how I use and implement 
databases in an experimental lab. I will also
show the possible landscape of applications by having everything 
centralized into a single database. 

# Database: what is it exactly?

Here a database is meant to be a collection of structured data,
to be more specific, dataframes. Dataframes are 2-dimensional
structures organized in rows and columns. For example, if you
want to structure the price of houses in a neighborhood, you can
organize the data in a table where the columns are address, 
price and number of rooms in the house. Each row will correspond
to a different house. 

In practice what one can do is to simply pool all tables
in a single folder and whenever you want to access the data
you go to this folder and load it up in your software of 
preference. There are several issues, but one of them is that it does 
not scale well and it becomes very hard to deal with. 

Several tools and programming languages have been developed 
to tackle this problem, such as MySQL, MariaDB, PostgreSQL and
SQLite. They help you organize and distribute your databases
to several people at the same time. Another example:
whenever a purchase
is done in a website, a row is added to a table in the database hosted in a 
server. The company's data scientist can then access and retrieve
data from this table by querying it in any language that has 
some support to SQL. 

# What to organize? 

In the lab that I work specifically, several data types are produced,
ranging from RNA-seq to longitudinal measurements of in vivo 
experiments. We wanted to organize these datasets in a centralized
manner with descriptions, so it can be fetched easily and file paths
don't need to be exactly set when analysing data.

More specifically, there are two types of data to be organized:
RNA-seq counts and longitudinal measurements. RNA-seq counts
are tables where rows correspond to genes and columns to 
samples. In longitudinal measurements rows corresponds
to time and columns to samples. They usually have
the same structure, meaning that we could use databases to 
organize and centralize them. 

# How to organize? 

When talking to the IT department of my 
institute[^2],
I asked what are the options to solve this problem. They suggested
to use MariaDB, an open source database created by the original
developers of MySQL. So the IT installed MariaDB in a server that we
have that is currently used for shiny apps as well. 

[^2]: I'm currently working at EPFL in the life science department.

In MariaDB schemas and databases are synonyms. Thus, I created a database
for RNA-seq and one database for the longitudinal measurements. 

The RNA-seq
database has two tables that should always be there, an `experiments` table
and `patient_status`. In the first table, each row corresponds to an
experiment, where some of the columns are the conditions,
number of samples, a unique ID and a description field. The description
column is where we can write
whatever we want that is relative to the experiment. We work 
a lot with patient derived xenografts (PDXs), so it is interesting to have
a table containing some basic information on the PDXs. This information
is saved in the table `patient_status`. There is a column identifying in which
experiments they were used already and what is the age
of the patient when they were collected for example. Lastly, the 
RNA-seq counts for each experiment are individual tables whose name
is the unique ID written in the `experiments` table. This way we bind 
the `experiments` table with all the counts tables in the database. 
If we want to see what are the experiments that have some specific 
condition, we can filter the experiments folder, extract the IDs 
and then retrieve the RNA-seq counts. 

The longitudinal measurements have a similar database structure. There is
an `experiments` table with the same columns as the
RNA-seq database. Since the two data types are inherently
different, they stay in different databases. One could put everything
together and then append the name of the table with the type of
experiment, but I think this way you insulate and organize the
database better.

# The perks of having a database

Organizing all the files in such a way makes it much easier to
collaboret with my colleagues, as they only need to upload the
tables to the database. For the longitudinal measurements this is
very straight forward. The MySQL Workbench software can be used.
For RNA-seq counts, it is best to load directly in the database by
using SQL in the command line. 
For this I developed a package that extracts the column
names of the counts, creates a `.sql` file to create the table in
the database. Then data can be uploaded using the SQL commands directly
from the command line.

Shiny apps become even more powerful when using databases. Instead of
hard coding paths to files in the application, we can specify the
path to the database, usually a link to the host, and connect
directly to it. For example, if you have a shiny app that provides
visualization tools for
some data, as soon as the database is updated, the 
visualization is updated, no changes are necessary in the
shiny app.

Since data is already centralized, this makes onboarding of new
lab members much easier. They can start working with available data,
and their context, straight away, not necessarily needing to talk to
people to discover where data is and how to get them. This makes
exploring the data easier and facilitates the generation of 
hypothesis.  

A final point is that by having a column description
in the `experiments` table, researchers can write what they did
in that experiment, so information is not lost and its details
are kept all in one place. This is extremely important, as 
a lab is mostly consisted of PhD students and post-docs, and
they come and go every few years.

# Possible next steps and suggestions?

These are only two data types I started organizing in the lab. 
I wonder if there are any other ways to organize the database and
other data types to use. For example, qPCR data. How would one organize 
qPCR data, considering that people in the lab use different machines
and therefore they have different outputs and tables. Should one develop
a package that formats their data so it fits the database? 

I would greatly appreciate any advice or suggestions on how to best
use databases in this setting. If you have any word of wisdom or 
criticism feel free to send me an email at carlos.ronchi@epfl.ch or 
a DM on twitter (@chronchi). 
