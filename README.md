```
ykarabul@ykarabul:~/GolandProjects/TheBookLibraryApi$ tree .
.
├── cmd
│   └── server
│       └── main.go
├── database
│   ├── connectBookTest.go
│   ├── connectUserTest.go
│   └── dbconnect.go
├── go.mod
├── go.sum
└── src
    ├── apiserver
    │   └── apiserver.go
    └── internal
        ├── service
        │   ├── bookService
        │   │   ├── base.go
        │   │   ├── base_test.go
        │   │   ├── delete.go
        │   │   ├── delete_test.go
        │   │   ├── filterby.go
        │   │   ├── filterby_test.go
        │   │   ├── get.go
        │   │   ├── get_test.go
        │   │   ├── list.go
        │   │   ├── list_test.go
        │   │   ├── request.go
        │   │   ├── response.go
        │   │   ├── set.go
        │   │   ├── set_test.go
        │   │   ├── update.go
        │   │   └── update_test.go
        │   └── userService
        │       ├── base.go
        │       ├── base_test.go
        │       ├── delete.go
        │       ├── delete_test.go
        │       ├── get.go
        │       ├── get_test.go
        │       ├── list.go
        │       ├── list_test.go
        │       ├── request.go
        │       ├── response.go
        │       ├── set.go
        │       ├── set_test.go
        │       ├── update.go
        │       └── update_test.go
        ├── storage
        │   ├── books
        │   │   ├── base.go
        │   │   ├── delete.go
        │   │   ├── delete_test.go
        │   │   ├── filterby.go
        │   │   ├── filterby_test.go
        │   │   ├── get.go
        │   │   ├── get_test.go
        │   │   ├── list.go
        │   │   ├── list_test.go
        │   │   ├── set.go
        │   │   ├── set_test.go
        │   │   ├── update.go
        │   │   └── update_test.go
        │   ├── models
        │   │   ├── bookModel.go
        │   │   └── userModel.go
        │   └── users
        │       ├── base.go
        │       ├── delete.go
        │       ├── delete_test.go
        │       ├── get.go
        │       ├── get_test.go
        │       ├── list.go
        │       ├── list_test.go
        │       ├── set.go
        │       ├── set_test.go
        │       ├── update.go
        │       └── update_test.go
        └── transport
            └── http
                ├── basehttphandler
                │   └── basehttphandler.go
                ├── httpservice
                │   ├── base.go
                │   ├── base_test.go
                │   ├── createBook.go
                │   ├── deleteBook.go
                │   ├── deleteBook_test.go
                │   ├── filterBooks.go
                │   ├── getBook.go
                │   ├── getBooks.go
                │   ├── getBooks_test.go
                │   ├── getBook_test.go
                │   ├── loginUser.go
                │   ├── logoutUser.go
                │   ├── registerUser.go
                │   ├── router.go
                │   └── updateBook.go
                └── middlewares
                    └── jwtmiddleware.go







```

# MongoDB and Golang Working Notes 

## Connection

```go

	serverApi := options.ServerAPI(options.ServerAPIVersion1)
	opts := options.Client().ApplyURI("connection-key").SetServerAPIOptions(serverApi)

	ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)
	client, err := mongo.Connect(ctx, opts)
	if err != nil {
		log.Fatal(err)
	}

	defer client.Disconnect(ctx)

	if err := client.Database("admin").RunCommand(ctx, bson.D{{"ping", 1}}).Err(); err != nil {
		panic(err)
	}
	fmt.Println("Pinged your deployment. You successfully connected to MongoDB!")
```

## DB Create

```go
	quickstartDatabase := client.Database("quickstart")
	podcastsCollection := quickstartDatabase.Collection("podcasts")
	episodesCollection := quickstartDatabase.Collection("episodes")
```

## Create One Document

```go
	newDoc := bson.D{
		{"key1", "value1"},
		{"key2", "value2"},
		{"tags", bson.A{"development", "programming", "coding"}},
	}

	insertResult, err := podcastsCollection.InsertOne(ctx, newDoc)

	if err != nil {
		log.Fatal(err)
	}

```

## Create Multiple Documents

```go
	episodeResult, err := episodesCollection.InsertMany(ctx, []interface{}{
		bson.D{
			{"podcast", insertResult.InsertedID},
			{"title", "Episode #1"},
			{"description", "This is the first episode."},
			{"duration", 25},
		},
		bson.D{
			{"podcast", insertResult.InsertedID},
			{"title", "Episode #2"},
			{"description", "This is the second episode."},
			{"duration", 32},
		},
	})
	if err != nil {
		log.Fatal(err)
	}
```

## Find All Documents

```go
	cursor, err := episodesCollection.Find(ctx, bson.M{})

	var episodes []bson.M
	if err = cursor.All(ctx, &episodes); err != nil {
	    log.Fatal(err)
	}
	for _, episode := range episodes {
	    fmt.Println(episode["title"])
	}

```

## Iterate in Collection

```go
	for cursor.Next(ctx) {
		var episode bson.M
		if err = cursor.Decode(&episode); err != nil {
			log.Fatal(err)
		}
		fmt.Println(episode)
}

```

## Find a Specific Document

```go
	var podcast bson.M
	if err = podcastsCollection.FindOne(ctx, bson.M{"add in filter"}).Decode(&podcast); err != nil {
		log.Fatal(err)
	}
	fmt.Println(podcast)

```

## Filtered Find

```go
	filterCursor, err := episodesCollection.Find(ctx, bson.M{"duration": 25})
	if err != nil {
		log.Fatal(err)
	}
	var episodesFiltered []bson.M
	if err = filterCursor.All(ctx, &episodesFiltered); err != nil {
		log.Fatal(err)
	}
	fmt.Println(episodesFiltered)

```

## Search by Sorted

```go
	opts := options.Find()
	opts.SetSort(bson.D{{"duration", 1}}) // duration ı küçükten büyüğe; -->  -1 büyükten küçüğe sıralayarak getiricek opts döndürüyor
	
	sortCursor, err := episodesCollection.Find(ctx, bson.D{
	    {"duration", bson.D{
	        {"$gt", 24}, // -->>  24 ten büyük olanları bul.  "$gt" filitreliyor
	    }},
	}, opts)  // oppts yi veriyoruz ki yukarıdaki setsort u definelasın
	
	var episodesSorted []bson.M
	if err = sortCursor.All(ctx, &episodesSorted); err != nil {
	    log.Fatal(err)
	}
	fmt.Println(episodesSorted)

```

## Update a One Document

```go
	id, _ := primitive.ObjectIDFromHex("64f62b9827370af0fafd7dd7")
	
	result, err := podcastCollection.UpdateOne(
	    ctx,
	    bson.M{"_id": id},  // setliyceğimiz id yi verdik.
	    bson.D{
	        {"$set", bson.D{{"author", "Vigoo"}}},  // setliyceğimiz key ve value bson.D ye parametre geçiyoruz 
	    },
	)
```

## Update a Multiple Documents

```go
	result, err = podcastCollection.UpdateMany(
	    ctx,
	    bson.M{"title": "Software Developer"},  // title ı S.Developer olanları filtreliyoruz
	    bson.D{
	        {"$set", bson.D{{"author", "Yiğithan Karabulut"}}},  // üstteki filterdan geçen tüm documentlerin author'unu setliyoruz. **Eğer author diye bir field'ı yoksa bu field'ı yeni değeriyle ekler
	    },
	)
```

### Replace Document
```go
	result, err = podcastCollection.ReplaceOne(
	    ctx,
	    bson.M{"author": "Yiğithan Karabulut"}, 	// Filtering
	    bson.M{					// Üstteki filtreden gelen tüm doc. ların içeriğini aşağıdaki map'e geçtiğimiz key-val'larla değiştiriyoruz
	        "title":  "The ykarabul times!",
	        "author": "Yiğit",
	    },
	)
```

## Delete One Document

```go
	result, err := episodesCollection.DeleteOne(ctx, bson.M{"duration": 25})
```

## Delete Multiple Documents

```go
	result, err := episodesCollection.DeleteMany(ctx, bson.M{"duration": 32})
```

## Delete Collection

```go
	databases := client.Database("quickstart")
	podcastCollection := databases.Collection("podcasts")
	podcastCollection.Drop(ctx)
```
## Modeling MongoDB Documents with Native Go Data Structures

```go
type Podcast struct {
	ID     primitive.ObjectID `bson:"_id,omitempty"	   json:"id,omitempty"`
	Title  string             `bson:"title,omitempty"  json:"title,omitempty"`
	Author string             `bson:"author,omitempty" json:"author,omitempty"`
	Tags   []string           `bson:"tags,omitempty"   json:"tags,omitempty"`
}

type Episode struct {
	ID          primitive.ObjectID `bson:"_id,omitempty"`
	Podcast     primitive.ObjectID `bson:"podcast,omitempty"`
	Title       string             `bson:"title,omitempty"`
	Description string             `bson:"description,omitempty"`
	Duration    int32              `bson:"duration,omitempty"`
}

func main() {

	serverApi := options.ServerAPI(options.ServerAPIVersion1)
	opts := options.Client().ApplyURI("connection-key").SetServerAPIOptions(serverApi)

	ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)
	client, err := mongo.Connect(ctx, opts)
	if err != nil {
		log.Fatal(err)
	}

	defer client.Disconnect(ctx)

	quickstartDatabase := client.Database("quickstart")
	podcastsCollection := quickstartDatabase.Collection("podcasts")
	episodesCollection := quickstartDatabase.Collection("episodes")

	mongoPodcast := Podcast{
		Title:  "The MongoDB Podcast",
		Author: "Yiğithan Karabulut",
		Tags:   []string{"mongodb", "nosql"},
	}

	insertResult, err := podcastsCollection.InsertOne(ctx, mongoPodcast)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Inserted %v!\n", insertResult.InsertedID)

	var podcasts []Podcast
	podcastsCursor, err := podcastsCollection.Find(ctx, bson.M{})
	if err != nil {
		panic(err)
	}
	if err = podcastsCursor.All(ctx, &podcasts); err != nil {
		panic(err)
	}
	fmt.Println(podcasts[1].Author)

	var episodes []Episode
	episodeCursor, err := episodesCollection.Find(ctx, bson.M{})
	if err != nil {
		panic(err)
	}
	if err = episodeCursor.All(ctx, &episodes); err != nil {
		panic(err)
	}
	fmt.Println(episodes)
}

```

## Query Operators
```
NAME		DESCRIPTION

$eq		Matches values that are equal to a specified value.
$gt		Matches values that are greater than a specified value.
$gte		Matches values that are greater than or equal to a specified value.
$in		Matches any of the values specified in an array.
$lt		Matches values that are less than a specified value.
$lte		Matches values that are less than or equal to a specified value.
$ne		Matches all values that are not equal to a specified value.
$nin		Matches none of the values specified in an array.
```

### Logical
```
$and		Joins query clauses with a logical AND returns all documents that match the conditions of both clauses.
$not		Inverts the effect of a query expression and returns documents that do not match the query expression.
$nor		Joins query clauses with a logical NOR returns all documents that fail to match both clauses.
$or		Joins query clauses with a logical OR returns all documents that match the conditions of either clause.
```

### Element
```
$exists		Matches documents that have the specified field.
$type		Selects documents if a field is of the specified type.
```

### Evaluation
```
$expr		Allows use of aggregation expressions within the query language.
$jsonSchema	Validate documents against the given JSON Schema.
$mod		Performs a modulo operation on the value of a field and selects documents with a specified result.
$regex		Selects documents where values match a specified regular expression.
$text		Performs text search.
$where		Matches documents that satisfy a JavaScript expression.
```

## Array
```
$all		Matches arrays that contain all elements specified in the query.
$elemMatch	Selects documents if element in the array field matches all the specified $elemMatch conditions.
$size		Selects documents if the array field is a specified size.
```

### Bitwise
```
$bitsAllClear	Matches numeric or binary values in which a set of bit positions all have a value of 0.
$bitsAllSet	Matches numeric or binary values in which a set of bit positions all have a value of 1.
$bitsAnyClear	Matches numeric or binary values in which any bit from a set of bit positions has a value of 0.
$bitsAnySet	Matches numeric or binary values in which any bit from a set of bit positions has a value of 1.
```

### Projection Operators
```
$		Projects the first element in an array that matches the query condition.
$elemMatch	Projects the first element in an array that matches the specified $elemMatch condition.
$meta		Projects the document's score assigned during $text operation.
$slice		Limits the number of elements projected from an array. Supports skip and limit slices.
```

### Miscellaneous Operators
```
$comment	Adds a comment to a query predicate.
$rand		Generates a random float between 0 and 1.
```
