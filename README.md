# Migrate example: Paths

When content URLs change during migrations, it is always a good idea to do something to handle the old URLs to prevent them from suddenly starting to throw 404s which are bad for SEO. Since we already discussed about migrating basic data to Drupal 8 and migrating translated content to Drupal 8, in this article, we will discuss how to migrate URL aliases provided by the path module (part of D8 core) and URL redirects provided by the redirect module.

## The Problem

Say we have two CSV files (given to us by the client):

* Article data provided in [article.csv](import/article.csv)
* Category data provided in [category.csv](import/category.csv)

The project requirement is to:

* Migrate the contents of `article.csv` as article nodes.
* Migrate the contents of `category.csv` as terms of a category taxonomy.
* Make the articles accessible at the path `blog/{{ category-slug }}/{{ article-slug }}`.
* Make `blog/{{ slug }}.php` redirect to `article/{{ article-slug }}`.

Here, the term [slug](https://en.wikipedia.org/wiki/Semantic_URL#Slug) refers to a unique URL friendly and SEO friendly string.

## Before We Start

* If you are new to migrations in Drupal 8, I recommend that you read about [migrating basic data to Drupal 8](https://evolvingweb.ca/blog/drupal-8-migration-migrating-basic-data-part-1) first.
* You will need to install [drush](http://www.drush.org/en/master/) for executing the migrations.

## Migrate Node and Category Data

This part consists of two simple migrations:

* The [categories_data](config/install/migrate_plus.migration.example_category_data.yml) migration creates taxonomy terms for each category.
* The [article_data](config/install/migrate_plus.migration.example_article_data.yml) migration creates article nodes.

The article data migration depends on the category data migration to associate each node to a specific category like:

## Generate URL Aliases with Migrations

The next task will be to make the articles available at URLs like `/blog/{{ category-slug }}/{{ article-slug }}`. We use the [example_article_alias](config/install/migrate_plus.migration.example_article_alias.yml) migration to generate these additional URL aliases.

## Generate URL Redirects with Migrations

For the last requirement, we need to generate redirects, which takes us to the redirect module. So, we create another migration named [example_article_redirect](config/install/migrate_plus.migration.example_article_redirect.yml) to generate redirects from `/blog/{{ slug }}.php` to the relevant nodes.

## Migration dependencies

    migration_dependencies:
      required:
        - 'example_article_data'

Since the migration of aliases and the migration of redirects both require access to the ID of the node which was generated during the article data migration, we need to add the above lines to define `migration_dependencies`. It will ensure that the `example_article_data` and `example_category_data` migrations are executed before the alias and redirect migrations. So if we run all the migrations of this example, we should see them executing in the correct order like:

    $ drush mi --tag=migrate-example-paths
    Processed 5 items (5 created, 0 updated, 0 failed, 0 ignored) - done with 'example_category_data'
    Processed 50 items (50 created, 0 updated, 0 failed, 0 ignored) - done with 'example_article_data'
    Processed 50 items (50 created, 0 updated, 0 failed, 0 ignored) - done with 'example_article_alias'
    Processed 50 items (50 created, 0 updated, 0 failed, 0 ignored) - done with 'example_article_redirect'
