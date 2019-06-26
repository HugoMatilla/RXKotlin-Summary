This is my summary of the  of the course RX Kotlin by raywenderlich.com. I use it while learning and as quick reference. It is not intended to be an standalone substitution of the course so if you really want to learn the concepts here presented, buy the course and use this repository as a reference and guide.

If you are the publisher and think this repository should not be public, just write me an email at hugomatilla [at] gmail [dot] com and I will make it private.

Contributions: Issues, comments and pull requests are welcome 😃 

# RXKotlin
- [RXKotlin](#RXKotlin)
- [1. RXKotlin Basics](#1-RXKotlin-Basics)
  - [1. 1 Basic Types](#1-1-Basic-Types)
  - [1. 2 Observables LifeCycle](#1-2-Observables-LifeCycle)
  - [1. 3 Create Operator](#1-3-Create-Operator)
  - [1. 4 Subjects (Types of observers)](#1-4-Subjects-Types-of-observers)
- [2. Filtering Observables](#2-Filtering-Observables)
  - [2. 1 Filtering Operators](#2-1-Filtering-Operators)
  - [2. 2 Sharing observables](#2-2-Sharing-observables)
  - [2. 3 Schedulers and Timing Operators](#2-3-Schedulers-and-Timing-Operators)
- [3. Transforming Operators](#3-Transforming-Operators)
  - [3. 1 Map](#3-1-Map)
  - [3. 2 FlatMap (Combine Observables)](#3-2-FlatMap-Combine-Observables)
  - [3. 3 SwitchMap](#3-3-SwitchMap)
- [4 Combining Operators](#4-Combining-Operators)
  - [4. 1 StartWith](#4-1-StartWith)
  - [4. 2 Concat](#4-2-Concat)
  - [4. 3 Merge](#4-3-Merge)
  - [4. 4 CombineLatests](#4-4-CombineLatests)
  - [4. 5 Zip](#4-5-Zip)
  - [4. 6 Amb](#4-6-Amb)
  - [4. 7 Reduce](#4-7-Reduce)
  - [4. 8 Scan (reduce with intermediate results)](#4-8-Scan-reduce-with-intermediate-results)
  - [4. 9 Sample Scan and Zip](#4-9-Sample-Scan-and-Zip)
- [5 Combining Operators in Android](#5-Combining-Operators-in-Android)
  - [5. 1 Map](#5-1-Map)
  - [5. 2 Concat](#5-2-Concat)
  - [5. 3 CombineLatest](#5-3-CombineLatest)

# 1. RXKotlin Basics

## 1. 1 Basic Types
* Observable (Publisher)
	* Emits an event containing an element 
* Observer (Subscriber)
	* Can react to each element emitted by the Observable

```kotlin
    val mostPopular: Observable<String> = Observable.just(episodeV)
    val originalTrilogy: Observable<List<String>> = Observable.just(episodeIV, episodeV, episodeVI)
    val prequelTrilogy: Observable<List<String>> = Observable.just(listOf(episodeI, episodeII, episodeIII))
    val sequelTrilogy: Observable<String> = Observable.fromIterable(listOf(episodeVII, episodeVIII, episodeIX))
    val stories: Observable<String> = listOf(solo, rogueOne).toObservable()
```

## 1. 2 Observables LifeCycle
### 1. 2. 1 LifeCycle
* Next: Many Events.
* Completed: Only 1 Event. Observable terminated
* Error: Only 1 Event. Observable terminated

### 1. 2. 2 Disposable
* Subscriptions returns a `Disposable` object
* `Disposable` should be dispose at the appropriate time

### 1. 2. 3 Observable creation types
* **just**: Only one event

```kotlin
Observable.just(episodeIV, episodeV, episodeVI)
	.subscribeBy(
        onNext = { println(it) },
        onComplete = { println("Completed") })
```
```
> 
A New Hope
The Empire Strikes Back
Return of the Jedi
Completed
```
* **empty**: No event, just `Completed`

```kotlin
Observable.empty<Unit>()
	.subscribeBy(
		onNext = { println(it) },
		onComplete = { println("Completed") })
```
```
> Completed
```
* **never**: Infinite events, never `Completed`

```kotlin
Observable.never<Any>()
	.subscribeBy(
        onNext = { println(it) },
        onComplete = { println("Completed") })
```
```
> <empty>
```
### 1. 2. 4   Dispose
* **dispose**
Saving the subscription in a variable let as dispose it whenever we want

```kotlin
val mostPopular: Observable<String> = Observable.just(episodeV, episodeIV, episodeVI)
val subscription = mostPopular.subscribe {println(it)}
subscription.dispose()
```

* **CompositeDisposable**
Let us simplify the dispose task, grouping many subscribers "disposes"

```kotlin
val subscriptions = CompositeDisposable()
subscriptions.add(listOf(episodeVII, episodeI, rogueOne)
        .toObservable()
        .subscribe {
            println(it)
        })
subscriptions.dispose()
```

## 1. 3 Create Operator
**create** is a different way to create Observables

```kotlin
val subscriptions = CompositeDisposable()

val droids = Observable.create<String> { emitter ->
    emitter.onNext("R2-D2")
    emitter.onNext("C-3PO")
    emitter.onNext("K-2SO")
    emitter.onComplete()
}

val observer = droids.subscribeBy(
        onNext = { println(it) },
        onError = { println("Error, $it") },
        onComplete = { println("Completed") })

subscriptions.add(observer)
```
```
>
R2-D2 
C-3PO
K-2SO
Completed
```
### 1. 3. 1 Advance Observables
* **single**: One Next event or an error event

* **completable**: Completed event or an error event

* **maybe**:  One Next, Completed or an error event

All of them use `onSuccess` and `onError` that simplifies the use of RXKotlin.


```kotlin
// single sample
val subscriptions = CompositeDisposable()

    fun loadText(filename: String): Single<String> {
        return Single.create create@{ emitter ->
            val file = File(filename)
            if (!file.exists()) {
                emitter.onError(FileReadError.FileNotFound())
                return@create
            }
            val contents = file.readText(UTF_8)
            emitter.onSuccess(contents)
        }
    }

   val observer = loadText("ANewHope.txt")
            .subscribe(
            	{println(it)},
             	{println("Error, $it")})

	subscriptions.add(observer)
```

### 1. 3. 2 Side Effects
```kotlin
observable
	.doOnSubscribe{...}
	.doOnNext{element -> ...}
	.doOnError{...}
	.doOnComplete{...}
	.doOnDispose{...}

```
## 1. 4 Subjects (Types of observers)
### 1. 4. 1 PublishSubject
* Start Empty
* Emits new next events to new subscribers 

<img src="http://reactivex.io/documentation/operators/images/S.PublishSubject.png" alt="alt text" width="300" >

_PublishSubject_

```kotlin


 val quotes = PublishSubject.create<String>()

        quotes.onNext(itsNotMyFault)

        val subscriptionOne = quotes.subscribeBy(
                onNext = { printWithLabel("1)", it) },
                onComplete = { printWithLabel("1)", "Complete") }
        )
        // > <empty>

        quotes.onNext(doOrDoNot)

		// > 1) doOrDoNot
        
        val subscriptionTwo = quotes.subscribeBy(
                onNext = { printWithLabel("2)", it) },
                onComplete = { printWithLabel("2)", "Complete") }
        )

        quotes.onNext(lackOfFaith)

		// > 1) lackOfFaith
		// > 2) lackOfFaith

        subscriptionOne.dispose()

        quotes.onNext(eyesCanDeceive)

		// > 2) eyesCanDeceive

        quotes.onComplete()

        val subscriptionThree = quotes.subscribeBy(
                onNext = { printWithLabel("3)", it) },
                onComplete = { printWithLabel("3)", "Complete") }
        )

        // > 2) Complete
        // > 3) Complete

        quotes.onNext(stayOnTarget)

        // > <empty>
```

### 1. 4. 2 BehaviorSubject

* Starts with initial value
* Replays initial / latest value to new subscribers
* Stateful
* Maintains state in value property
* Can access the value at any time

<img src="http://reactivex.io/documentation/operators/images/S.BehaviorSubject.png" alt="alt text" width="300" >

_BehaviorSubject_

```kotlin
val subscriptions = CompositeDisposable()

val quotes = BehaviorSubject.createDefault(iAmYourFather)

val subscriptionOne = quotes.subscribeBy(
    onNext = { printWithLabel("1)", it) },
    onError = { printWithLabel("1)", it) },
    onComplete = { printWithLabel("1)", "Complete") }
)

subscriptions.add(subscriptionOne)

// > 1) iAmYourFather

quotes.onError(Quote.NeverSaidThat())

// > 1) NeverSaidThat

subscriptions.add(quotes.subscribeBy(
    onNext = { printWithLabel("2)", it) },
    onError = { printWithLabel("2)", it) },
    onComplete = { printWithLabel("2)", "Complete") }
))

// > 2) NeverSaidThat

```
#### 1. 4. 2. 1 BehaviorSubject State
```kotlin
val subscriptions = CompositeDisposable()

    val quotes = BehaviorSubject.createDefault(mayTheForceBeWithYou)

    println(quotes.value)

    // > mayTheForceBeWithYou

    subscriptions.add(quotes.subscribeBy {
      printWithLabel("1)", it)
    })
	
	// > 1) mayTheForceBeWithYou
    
    quotes.onNext(mayThe4thBeWithYou)

    // > 1) mayThe4thBeWithYou
    
    println(quotes.value)

    // > mayThe4thBeWithYou
```


### 1. 4. 3 ReplaySubject
* Starts empty, with buffer size
* Replays buffer to new subscribers
* Don't make the buffer to big, it will be on memory the live of the Observable

<img src="http://reactivex.io/documentation/operators/images/S.ReplaySubject.png" alt="alt text" width="300" >

_ReplaySubject_

```kotlin

val subscriptions = CompositeDisposable()

val subject = ReplaySubject.createWithSize<String>(2)

subject.onNext(useTheForce)

// > 1) useTheForce

subscriptions.add(subject.subscribeBy(
    onNext = { printWithLabel("1)", it) },
    onError = { printWithLabel("1)", it) },
    onComplete = { printWithLabel("1)", "Complete") }
))

subject.onNext(theForceIsStrong)

// > 1) theForceIsStrong

subscriptions.add(subject.subscribeBy(
    onNext = { printWithLabel("2)", it) },
    onError = { printWithLabel("2)", it) },
    onComplete = { printWithLabel("2)", "Complete") }
))

// > 2) useTheForce
// > 2) theForceIsStrong

```

# 2. Filtering Observables

## 2. 1 Filtering Operators

### 2. 1. 1 IgnoreElements
Do not emit any items from an Observable but mirror its termination notification

<img src="http://reactivex.io/documentation/operators/images/ignoreElements.c.png" alt="alt text" width="300" >

_IgnoreElements_

```kotlin
val subscriptions = CompositeDisposable()

val cannedProjects = PublishSubject.create<String>()

subscriptions.add(
        cannedProjects.ignoreElements() // Returns a Completable, so no onNext in subscribeBy
                .subscribeBy {
                    println("Completed")
                })

cannedProjects.onNext(landOfDroids)
cannedProjects.onNext(wookieWorld)
cannedProjects.onNext(detours)

cannedProjects.onComplete()
```

```
>
Completed
```

### 2. 1. 2 ElementAt
Emit only item n emitted by an Observable

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/elementAt.png" >

_ElementAt_

```kotlin
val quotes = PublishSubject.create<String>()

subscriptions.add(
        quotes.elementAt(2) // Returns a Maybe, subscribe with onSuccess instead of onNext
                .subscribeBy(
                        onSuccess = { println(it) },
                        onComplete = { println("Completed") }
                ))

quotes.onNext(mayTheOdds)
quotes.onNext(liveLongAndProsper)
quotes.onNext(mayTheForce)
```

```
>
mayTheForce
```

### 2. 1. 3 Filter
Emit only those items from an Observable that pass a predicate test

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/filter.png" >

_Filter_

```kotlin
subscriptions.add(
        Observable.fromIterable(tomatometerRatings)
                .filter { movie ->
                    movie.rating >= 90
                }.subscribe {
            println(it)
        })
```

```
>
Movie(title=A New Hope, rating=93)
Movie(title=The Empire Strikes Back, rating=94)
Movie(title=The Force Awakens, rating=93)
Movie(title=The Last Jedi, rating=91)
```
### 2. 1. 4 SkipWhile
Discard items emitted by an Observable until a specified condition becomes false

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/skipWhile.c.png" >

_SkipWhile_

```kotlin
subscriptions.add(
    Observable.fromIterable(tomatometerRatings)
            .skipWhile { movie ->
                movie.rating < 90
            }.subscribe {
        println(it)
    })
```

```
>
Movie(title=A New Hope, rating=93)
Movie(title=The Empire Strikes Back, rating=94)
Movie(title=Return Of The Jedi, rating=80)
Movie(title=The Force Awakens, rating=93)
Movie(title=The Last Jedi, rating=91)
```

### 2. 1. 5 SkipUntil
Discard items emitted by an Observable until a second Observable emits an item

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/skipUntil.png" >

_SkipUntil_

```kotlin
val subject = PublishSubject.create<String>()
val trigger = PublishSubject.create<Unit>()

subscriptions.add(
        subject.skipUntil(trigger)
                .subscribe {
                    println(it)
                })

subject.onNext(episodeI.title)
subject.onNext(episodeII.title)
subject.onNext(episodeIII.title)

trigger.onNext(Unit)

subject.onNext(episodeIV.title)
```

```
>
A New Hope
```

### 2. 1. 6 DistinctUntilChanged
Suppress duplicate items emitted by an Observable

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/skipUntil.png" >

_DistinctUntilChanged_

```kotlin
subscriptions.add(
        Observable.just(Droid.R2D2, Droid.C3PO, Droid.C3PO, Droid.R2D2)
                .distinctUntilChanged()
                .subscribe {
                    println(it)
                })
```

```
>
R2D2
C3PO
```

## 2. 2 Sharing observables
Use share to share same observable and be sure that same data arrives to different subscriptions.

```kotlin
private val selectedPhotosSubject = PublishSubject.create<Photo>()

val newPhotos = fragment.selectedPhotos.share()

subscriptions.add(newPhotos
    .subscribe { photo ->
      imagesSubject.value.add(photo)
    })

subscriptions.add(newPhotos
     .subscribe {
      thumbnailStatus.postValue(ThumbnailStatus.READY)
    })

```
## 2. 3 Schedulers and Timing Operators
### 2. 3. 1 Schedulers
* Abstract thread management
* Where the process takes place
* Schedulers.io
* AndroidSchedulers.mainThreaf()

#### - observeOn
Only directs where events are received after call

#### - fgsubscribeOn
Directs Where entire subscription operates

### 2. 3. 2 Timing Operators
#### - debounce
* If multiple events are emitted back to back, take only the last event, or at least the last event in certain time window
* Great for taking some long running action based on tapping button, limits only latest action


```kotlin
subscriptions.add(newPhotos
    .debounce(250, TimeUnit.MILLISECONDS, AndroidSchedulers.mainThread())
        .subscribe { photo ->
          imagesSubject.value.add(photo)
        })
```

# 3. Transforming Operators
## 3. 1 Map

Transform the items emitted by an Observable by applying a function to each item

<img alt="alt text" width="300" src="http://reactivex.io/documentation/operators/images/map.png" >

_Map_

```kotlin
subscriptions.add(
        Observable.fromIterable(listOf("I", "II", "III", "IV", "V"))
        .map { it.romanNumeralIntValue() }
        .subscribeBy { println(it) })
}
```

```
>
1
2
3
4
5
```

## 3. 2 FlatMap (Combine Observables)
Transform the items emitted by an Observable into Observables, then flatten the emissions from those into a single Observable

<img alt="alt text" width="300" src="https://t1.daumcdn.net/cfile/tistory/99DC09425B0D1CB01F" >

_FlatMap_

```kotlin
val ryan = Jedi(BehaviorSubject.createDefault<JediRank>(JediRank.Youngling))
val charlotte = Jedi(BehaviorSubject.createDefault<JediRank>(JediRank.Youngling))

val student = PublishSubject.create<Jedi>()

subscriptions.add(student
                .flatMap { it.rank }
                .subscribeBy { println(it) })

student.onNext(ryan)
// > Youngling
ryan.rank.onNext(JediRank.Padawan)
// > Padawan
student.onNext(charlotte)
// > Youngling
ryan.rank.onNext(JediRank.JediKnight)
// > JediKnight
charlotte.rank.onNext(JediRank.JediMaster)
// > JediMaster
```

### 3. 2. 1 Flatmap in action
2 consecutive and dependent api calls 

```kotlin
 fun fetchEvents() {
    eventLiveData.value = EventsStore.readEvents()

    val apiResponse = gitHubApi.fetchTopKotlinRepos()

    val eventsSubscription = apiResponse
        // Get top Kotlin Repos and convert the result into an observable to be consumed for the next call
        .flatMap { response: TopResponse ->
            if (response.items == null) { Observable.empty() }
            else { Observable.fromIterable(response.items.map { it["full_name"] as String } )}
        }
        // Fetch the events that return N Observables and convert them into one observable with N*X events
        .flatMap { gitHubApi.fetchEvents(it) }
        .filter { response -> (200..300).contains(response.code())}
        .map { response -> response.body()}
        .filter { objects -> objects.isNotEmpty()}
        .map { objects -> objects.mapNotNull { Event.fromAnyDict(it) } }
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(
            { events -> processEvents(events) },
            { error -> println("Events Error ::: ${error.message}") }
        )
    disposables.add(eventsSubscription)
  }
```


## 3. 3 SwitchMap
Like Flatmap but only produce values from the most recent observable sequence 

<img width="300" src="https://cdn-images-1.medium.com/max/1600/0*siKC8C40lRTO6K3T.png" >

_SwitchMap_

```kotlin
val ryan = Jedi(BehaviorSubject.createDefault<JediRank>(JediRank.Youngling))
val charlotte = Jedi(BehaviorSubject.createDefault<JediRank>(JediRank.Youngling))

val student = PublishSubject.create<Jedi>()

subscriptions.add(student
                .switchMap { it.rank }
                .subscribeBy { println(it) })

student.onNext(ryan)
// > Youngling
ryan.rank.onNext(JediRank.Padawan)
// > Padawan
student.onNext(charlotte)
// > Youngling
ryan.rank.onNext(JediRank.JediKnight)
// > <no emitted>
charlotte.rank.onNext(JediRank.JediMaster)
// > JediMaster
```

# 4 Combining Operators
## 4. 1 StartWith
Emit a specified sequence of items before beginning to emit the items from the source Observable

<img width="300" src="http://reactivex.io/documentation/operators/images/startWith.png" >

_StartWith_

```kotlin
   val prequelEpisodes = Observable.just(episodeI, episodeII, episodeIII)

    val flashback = prequelEpisodes.startWith(listOf(episodeIV, episodeV))

    subscriptions.add(
        flashback
            .subscribe { episode -> println(episode) })
```

```
>
episodeIV
episodeV
episodeI
episodeII
episodeIII
```

## 4. 2 Concat
Emit the emissions from two or more Observables without interleaving them

<img width="300" src="http://reactivex.io/documentation/operators/images/concat.png" >

_Concat_

```kotlin
    val prequelTrilogy = Observable.just(episodeI, episodeII, episodeIII)
    val originalTrilogy = Observable.just(episodeIV, episodeV, episodeVI)

    subscriptions.add(
        prequelTrilogy
            .concatWith(originalTrilogy)
            .subscribe { episode -> println(episode) })
```

```
>
episodeI
episodeII
episodeIII
episodeIV
episodeV
episodeVI
```

## 4. 3 Merge
Combine multiple Observables into one by merging their emissions

<img width="300" src="http://reactivex.io/documentation/operators/images/merge.png" >

_Merge_

```kotlin
   val filmTrilogies = PublishSubject.create<String>()
    val standAloneFilms = PublishSubject.create<String>()

    subscriptions.add(
        filmTrilogies.mergeWith(standAloneFilms)
            .subscribe {
              println(it)
            })

    filmTrilogies.onNext(episodeI)
    filmTrilogies.onNext(episodeII)

    standAloneFilms.onNext(theCloneWars)

    filmTrilogies.onNext(episodeIII)

    standAloneFilms.onNext(solo)
    standAloneFilms.onNext(rogueOne)

    filmTrilogies.onNext(episodeIV)
```

```
>
episodeI
episodeII
theCloneWars
episodeIII
solo
rogueOne
episodeIV
```

## 4. 4 CombineLatests
When an item is emitted by either of two Observables, combine the latest item emitted by each Observable via a specified function and emit items based on the results of this function

<img width="300" src="http://reactivex.io/documentation/operators/images/combineLatest.png" >

_CombineLatests_

```kotlin
val characters = PublishSubject.create<String>()
val primaryWeapons = PublishSubject.create<String>()

subscriptions.add(
        Observables.combineLatest(characters, primaryWeapons) 
        { character, weapon -> "$character: $weapon"}
        .subscribe {println(it)})

characters.onNext(luke)
primaryWeapons.onNext(lightsaber)
characters.onNext(hanSolo)
primaryWeapons.onNext(dl44)
characters.onNext(leia)
primaryWeapons.onNext(defender)
characters.onNext(chewbacca)
primaryWeapons.onNext(bowcaster)
```

```
>
Luke Skywalker: Lightsaber
Han Solo: Lightsaber
Han Solo: DL-44 Blaster
Princess Leia: DL-44 Blaster
Princess Leia: Defender Sporting Blaster
Chewbacca: Defender Sporting Blaster
Chewbacca: Bowcaster
```

## 4. 5 Zip
Combine the emissions of multiple Observables together via a specified function and emit single items for each combination based on the results of this function

<img width="300" src="http://reactivex.io/documentation/operators/images/zip.png" >

_CombineLatests_

```kotlin
subscriptions.add(
            Observables.zip(characters, primaryWeapons) 
            { character, weapon -> "$character: $weapon"}
            .subscribe {println(it)})

characters.onNext(luke)
primaryWeapons.onNext(lightsaber)
characters.onNext(hanSolo)
primaryWeapons.onNext(dl44)
characters.onNext(leia)
primaryWeapons.onNext(defender)
characters.onNext(chewbacca)
primaryWeapons.onNext(bowcaster)
```

```
>
Luke Skywalker: Lightsaber
Han Solo: DL-44 Blaster
Princess Leia: Defender Sporting Blaster
Chewbacca: Bowcaster
```

## 4. 6 Amb
Given two or more source Observables, emit all of the items from only the first of these Observables to emit an item or notification

<img width="300" src="http://reactivex.io/documentation/operators/images/amb.png" >

_Amb_

```kotlin
subscriptions.add(
        prequelEpisodes.ambWith(originalEpisodes)
                .subscribe {println(it)})

originalEpisodes.onNext(EPISODEIV)
prequelEpisodes.onNext(EPISODEI)
prequelEpisodes.onNext(EPISODEII)
originalEpisodes.onNext(EPISODEV)
```

```
>
Episode IV - A New Hope
Episode V - The Empire Strikes Back
```

## 4. 7 Reduce
Apply a function to each item emitted by an Observable, sequentially, and emit the final value

<img width="300" src="http://reactivex.io/documentation/operators/images/reduce.png" >

_Reduce_

```kotlin
subscriptions.add(
    Observable.fromIterable(runtimes.values)
            .reduce { a, b -> a + b }
            .subscribeBy(onSuccess = {println(stringFrom(it))})
        )
```

```
>
24:27
```
## 4. 8 Scan (reduce with intermediate results)
Apply a function to each item emitted by an Observable, sequentially, and emit each successive value
Same as reduce but with intermidiate results

<img width="300" src="http://reactivex.io/documentation/operators/images/scan.png" >

_Reduce_

```kotlin
subscriptions.add(
    Observable.fromIterable(runtimes.values)
            .reduce { a, b -> a + b }
            .subscribeBy(onSuccess = {println(stringFrom(it))})
        )
```

```
>
2:16
4:38
6:16
8:36
10:58
13:20
15:21
17:25
19:39
21:55
24:27
```

## 4. 9 Sample Scan and Zip 
```kotlin
val subscriptions = CompositeDisposable()

val runtimesValues = Observable.fromIterable(runtimes2.values)
val runtimesKeys = Observable.fromIterable(runtimes2.keys)
val scanTotals = runtimesValues.scan { a, b -> a + b }

val results = Observables.zip(runtimesKeys, runtimesValues, scanTotals) {
    key, value, total ->
    Triple(key, value, total)
}

subscriptions.add(results.subscribe {
    println("${it.first}: ${stringFrom(it.second)} ( ${stringFrom(it.third)})")
})
```

```
>
Episode I - The Phantom Menace: 2:16 ( 2:16)
Episode II - Attack of the Clones: 2:22 ( 4:38)
THECLONEWARS: 1:38 ( 6:16)
Episode III - Revenge of the Sith: 2:20 ( 8:36)
Solo: 2:22 ( 10:58)
ROGUE ONE: 2:22 ( 13:20)
Episode IV - A New Hope: 2:01 ( 15:21)
Episode V - The Empire Strikes Back: 2:04 ( 17:25)
Episode VI - Return Of The Jedi: 2:14 ( 19:39)
Episode VII - The Force Awakens: 2:16 ( 21:55)
Episode VIII - The Last Jedi: 2:32 ( 24:27)
```
# 5 Combining Operators in Android
## 5. 1 Map
```kotlin
val subscription = EONET.fetchCategories()
    .map { response ->
        val categories = response.categories
        categories.mapNotNull { EOCategory.fromJson(it) }
    }
    .share()
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe {
        categoriesLiveData.value = it
    }
disposables.add(subscription)
```

## 5. 2 Concat
Download in sequence
```kotlin
fun fetchEvents(forLastDays: Int = 360): Observable<List<EOEvent>> {
    val openEvents = events(forLastDays, false)
    val closedEvents = events(forLastDays, true)
    return openEvents.concatWith(closedEvents)
}

private fun events(forLastDays: Int, closed: Boolean): Observable<List<EOEvent>> {
    val status = if (closed) "closed" else "open"
    return eonet.fetchEvents(forLastDays, status)
            .map { response ->
                val events = response.events
                events.mapNotNull { EOEvent.fromJson(it) }
            }
}
```

## 5. 3 CombineLatest
```kotlin
val eoCategories = EONET.fetchCategories()
    .map { response ->
      val categories = response.categories
      categories.mapNotNull { it ->
        EOCategory.fromJson(it)
      }
    }
//> 1 Element: List<Categories>    

val downloadedEvents = EONET.fetchEvents()
//> 1 Element: List<Events>

val updatedCategories =
    Observables.combineLatest(eoCategories, downloadedEvents) { categoriesResponse, eventsResponse ->
      categoriesResponse.map { category ->
        val cat = category.copy()
        cat.events.addAll(eventsResponse.filter {
          it.categories.contains(category.id)
        })
        cat
      }
    }
```