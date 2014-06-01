# 神.java | Shen for Java (Shen 15)

http://shenlanguage.org/

Shen is a portable functional programming language by [Mark Tarver](http://www.lambdassociates.org/) that offers

* pattern matching,
* λ calculus consistency,
* macros,
* optional lazy evaluation,
* static type checking,
* an integrated fully functional Prolog,
* and an inbuilt compiler-compiler.

See also: [shen.clj](https://github.com/hraberg/shen.clj)


## This Java Port

**Shen.java is an [invokedynamic](http://www.slideshare.net/CharlesNutter/jax-2012-invoke-dynamic-keynote) based [K Lambda](http://www.shenlanguage.org/documentation/shendoc.htm) compiler.** All code lives in [`Shen.java`](https://github.com/hraberg/Shen.java/blob/master/src/shen/Shen.java). It passes the Shen test suite.

This port is loosely based on [`shen.clj`](https://github.com/hraberg/shen.clj), but has no dependency on Clojure.
Started as an [interpreter](https://github.com/hraberg/Shen.java/blob/2359095c59435597e5761c72dbe9f0246fad0864/src/shen/Shen.java) using [MethodHandles](http://docs.oracle.com/javase/7/docs/api/java/lang/invoke/MethodHandle.html) as a primitive. It's about 2x faster than `shen.clj`.

Core requirements : 
* [JDK 8 Early Access Release](https://jdk8.java.net/archive/8u20-b14.html). Tested with 8u20 Build b14. Does NOT work with b15 or b16.
* [Maven](http://maven.apache.org/). See [Maven project file](https://github.com/artella-coding/Shen.java/blob/master/pom.xml).

Optional requirements : There's an IntelliJ project, which requires [IDEA 12](http://www.jetbrains.com/idea/download/index.html). 


### To run the REPL:

#### In Linux : 

    export JAVA_HOME=/path/to/jdk1.8.0/with/lambdas
    ./shen.java

    Shen 2010, copyright (C) 2010 Mark Tarver
    www.shenlanguage.org, version 8
    running under Java, implementation: [jvm 1.8.0-ea]
    port 0.1.0-SNAPSHOT ported by Håkan Råberg


    (0-) (define super
           [Value Succ End] Action Combine Zero ->
             (if (End Value)
                 Zero
                 (Combine (Action Value)
                          (super [(Succ Value) Succ End]
                                 Action Combine Zero))))
    super

    (1-) (define for
           Stream Action -> (super Stream Action do 0))
    for

    (2-) (define filter
           Stream Condition ->
             (super Stream
                    (/. Val (if (Condition Val) [Val] []))
                    append
                    []))
    filter

    (3-) (for [0 (+ 1) (= 10)] print)
    01234567890

    (4-) (filter [0 (+ 1) (= 100)]
                 (/. X (integer? (/ X 3))))
    [0 3 6 9 12 15 18 21 24 27 30 33 36 39 42 45 48 51 54 57 60... etc]

#### In Windows XP :

* Click "Download ZIP" button at https://github.com/artella-coding/Shen.java.
Alternatively : https://github.com/artella-coding/Shen.java/archive/master.zip.

* Download Apache maven from http://maven.apache.org/download.cgi & extract.
Suppose extracted directory is C:\Program Files\maven\apache-maven-2.2.1.

* Download jdk8 from https://jdk8.java.net/download.html

  To extract in Windows XP :

    * Right click on jdk-8-fcs-bin-b129-windows-i586-07_feb_2014.exe 

    * Choose 7-Zip, then choose 'extract to "jdk-8-fcs-bin-b129-windows-i586-07_feb_2014"'. Suppose extracted dictory is : 

    C:\Program Files\jdk-8-fcs-bin-b129-windows-i586-07_feb_2014

    * Then within extracted directory, choose tools.zip, right click, choose 7-Zip, and "Extract Here".

    Then create a file run.bat with the following contents at the top level of the jdk directory : 

        @echo off

        set "JAVA_HOME=C:\Program Files\jdk-8-fcs-bin-b129-windows-i586-07_feb_2014"

        FOR /R %%f IN (*.pack) DO "%JAVA_HOME%\bin\unpack200.exe" -r -v "%%f" "%%~pf%%~nf.jar"

    and run it.

* In **buildAndRunWindows.bat** :

  * Set **JAVA_HOME** appropriately (i.e. C:\Program Files\jdk-8-fcs-bin-b129-windows-i586-07_feb_2014 if performed as above).
  * Set **MAVEN_HOME** appropriately (i.e. C:\Program Files\maven\apache-maven-2.2.1) if performed as above).

Then first run of **buildAndRunWindows.bat** performs build.
Subsequent invocations runs the repl.


### The Shen Test Suite

Passes all tests. It is run at the end of the build:

    ./build   # or ./tests if the jar already exists.

    [... loads of output ...]
    passed ... 128
    failed ...0
    pass rate ...100.0%

    ok
    0

    run time: 37.771 secs


It's close to 2x faster than [`shen.clj`](https://github.com/hraberg/shen.clj).


The benchmarks can be run via:

    ./benchmarks


### What works?

* The K Lambda parser.
* KL special forms.
* Partial application.
* Implicit recur.
* Simple Java inter-op (based on Clojure's syntax).
  * This is pretty broken after changes to the number system, plan to revisit this entire area properly, see the road map below.
* [Dominik's tests](https://github.com/hraberg/Shen.java/blob/master/test/shen/PrimitivesTest.java) from [Shen to Clojure](http://code.google.com/p/shen-to-clojure/).
* The REPL.
* Pre-compilation of the `.kl` to `.class` files.
  * It speeds up start-up of the REPL by a factor of 2-3, but preferably I would like to avoid this step.
* The Shen test suite passes.
* Different bootstrap methods for invoke, apply and symbols. Evolving.
* SwitchPoints for symbols - used when redefining functions.
* Cons as persistent collection vs. cons pairs.
* Recompilation of fns based on Shen types or runtime values.
  * This is primarily to narrow types down to primitives, but not sure typing `Object` arguments matter - all calls are linked via indy anyway.
* `long` reused as double via `doubleToLongBits` and a tag, as guards on primitives seems to require boxing.
  * 63 bit precision. bit 0 is a tag that's either 0 for double or 1 for long.
  * This is highly experimental, arithmetic with tagged longs is faster than boxed Java, doubles on par.
  * There's some wonderful potential inlining issue making the first branch taken (long vs. double) faster, and potentially destroys performance of the other one.
  * I think there's some fundamental piece of InvokeDynamic and primitives I don't understand that led me down this road.
  * Would obviously prefer to use real doubles.


### Road Map

This is bound to change as we go:

* Saner choice of target method. Currently this is done by a mix of instanceof guards and earlier also fallback to `ClassCastException`. It's really only used for built-ins.
* Proper arithmetic. Shen.java uses long and double, but currently there's probably a lot of boxing going on. Currently experimenting with long only, see the section above.
* Performance. My use of invokedynamic is pretty naive so far, so there's a lot of work to be done here.
* Revisit how call sites are built and cached, see above.
* Proper Java inter-op. Potentially using [Dynalink](https://github.com/szegedi/dynalink).
* Reader macros/extension for [`edn`](https://github.com/edn-format/edn) to support embedded Clojure-like maps/sets.
* Persistent collections for the above.
* JSR-223 script engine
* Investigate Clojure(Script) -> K Lambda.


## References

[The Book of Shen](http://www.shenlanguage.org/tbos.html) Mark Tarver, 2012

[LISP in Small Pieces](http://pagesperso-systeme.lip6.fr/Christian.Queinnec/WWW/LiSP.html) Christian Queinnec, 1996 "The aim of this book is to cover, in widest possible scope, the semantics and implementations of interpreters and compilers for applicative languages."

[Performance and Evaluation of Lisp Systems](http://dreamsongs.org/Files/Timrep.pdf) Richard P. Gabriel, 1985

[Asm 4.0](http://asm.ow2.org/index.html) Eric Bruneton, 2007-12 -"A Java bytecode engineering library"

[JDK 8 with Lambda support](http://jdk8.java.net/lambda/)

[InvokeDynamic - You Ain't Seen Nothin Yet](http://www.slideshare.net/CharlesNutter/jax-2012-invoke-dynamic-keynote) Charles Nutter, 2012

[JSR292 Cookbook](http://code.google.com/p/jsr292-cookbook/) | [video](http://medianetwork.oracle.com/video/player/1113248965001) Rémi Forax, 2011

[Scheme in one class](https://blogs.oracle.com/jrose/entry/scheme_in_one_class) John Rose, 2010 - Parts of this looks pretty similar actually! Slightly more advanced/complex, has Java interop but no lambdas. Haven't been updated from the older java.dyn package. "semi-compiled" to MHs, no ASM used.

[Optimizing JavaScript and Dynamic Languages on the JVM](http://www.oracle.com/javaone/lad-en/program/schedule/sessions/con5390-enok-1885659.pdf) Marcus Lagergren and Staffan Friberg, 2012

[Nashorn](https://blogs.oracle.com/nashorn/entry/open_for_business) Jim Laskey et al, 2012 "ECMAScript 5.1 that runs on top of JVM."

[Dynalink](https://github.com/szegedi/dynalink) Attila Szegedi, 2010-12 "Dynamic Linker Framework for Languages on the JVM"

[Invokedynamic them all](https://speakerdeck.com/forax/invokedynamic-them-all) Rémi Forax, 2012

[Golo, a lightweight dynamic language for the JVM](http://golo-lang.org/) Julien Ponge, 2013 - "Golo is a simple dynamic, weakly-typed language for the JVM. Built from day 1 with `invokedynamic`."

[Runtime metaprogramming via java.lang.invoke.MethodHandle](http://lampwww.epfl.ch/~magarcia/ScalaCompilerCornerReloaded/2012Q2/RuntimeMP.pdf) Miguel Garcia, 2012 - The idea of building the AST from MethodHandles without using ASM did occur to me, and looks like it could be possible. Not sure you can actually create a fn definition though (see above). Did a spike, doesn't seem easy/worth the hassle, may revisit.

[Patterns and Performance of InvokeDynamic in JRuby](http://bit.ly/jjug-indy-jruby-en) Hiroshi Nakamura, 2012

[InvokeDynamic: Your API for HotSpot](http://www.slideshare.net/boundaryinc/invoke-dynamic-your-api-to-hotspot) Tony Arcieri, 2012

[Invokedynamic and JRuby](http://vimeo.com/27207224) Ola Bini, 2011 - Last time I met Ola he said that he considered to go back to C for his next language VM.

[Dynamate: A Framework for Method Dispatch using invokedynamic](http://www.ec-spride.tu-darmstadt.de/media/ec_spride/secure_software_engineering/theses_1/kamil_erhard_master_thesis.pdf) Kamil Erhard, 2012

[A Lisp compiler for the JVM](http://www.csc.kth.se/utbildning/kth/kurser/DD143X/dkand12/Group2Mads/report/AntonKindestam.pdf) Anton Kindestam, 2012


## License

http://shenlanguage.org/license.html

Shen, Copyright © 2010-2012 Mark Tarver

Shen.java, Copyright © 2012-2013 Håkan Råberg

---
YourKit is kindly supporting open source projects with its full-featured Java Profiler.
YourKit, LLC is the creator of innovative and intelligent tools for profiling
Java and .NET applications. Take a look at YourKit's leading software products:
<a href="http://www.yourkit.com/java/profiler/index.jsp">YourKit Java Profiler</a> and
<a href="http://www.yourkit.com/.net/profiler/index.jsp">YourKit .NET Profiler</a>.
