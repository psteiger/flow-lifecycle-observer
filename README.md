# flow-lifecycle-observer

[![](https://jitpack.io/v/psteiger/flow-lifecycle-observer.svg)](https://jitpack.io/#psteiger/flow-lifecycle-observer)

This library provides two `Flow<T>` extension functions:

1. `Flow<T>.observeIn(LifecycleOwner)`
2. `Flow<T>.observe(LifecycleOwner, suspend (T) -> Unit)`

Those functions collect the flow and:

1. Destroy its collection job on LifecycleOwner's `onStop()`.
2. Recreates its collection job on LifecycleOwner's `onStart()`.

## Motivation

The main motivation for this library is to have lifecycle-aware collectors much like those that can be launched with:

```kotlin
lifecycleScope.launchWhenStarted {
    flow.collect { }
}
```

The issue with the above approach is that if our flow is a `SharedFlow<T>`, *paused collectors will still be subscribed collectors*, so they will have no effect on `SharedFlow`'s `SharingStarted.WhileSubscribed()` and `SharedFlow<T>.subscriptionCount` configurations.

Read more on https://proandroiddev.com/should-we-choose-kotlins-stateflow-or-sharedflow-to-substitute-for-android-s-livedata-2d69f2bd6fa5?source=activity---post_recommended_rollup

## Installation 

On project-level `build.gradle`, add [Jitpack](https://jitpack.io/) repository:

```groovy
allprojects {
  repositories {
    maven { url 'https://jitpack.io' }
  }
}
```

On app-level `build.gradle`, add dependency:

```groovy
dependencies {
    implementation 'com.github.psteiger:flow-lifecycle-observer:0.1.2'
}
```

## Usage

```kotlin
@AndroidEntryPoint
class NearbyUsersActivity : AppCompatActivity() {
    
    private val viewModel: NearbyUsersViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        viewModel
            .locations
            .onEach { /* new locations received */ }
            .observeIn(this)
            
        viewModel
            .users
            .observe(this) {
                /* new users received */
            }
    }
}
```
