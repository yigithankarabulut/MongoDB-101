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

## Created One Document

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

## Created Multiple Documents

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

## Updates a Multiple Documents

```go
	result, err = podcastCollection.UpdateMany(
	    ctx,
	    bson.M{"title": "Software Developer"},  // title ı S.Developer olanları filtreliyoruz
	    bson.D{
	        {"$set", bson.D{{"author", "Yiğithan Karabulut"}}},  // üstteki filterdan geçen tüm documentlerin author'unu Y.K yapıyoruz
	    },
	)
```

### Replaced Document
```go
	result, err = podcastCollection.ReplaceOne(
	    ctx,
	    bson.M{"author": "Yiğithan Karabulut"}, 
	    bson.M{																		// Üstteki filtreden gelen tüm doc. ların içeriğini aşağıdaki map'e geçtiğimiz k-v larla değiştiriyoruz
	        "title":  "The ykarabul times!",
	        "author": "Yiğit",
	    },
	)
```

## Deleted One Document

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
