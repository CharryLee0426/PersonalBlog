# How to use Mongodb

## What is Mongodb

Mongodb is an open-source document oriented database developed by Mongodb.Inc. It is classified as a NoSQL database. It is widely used as document database in microservice projects.

## How to install Mongodb

Mongodb has a lot of good tools for us to put our database on cloud. I don't really know about it. Therefore, I installed `mongodb-atlas` at first. However, this is not mongodb running on locally. It contains several shell tools and a project called `atlas` to create a cluster on your mongodb cloud.

If you wanna deploy your database on cloud, you can install `mongodb-atlas` and setup your mongodb atlas.

```
brew install mongodb-atlas
atlas setup
```

If you prefer to run mongodb locally, you should install mongodb-community@6.0 instead. For more comprehensive and detailed information, you can read this guide [Install Mongodb Community Edition](https://www.mongodb.com/docs/manual/administration/install-community/).

## Use Mongodb in shell

It's very necessary to know how to CRUD in mongodb at first.

We have a database called `use sample_mflix` in this tutorial. You can use `show dbs` to show all databases. `show collections` can list all collections in your selected(used) database.

### Create

Insert one record.

```javascript
use sample_mflix

db.movies.insertOne(
  {
    title: "The Favourite",
    genres: [ "Drama", "History" ],
    runtime: 121,
    rated: "R",
    year: 2018,
    directors: [ "Yorgos Lanthimos" ],
    cast: [ "Olivia Colman", "Emma Stone", "Rachel Weisz" ],
    type: "movie"
  }
)
```

Insert many records.

```javascript
use sample_mflix

db.movies.insertMany([
   {
      title: "Jurassic World: Fallen Kingdom",
      genres: [ "Action", "Sci-Fi" ],
      runtime: 130,
      rated: "PG-13",
      year: 2018,
      directors: [ "J. A. Bayona" ],
      cast: [ "Chris Pratt", "Bryce Dallas Howard", "Rafe Spall" ],
      type: "movie"
    },
    {
      title: "Tag",
      genres: [ "Comedy", "Action" ],
      runtime: 105,
      rated: "R",
      year: 2018,
      directors: [ "Jeff Tomsic" ],
      cast: [ "Annabelle Wallis", "Jeremy Renner", "Jon Hamm" ],
      type: "movie"
    }
])
```

Then we can use `db.collection_name.find()` method to find records we added before.

### Read

Read all records.

```
db.movies.find()

// select * from movies;
```

Specify Conditions.

```
db.movies.find( { "title": "The Favourite" } )

// select * from movies where title='The Favourite';

db.movies.find( { rated: { $in: [ "PG", "PG-13" ] } } )

// select * from movies where rated in ("PG", "PG-13");

// and
db.movies.find( { countries: "Mexico", "imdb.rating": { $gte: 7 } } )

// or
db.movies.find( {
     year: 2010,
     $or: [ { "awards.wins": { $gte: 5 } }, { genres: "Drama" } ]
} )
```

### Update

Update a single record.

```
db.movies.updateOne( { title: "Tag" },
{
  $set: {
    plot: "One month every year, five highly competitive friends
           hit the ground running for a no-holds-barred game of tag"
  }
  { $currentDate: { lastUpdated: true } }
})
```

Update several records.

```
db.listingsAndReviews.updateMany(
  { security_deposit: { $lt: 100 } },
  {
    $set: { security_deposit: 100, minimum_nights: 1 }
  }
)
```

Replace all content except id.

```
db.accounts.replaceOne(
  { account_id: 371138 },
  { account_id: 893421, limit: 5000, products: [ "Investment", "Brokerage" ] }
)
```

### Delete

Delete all.

```
db.movies.deleteMany({})
```

Delete all match some condition.

```
db.movies.deleteMany( { title: "Titanic" } )
```

Delete only one match some condition.

```
db.movies.deleteOne( { cast: "Brad Pitt" } )
```
