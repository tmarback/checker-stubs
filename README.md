# checker-stubs

Assorted stub library files for use with the [Checker Framework](https://checkerframework.org/), mainly for unannotated libraries and missing annotations in general.

To use, copy these files in a directory somewhere and specify `-Astubs=path/to/stubs` when compiling with the framework.

Note that these are not *complete* stubs for their libraries, only the parts that cause the most problems.

## Provided stub files

- [reactor.astub](./reactor.astub):

    Adds [@Nullable](https://checkerframework.org/api/org/checkerframework/checker/nullness/qual/Nullable.html) to the return type in the function argument of the `mapNotNull()` method of [Reactor's](https://projectreactor.io/) [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#mapNotNull-java.util.function.Function-) and [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#mapNotNull-java.util.function.Function-) types. The whole point of that method is to allow the function to return `null`, so not having this annotation does not make sense.

- [slf4j.astub](./slf4j.astub):

    Adds [@SideEffectFree](https://checkerframework.org/api/org/checkerframework/dataflow/qual/SideEffectFree.html) to all the methods of [SLF4J's](https://www.slf4j.org/) [Logger](https://www.slf4j.org/api/org/slf4j/Logger.html) interface. 

    This is *technically* not correct, as logging *does* sometimes have technically visible side-effects (like writing to a file or using internal logging queues). However, the whole point of logging is to transparently record runtime information without modifying the actual program state, so for the purposes of program correctness, logging should be *effectively invisible*. The only cases this would not be true is if the program is misconfigured to the point of conflicting with logging resources (like trying to write to the same file) or if there is a bug with the logging implementation, which are arguably outside of the scope of what the Checker Framework is trying to solve.

    Meanwhile, *not* doing this tends make Checker severely problematic when using logging, as any logging call causes type refinement information to be lost:

    ```java
    class Foo {
        private Logger logger = ...
        public @Nullable SomeType obj;

        // ...
        
        public int bar() {
            if ( obj != null ) {
                // Assume obj.baz() does some lengthy and/or important 
                // operation so it should be logged before beginning
                logger.info( "Calling baz" );
                // Causes a nullness error, even though the 
                // logging call should never modify s
                return obj.baz();
            } else {
                return 0;
            }
        }
    }
    ```

    Of course, there are ways around this, such as checking again if `obj` is `null` or using `NullnessUtil`, but it just means more and more boilerplate. Since the chance of `logger` actually changing any visible program state is, as aforementioned, effectively nil (as far as the programmer is concerned), it makes more sense to make these methods `@SideEffectFree` than not.
