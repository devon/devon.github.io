---
layout: post
title:  "ClickHouse in Rails"
date:   2021-11-15 15:20:06 +0800
categories: [Rails]
tags: [ruby, rails, database]
---

## Background

ClickHouse is an open-source column-oriented DBMS (columnar database management system) for online analytical processing (OLAP) that allows users to generate analytical reports using SQL queries in real-time.

Let’s try to setup ClickHouse connection in Rails as a second database.

```
Rails 6.1
ruby 3.0.2
ClickHouse version 21.12
```

## Install ClickHouse on MacOSX

```shell
mkdir -p /Applications/ClickHouse
cd /Applications/ClickHouse

wget 'https://builds.clickhouse.com/master/macos/clickhouse'
chmod a+x ./clickhouse

// Add a soft link to /usr/local/bin
ln -s /Applications/ClickHouse/clickhouse /usr/local/bin

// Run the server
clickhouse server
```

Open a new Terminal window to test the connection:

```sql
clickhouse client

show databases;
```

## Rails Integration

Prepare a new rails project:

```shell
rails new house -d=mysql
```

Add the gem `clickhouse-activerecord` to `Gemfile`, and then run `bundle` command:

```ruby
gem 'clickhouse-activerecord'
```

Modify `database.yml` to support multiple databases. Add two databases: primary and clickhouse under `development`.

```yml
development:
  primary:
    <<: *default
    database: house_development

  clickhouse:
    adapter: clickhouse
    database: house_development_clickhouse
    host: 127.0.0.1
    port: 8123
    username: default
    debug: true
    migrations_paths: db/clickhouse_migrate
```

## Model for ClickHouse

Create databases for primary and clickhouse:

```shell
rake db:create
```

Add a new model which connects to ClickHouse with `--database`:

```shell
bin/rails g model Event name:string speed:float --database clickhouse
rake db:migrate
```

We will get two model files: `clickhouse_record.rb` and `events.rb`

Content of `clickhouse_record.rb`:

```ruby
class ClickhouseRecord < ApplicationRecord
  self.abstract_class = true

  connects_to database: { writing: :clickhouse }
end
```

Content of `events.rb`:

```ruby
class Event < ClickhouseRecord
end
```

Visit ClickHouse client to make sure we have the database and table:

```sql
clickhouse client

show databases;
use house_development_clickhouse;
show tables;
```

![useful image]({{ site.url }}/assets/images/2021/clickhouse_in_rails_01.png)

We will find out the database and the table.

## Use ClickHouse in Rails Console

Let's try something fun with ClickHouse:

```ruby
bin/rails console

// Create 3 events
Event.create(name: 'a', speed: 1.2)
Event.create(name: 'b', speed: 1.8)
Event.create(name: 'c', speed: 3.6)

// Calculate
Event.sum(:speed)
Event.average(:speed)
```

## Read more

* [ClickHouse](https://clickhouse.com/docs/en/)
* [Gem: Clickhouse Activerecord](https://github.com/PNixx/clickhouse-activerecord)
* [Rails: multiple databases](https://guides.rubyonrails.org/active_record_multiple_databases.html)