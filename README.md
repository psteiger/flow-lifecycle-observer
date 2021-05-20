### Important Note

Google's library `androidx.lifecycle:lifecycle-runtime-ktx:2.4.0-alpha01` introduced three new APIs that fullfil the same use cases of `flow-lifecycle-observer`, and probably more: `LifecycleOwner.addRepeatingJob`, `Lifecycle.repeatOnLifecycle`, and `Flow.flowWithLifecycle`. You can check [this Medium post](https://medium.com/androiddevelopers/a-safer-way-to-collect-flows-from-android-uis-23080b1f8bda) and the [official documentation](https://developer.android.com/jetpack/androidx/releases/lifecycle) for more information.

# flow-lifecycle-observer

[![](https://jitpack.io/v/psteiger/flow-lifecycle-observer.svg)](https://jitpack.io/#psteiger/flow-lifecycle-observer)

This library provides four `Flow<T>` extension functions for collecting flows in a lifecycle-aware manner.

Two of the extensions functions are for collecting `Flow<T>` while `LifecycleOwner` is in `RESUMED` state:

1. `Flow<T>.collectWhileResumedIn(LifecycleOwner)`
2. `Flow<T>.collectWhileResumed(LifecycleOwner, suspend (T) -> Unit)`

The other two are for collecting `Flow<T>` while `LifecycleOwner` is in `STARTED` state:

3. `Flow<T>.collectWhileStartedIn(LifecycleOwner)`
4. `Flow<T>.collectWhileStarted(LifecycleOwner, suspend (T) -> Unit)`

Those functions collect the flow and:

1. Destroy its collection job on LifecycleOwner's `onPause()` (`collectWhileResumed`/`collectWhileResumedIn`) or `onStop()` (`collectWhileStarted`/`collectWhileStartedIn`).
2. Recreates its collection job on LifecycleOwner's `onResume()` (`collectWhileResumed`/`collectWhileResumedIn`) or `onStart()` (`collectWhileStarted`/`collectWhileStartedIn`).

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
    implementation 'com.github.psteiger:flow-lifecycle-observer:0.2.1'
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
            .collectWhileStartedIn(this)
            
        viewModel
            .users
            .collectWhileStarted(this) {
                /* new users received */
            }
    }
}
```
