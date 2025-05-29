---
title: "Early data validation saves you trouble down the line"
author: Stefan Heyder
date: '2025-05-29'
slug: early_data_validation
categories: [data, data science]
tags: [data validation, Pydantic]
draft: false
---
Working with poor-quality data sucks. It leads to bugs due to the implicit assumptions we make that turn out to be incorrect. For example:

> - This column looks like an ID, so surely it is unique (_nope_) and not null (_you wish_).
> - This column is an amount of money in euros, so it should be formatted consistently (_haha_), e.g. it should never contain dollar amounts, it should have commas as decimal separators, it should contain no spaces, etc..

In a recent project we performed a lot of ETL tasks on data that comes from an API. These data are then transformed by a Python package we wrote using airflow and sent back to that API.

Here is the problem: The quality of the data we receive is suboptimal as it is based on input from humans into a web form. To avoid running into problems further down the line, we use a [Pydantic](https://docs.pydantic.dev/)  model that makes our implicit assumptions explicit. It works well and has saved us a lot of worry. However, every time there is a validation error occurs, we have to write ticket for the data to be fixed, resulting in a lot of unnecessary work (this happens roughly once a week).

Much of this could have been avoided, if data validation had taken place much earlier in the process. Although there is validation on the frontend of the web form, there is none on the back end, meaning users can submit anything they want, if they have a JavaScript blocker in place.

## advantages of early data validation
Once data has passed validation, you have ensured that the data conforms to restrictions such as having specific types, having values in a certain range or not being empty.[Thanks to Max for pointing out that this is not an interface, a term that was written here in an earlier version of this post.]{.aside} This means that you can avoid writing many conversion (e.g. string to date, parsing strings to floats, ...) and null checks later on, and you can be confident that nothing will break due to such implicit assumptions. If combined with properly typed and type-checked Python code, this goes a long way of avoiding bugs.

Additionally, early data validation also enables you to fail early. Let's say you persist the data in a database (PostrgeSQL in our case) for one reason or another, and then process that data later on. If you later notice that something is fishy with the data, you will have to clean up or invalidate the faulty data in the database. 

There are probably many more reasons, but  these are the ones that I have noticed in our project.

## hurdles to implement early data validation
However, there may be challenges in implementing these checks early on. In a bigger project, you have to coordinate with everyone that will use the data on a common definition of what a valid data entry is, including technical and non-technical stakeholders. 

If the data ingestion process is already in place before the goals of data processing are specified it may be hard to add validation later on. This may be due to inertia of the system and also monetary restrictions. Additionally, if validation is put into place after some non-validated data have already entered the system, you now have to distinguish between the new and old data, or clean up the old data. 