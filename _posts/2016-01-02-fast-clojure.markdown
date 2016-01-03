---
layout: post
title: Fast Clojure
date: 2016-01-02 11:04:03.000000000 -04:00
---

It's been a while since I posted. In the past year or so, my company has adopted
my new favorite language, [Clojure](clojure.org), a lisp that runs ontop of the
JVM with full Java interop.

Some of the code that we write needs to be quite fast. Clojure, out of the box,
is pretty snappy, but to really squeeze out the performance, it takes a mix of
trial and error testing and some understanding of what's happening under the
hood to really get great speed.

# A simple question

Pop Quiz, the following function calls each have the same result, which is the
fastest?

```clojure
(first [1 2 3 4 5 6])
```

```clojure
(get [1 2 3 4 5 6] 0)
```

```clojure
([1 2 3 4 5 6] 0)
```

Correct answer? **"I don't know, lets test it and find out."**

# Timing in the repl

Clojure makes this super easy. There are two built-in macros I use when testing,
`time` which counts how long something takes, and `dotimes` which will do
something some number of times.

```clojure
user=> (def some-vec [1 2 3 4 5 6])
#'user/some-vec
user=> (time (dotimes [_ 100000000] (first some-vec)))
"Elapsed time: 7725.573038 msecs"
nil
user=> (time (dotimes [_ 100000000] (get some-vec 0)))
"Elapsed time: 1064.079568 msecs"
nil
user=> (time (dotimes [_ 100000000] (some-vec 0)))
"Elapsed time: 208.812296 msecs"
nil
```

Boom. Right away you see the difference between three, seemingly similar
function calls. A couple notes here...

1. These numbers vary, your machine may get different results based on
resources available, your CPU, your JVM, and your version of clojure. The
important thing is to compare side-by-side in the same conditions.
2. I didn't start with 100000000 trials, I started with 100000 and kept adding
zeros until I was in the seconds. I find this is a good target.

From here, I usually ask myself "What is the shape of the difference?" That is,
is it a constant factor, is is say, one of them, O(n) and one of them O(1). To
test this, I use a bigger vector.

In this case, the results aren't that different. 

```clojure
user=> (def long-vec (vec (repeat 2000 1)))
#'user/long-vec
user=> (time (dotimes [_ 100000000] (first long-vec)))
"Elapsed time: 8116.026521 msecs"
nil
user=> (time (dotimes [_ 100000000] (get long-vec 0)))
"Elapsed time: 1647.245175 msecs"
nil
user=> (time (dotimes [_ 100000000] (long-vec 0)))
"Elapsed time: 683.53043 msecs"
nil
```

What's interesting here is that `first`, `get`, and the function call _have all
gotten longer_. However, even more interesting is that it's by _almost_ the same
amount (400-600ms). This suggests that each function call is not O(n). But, in
that case, why is it slower at all?

In clojure, operations on vectors that you would think is O(1) is actually
O(log_32(n)). And the overhead is being applied to the same portion of each run
time. It's easy and fun to demonstrate this.

```clojure
user=> (def vec1 (vec (repeat 32 1)))
#'user/vec1
user=> (def vec2 (vec (repeat 33 1)))
#'user/vec2
user=> (time (dotimes [_ 100000000] (vec1 0)))
"Elapsed time: 226.396047 msecs"
nil
user=> (time (dotimes [_ 100000000] (vec2 0)))
"Elapsed time: 478.947069 msecs"
nil
```

The next time it jumps seems to be `1056`, which is `32^2 + 32`.

```clojure
user=> (def vec3 (vec (repeat 1056 1)))
#'user/vec3
user=> (def vec4 (vec (repeat 1057 1)))
#'user/vec4
user=> (time (dotimes [_ 100000000] (vec3 0)))
"Elapsed time: 486.20501 msecs"
nil
user=> (time (dotimes [_ 100000000] (vec4 0)))
"Elapsed time: 712.855596 msecs"
nil
```

So, why O(log_32(n))? It's actually because clojure's persistent vectors aren't
really arrays or array-lists like they pretend they are, instead, they're chunks
of 32 elements stored in 32-array b-trees. This is all part of how they stay
persistent without requiring the big overhead of copying the entire vector every
time there's a change. If you're interested in understanding how that works,
[hyPiRion has a great three part series on them](http://hypirion.com/musings/understanding-persistent-vector-pt-1)
that I highly recommend!

So, next question, what is causing the constant factor difference in `first` and
`get` from the function call?

# Understanding what's under the hood

When trying to figure out what's happening, I immediately go to the source.
Luckily, clojures core library and java implementation are super readable. In
fact, [clojuredocs.org](https://clojuredocs.org/) always link to the source with
each function. Lets take a look.

### What happens when we call `first` on a vector?

Lets jump right into `clojure.core/first`. What do we see?

```clojure
;; https://github.com/clojure/clojure/blob/clojure-1.7.0/src/clj/clojure/core.clj#L1423-L1431
(def
 ^{:arglists '([coll])
   :doc "Returns the first item in the collection. Calls seq on its
    argument. If coll is nil, returns nil."
   :added "1.0"
   :static true}
 first (fn ^:static first [coll] (. clojure.lang.RT (first coll))))
```

For `first`, the answer is pretty clear, it's calling [`seq`](https://clojuredocs.org/clojure.core/seq).
on the collection. This is actually super common, and has caught me up more than
once.

If we dig into `clojure.lang.RT` we see what's actually being created. Follow
the `>` to see where we step into the next function.

```java
 // https://github.com/clojure/clojure/blob/clojure-1.7.0/src/jvm/clojure/lang/RT.java#L651-L658
 static public Object first(Object x){
     if(x instanceof ISeq)
         return ((ISeq) x).first();
>    ISeq seq = seq(x);
     if(seq == null)
         return null;
     return seq.first();
 }

 // https://github.com/clojure/clojure/blob/clojure-1.7.0/src/jvm/clojure/lang/RT.java#L503-L510
 static public ISeq seq(Object coll){
     if(coll instanceof ASeq)
         return (ASeq) coll;
     else if(coll instanceof LazySeq)
         return ((LazySeq) coll).seq();
     else
>        return seqFrom(coll);
 }
 
 static ISeq seqFrom(Object coll){
     if(coll instanceof Seqable)
         return ((Seqable) coll).seq();
     else if(coll == null)
         return null;
     else if(coll instanceof Iterable)
>        return chunkIteratorSeq(((Iterable) coll).iterator());
     else if(coll.getClass().isArray())
         return ArraySeq.createFromObject(coll);
     else if(coll instanceof CharSequence)
         return StringSeq.create((CharSequence) coll);
     else if(coll instanceof Map)
         return seq(((Map) coll).entrySet());
     else {
         Class c = coll.getClass();
         Class sc = c.getSuperclass();
         throw new IllegalArgumentException("Don't know how to create ISeq from: " + c.getName());
     }
}

 // https://github.com/clojure/clojure/blob/clojure-1.7.0/src/jvm/clojure/lang/RT.java#L487-L501
 private static final int CHUNK_SIZE = 32;
 public static ISeq chunkIteratorSeq(final Iterator iter){
     if(iter.hasNext()) {
         return new LazySeq(new AFn() {
             public Object invoke() {
                 Object[] arr = new Object[CHUNK_SIZE];
                 int n = 0;
                 while(iter.hasNext() && n < CHUNK_SIZE)
                     arr[n++] = iter.next();
                 return new ChunkedCons(new ArrayChunk(arr, 0, n), chunkIteratorSeq(iter));
             }
         });
     }
     return null;
 }
```

It was a bit of a walk, but what we see here is that the vector (which is an
instance of `Iterable` is ultimately turned into a lazy seq and walked over in
chunks of 32. We could go deeper, but the truth is, it's not necessary, here's
our answer, `first` requires creating a `seq` and a `seq` requires pulling out
32 elements. That's a huge freaking overhead.

However, since it's lazy, it's not an O(n) transformation, it will always be 32
elements. Hence, the overhead is constant.

### What happens when we call `get` on a vector?

Lets take a look at `clojure.core/get`.

```clojure
;; https://github.com/clojure/clojure/blob/clojure-1.7.0/src/clj/clojure/core.clj#L1423-L1431
(defn get
  "Returns the value mapped to key, not-found or nil if key not present."
  {:inline (fn  [m k & nf] `(. clojure.lang.RT (get ~m ~k ~@nf)))
   :inline-arities #{2 3}
   :added "1.0"}
  ([map key]
   (. clojure.lang.RT (get map key)))
  ([map key not-found]
   (. clojure.lang.RT (get map key not-found))))
```

```java
https://github.com/clojure/clojure/blob/clojure-1.7.0/src/jvm/clojure/lang/RT.java#L719-L723
static public Object get(Object coll, Object key){
	if(coll instanceof ILookup)
		return ((ILookup) coll).valAt(key);
	return getFrom(coll, key);
}
```

This takes us into the clojure implementation of the vector, `Vec`.

```clojure
 ;; https://github.com/clojure/clojure/blob/ae7acfeecda1e70cdba96bfa189b451ec999de2e/src/clj/clojure/gvec.clj#L273-L280
   clojure.lang.ILookup
   (valAt [this k not-found]
     (if (clojure.lang.Util/isInteger k)
       (let [i (int k)]
         (if (and (>= i 0) (< i cnt))
>          (.nth this i)
           not-found))
       not-found))
 
 ;; https://github.com/clojure/clojure/blob/ae7acfeecda1e70cdba96bfa189b451ec999de2e/src/clj/clojure/gvec.clj#L173-L176
   clojure.lang.Indexed
   (nth [this i]
     (let [a (.arrayFor this i)]
>      (.aget am a (bit-and i (int 0x1f)))))
 
 ;; https://github.com/clojure/clojure/blob/ae7acfeecda1e70cdba96bfa189b451ec999de2e/src/clj/clojure/gvec.clj#L306-L315
   (arrayFor [this i]
     (if (and  (<= (int 0) i) (< i cnt))
       (if (>= i (.tailoff this))
         tail
         (loop [node root level shift]
           (if (zero? level)
             (.arr node)
             (recur (aget ^objects (.arr node) (bit-and (bit-shift-right i level) (int 0x1f))) 
                    (- level (int 5))))))
       (throw (IndexOutOfBoundsException.))))
```

This is a little tricky to understand, but its pulling the element from the tree
structure that we mentioned earlier. Here's a break down...
1. Confirm `i` is greater than 0 and less than the count of the vector
2. Walk down the 32-child B-tree (0x1f == 31) and pull out the 32 element array
   that contains index `i`.
3. Return `i` mod 32 from the array.


### What happens when we call a vector as a function?

In order to call anything as a function, it must implement `clojure.lang.IFn`.
That means we go straight to the `Vec` deftype in `gvec.clj`. When we look, we
see that it quickly jumps to `nth`.

```clojure
 ;; https://github.com/clojure/clojure/blob/ae7acfeecda1e70cdba96bfa189b451ec999de2e/src/clj/clojure/gvec.clj#L284-L291
   clojure.lang.IFn
   (invoke [this k]
     (if (clojure.lang.Util/isInteger k)
       (let [i (int k)]
         (if (and (>= i 0) (< i cnt))
>          (.nth this i)
           (throw (IndexOutOfBoundsException.))))
       (throw (IllegalArgumentException. "Key must be integer"))))
```

This, of course, takes us right back to where we were when we used `get`. That
means, the only difference between calling the vector as a function and calling
`get`.

Now, if you look at `valAt` vs `invoke`, they're almost identical. The only
difference being the missing/error case. So, what makes `get` so much slower?
Simple, it's **reflection**. The `instanceof` call in the `clojure.lang.RT.get`
is our culprit.

# So, `nth` is the fastest?

Yes, and no. There is a function, `clojure.core/nth`, which you may be tempted
to call the victor, but the truth is, it's actually not the `nth` that `first`,
`get`, and `invoke` call. It's actually quite a bit slower than the function
call.

```clojure
user=> (time (dotimes [_ 100000000] (some-vec 0)))
"Elapsed time: 217.763043 msecs"
nil
user=> (time (dotimes [_ 100000000] (nth some-vec 0)))
"Elapsed time: 323.254749 msecs"
nil
```

You see, it too, does reflection to ensure that it's an `instanceof Indexed`
before calling `.nth` on it.

```clojure
;; https://github.com/clojure/clojure/blob/clojure-1.7.0/src/clj/clojure/core.clj#L854-L863
(defn nth
  "Returns the value at the index. get returns nil if index out of
  bounds, nth throws an exception unless not-found is supplied.  nth
  also works for strings, Java arrays, regex Matchers and Lists, and,
  in O(n) time, for sequences."
  {:inline (fn  [c i & nf] `(. clojure.lang.RT (nth ~c ~i ~@nf)))
   :inline-arities #{2 3}
   :added "1.0"}
  ([coll index] (. clojure.lang.RT (nth coll index)))
  ([coll index not-found] (. clojure.lang.RT (nth coll index not-found))))
```

```java
// https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/RT.java#L851-L855
static public Object nth(Object coll, int n){
	if(coll instanceof Indexed)
		return ((Indexed) coll).nth(n);
	return nthFrom(Util.ret1(coll, coll = null), n);
}
```

Now, you may be thinking, I'll just use `.nth` and call that `nth` definition on
the vector instance it's self. Unfortunately, if you call the method blindly,
then you'll walk yourself into an even worse case of reflection!

```clojure
user=> (time (dotimes [_ 100000000] (.nth some-vec 0)))
"Elapsed time: 500845.868323 msecs"
nil
```

If you really want the fastest way, then you can type-hint the vector. That
will get you the same performance as the other method definitions of the type
and bypass **all** reflection.

```clojure
user=> (time (dotimes [_ 100000000] (.nth ^clojure.lang.IPersistentVector some-vec 0)))
"Elapsed time: 207.94872 msecs"
nil
```

# Conclusion

Writing performant clojure is very possible, but it takes some thought. There's
this constant war in the language between top-level-namespace polymorphic
functions like `first` and `get` and the very real implications of reflection in
the language. Speeding up very tight loops requires constant vigilance,
constant testing, and constant understanding of whats going on under the hood.

Luckily, clojure makes those two things very easy. The repl is fantastic for
testing and timing snippets of code and the source is easy to understand.


_I was hoping to cover tools to identify performance issues as well, but didn't
get to it in this post. Hopefully I'll get around to covering some tequniques in
a part II!__
