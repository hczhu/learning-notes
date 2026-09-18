- **Source**: Daniel Lemire, "A quick overview of atomics in C," *Daniel Lemire's blog*, 9 September 2026 (~11 min). <https://lemire.me/blog/2026/09/09/a-quick-overview-of-atomics-in-c/>
- **One-liner**: A full memory barrier is too strong, so the standard splits it into two **one-way doors** — **release** says *"if you see me, you see everything I did before me"*, **acquire** says *"everything I do after this really happens after what I just took"* — and they are meaningless alone: they only buy you anything **as a pair**, one publishing and one subscribing through the same atomic.
- ## 0. Getting a second thread at all
	- C is single-threaded by default; **extra cores do not help until you create threads**. With `<threads.h>` you pass a function to `thrd_create` and wait with `thrd_join`:
	  ```c
	  #include <threads.h>
	  #include <stdio.h>
	  int worker(void *arg) {
	      printf("hello from thread %d\n", *(int *)arg);
	      return 0;
	  }
	  int main(void) {
	      thrd_t t;
	      int id = 1;
	      thrd_create(&t, worker, &id);
	      thrd_join(t, NULL);
	  }
	  ```
	- **Portability warning**: C11 threads are an **optional** feature. If `__STDC_NO_THREADS__` is defined you do not have them. **Apple's C library has never shipped `<threads.h>`**, so the above does not compile on macOS, and **glibc only added it in 2.28 (2018)**. Fall back on POSIX threads (`pthread_create`, `pthread_join`).
- ## 1. Two separate problems, often confused
	- **Problem 1 — atomicity (garbage values).** If two threads touch the same **non-atomic** variable with no ordering between them and **at least one writes**, C calls that a **data race**: undefined behavior. An **atomic** integer "is never garbage" — you always read a value that was *once written*.
		- Note this is a **language** guarantee, not a machine observation. On most machines you use today, aligned 8/16/32/64-bit loads and stores already are atomic; the C language does not care, so **if you do not ask for atomicity you can still be broken by your compiler**.
	- **Problem 2 — ordering (reordering).** Atomicity does not buy you order. Write
	  ```
	  x = 2
	  y = 3
	  ```
	  and `y` may be set before `x`.
	- **Everything about release/acquire is Problem 2.** Making a variable atomic fixes the value; it does not fix *when* other threads see it relative to everything else.
- ## 2. First principles: why is reordering allowed at all?
	- The honest answer in the post: **"our processors are quite complex. They have layers of buffers and they can execute multiple instructions at once. They can issue several memory loads or stores at once."**
	- This is the same machine described in [[2025-09-processors-are-getting-wider]] — a wide superscalar core with several execution units, store buffers, and multiple memory operations in flight. **Reordering is not a bug or a cheat; it is what "wide" means.** Forbidding it would mean giving back exactly the performance the last twenty years of CPU design was spent buying.
	- The enabling rule is that **the hardware and compiler are allowed to cheat as long as you do not catch them**. In a single thread you cannot catch them: the observable result is identical. The moment a *second* thread watches your memory, it becomes an observer that can catch you — and suddenly the reordering is visible.
	- **So the memory model is not a description of the hardware. It is a contract about what you are allowed to observe.** You are buying back specific, narrow ordering guarantees, at a price, exactly where you need them.
	- The corollary that makes the whole design make sense: **ordering is expensive, so you should buy the least of it that makes your program correct.** The taxonomy below is a price list.
- ## 3. The price list: three points on one axis
	- | Model | What you get | Cost |
	  |---|---|---|
	  | **Sequentially consistent** (C's default for atomics) | "As if there is an **oracle** that watches all threads and comes up with a consistent story where everything is in order" | Expensive — the default precisely so that *correct* is the easy path |
	  | **Relaxed** | Values are not garbage, and a given atomic still has **one modification order** (if a counter goes 0→1 you will never see 1 then 0) — but **no ordering with respect to other memory** | Cheapest |
	  | **Release / acquire** | Ordering **only** at the points you name, **only** between the threads that pair up | Intermediate — and on x64, nearly free |
	- Note what **relaxed** still gives you: per-variable sanity. It is not "anything goes"; it is "this variable is coherent, and it tells you nothing about anything else."
- ## 4. The core intuition: cut a full barrier into two one-way doors
	- Start with the strict thing and notice it is **too strong**. A full barrier says: *everything before me really happens before me, and everything after me really happens after me.* ("Really happens" = visible effects.)
	- That forbids motion in **both directions** at **one point**. But in real code you almost never need both — you need one direction, at each of two different places. So **split it**:
	- **The three shapes, drawn:**
	  ```
	    A FULL BARRIER -- nothing crosses, either way (too strong)

	        write A
	        write B
	      ================ barrier ================
	        read  C
	        read  D
	              nothing sinks below, nothing rises above


	    RELEASE -- a one-way FLOOR.  Earlier work cannot sink below it.

	        write A      --+
	        write B      --+-- may NOT move down past the release
	      --- store(release) ----------------------  the PUBLISH point
	        ...              later work MAY move up above it


	    ACQUIRE -- a one-way CEILING.  Later work cannot rise above it.

	        ...              earlier work MAY move down below it
	      --- load(acquire) -----------------------  the SUBSCRIBE point
	        read C       --+
	        read D       --+-- may NOT move up past the acquire
	  ```
	- **Say them in words, and they stop being scary:**
		- **Release** = *"if you see me, you see all the stuff before me."* It is a **publication**. You are packaging up everything you did and stamping it onto one atomic write.
		- **Acquire** = *"I take that package, and everything I do after this load really happens after it."* It is a **subscription**. You are receiving the package and promising not to start unwrapping it early.
	- **Why each is one-way.** A release only has to keep your *earlier* work from escaping *downward* past the publish point — work that comes *after* the release is not part of the package, so it is free to float up. Symmetrically, an acquire only has to keep your *later* work from starting *before* you received the package. Each half pays for exactly one direction, which is why the pair is cheaper than the full barrier.
	- **They only work in pairs.** A release with nobody acquiring publishes to no one; an acquire with nobody releasing subscribes to nothing. The guarantee is always of the form *"thread B's acquire read the value written by thread A's release ⟹ everything A did before the release is visible to B after the acquire."* The atomic variable is the **channel**, not the payload.
	- **The canonical pairing — message passing:**
	  ```
	    thread 1 (producer)                  thread 2 (consumer)
	    -------------------                  -------------------
	    write payload                         .
	    store flag (RELEASE)  ===========>    load flag (ACQUIRE)
	                           reads that         .
	                            value             read payload  <-- guaranteed
	                                                                to see the writes
	  ```
- ## 5. Why one half is never enough — the refcount story
	- The post's motivating case, stripped to its logic: a shared resource with a reference count; whoever drops the count to zero frees it.
	- **Step 0 — the counter must be one indivisible read-modify-write.** A separate "load, then decrement" is broken *both* ways round, and it is worth seeing both:
		- Load then decrement: two threads both read 2, both subtract, the counter reaches 0 and **nobody frees** → **leak**.
		- Decrement then test: with refs at 2, one thread decrements to 1, the other to 0, both then read 0 and **both call free** → **double free**.
		- Fix: **one atomic subtract that hands you the previous value**. Only the thread that saw `1` was the last owner. (`atomic_fetch_sub` returns the old value for exactly this reason.)
	- **Step 1 — why the decrement needs a release.** Within one thread, `access resource; decrement counter` may be *reordered*, so the decrement can become visible first. Then:
	  ```
	    [thread2] access resource
	    [thread2] decrement counter
	    [thread1] decrement counter
	    [thread2] free(resource)
	    [thread1] access resource      <-- use after free
	  ```
	  Making the decrement a **release** forbids the access from sinking below it: *"I am done with the payload"* is now stamped onto the decrement itself.
	- **Step 2 — why release alone is still not enough, and this is the whole lesson.** Release protects *your own* earlier work. It says nothing about *other* threads' work. The last decrementer performs a release — **and a release does not observe anything.** So:
	  ```
	    [thread2] access resource
	    [thread2] decrement counter using RELEASE
	    [thread1] decrement counter using RELEASE
	    [thread1] free(resource)       <-- thread2's access may not be finished
	  ```
	  Thread 2 did its access before its release, but **thread 1 never acquired**, so it is not required to see that access as finished.
	- **Step 3 — the acquire closes the loop.** The last owner must *receive* everyone else's publications before destroying the object:
	  ```
	    access resource
	    decrement counter with RELEASE        // "I am done with the payload"
	    if (counter is zero)
	        acquire barrier                   // "I have seen that everyone else is done"
	        free(resource)
	  ```
	  Equivalently, a single decrement carrying **both** release and acquire. *The two are equivalent but not necessarily equally cheap* — and the split version is usually cheaper, because only the **last** owner pays for the acquire.
	- **The asymmetry to remember**: **every** thread releases (each publishes "I'm done"), but **only one** thread acquires (the one about to free needs to read everyone else's publications). That asymmetry is exactly why splitting the barrier in two was worth doing.
- ## 6. The worked example — a copy-on-write shared array
	- The payload is a **plain** `int` array; **only the reference count is atomic**. That is deliberate: *"we never write `values` while another thread might be reading them."* The atomic is the **permission slip**; the ordering annotations are what make the permission trustworthy.
	  ```c
	  #include <assert.h>
	  #include <stdatomic.h>
	  #include <stdlib.h>
	  #define STR_SIZE 16
	  typedef struct {
	      atomic_int refs;
	      int values[STR_SIZE];
	  } shared_array;
	  ```
	- **Construction.** The caller owns one reference.
	  ```c
	  shared_array *str_new(void) {
	      shared_array *o = malloc(sizeof *o);
	      if (o == NULL) {
	          return NULL;
	      }
	      atomic_init(&o->refs, 1);
	      for (int i = 0; i < STR_SIZE; i++) {
	          o->values[i] = 0;
	      }
	      return o;
	  }
	  ```
		- `atomic_init` is **not an atomic access in the memory-model sense** — nobody else has the pointer yet, so there is no thread to race with. It is just how you initialize an `atomic_int`.
	- ### Release, in three drafts
	- **Draft 1 — broken, because load-and-decrement is two operations:**
	  ```c
	  // not real code
	  void obj_release(shared_array *o) {
	      auto ref = o->refs;
	      o->refs -= 1;
	      if (ref != 1)
	          return;
	      // we are the last copy
	      free(o);
	  }
	  ```
		- Two threads can both read 2, both subtract, the counter hits zero, and **nobody frees** → leak. Write the test the other way round (decrement first, then check for zero) and you get the mirror image: from 2, one thread decrements to 1, the other to 0, **both read 0 and both call `free`** → double free.
		- The fix is **one atomic subtract that hands you the previous value**: only the thread that saw `1` was last.
	- **Draft 2 — atomic, but relaxed, so still broken:**
	  ```c
	  void obj_release(shared_array *o) {
	      if (atomic_fetch_sub_explicit(&o->refs, 1, memory_order_relaxed) != 1)
	          return;
	      free(o);
	  }
	  ```
		- With `refs == 2` and two threads each doing `(void)o->values[4]; obj_release(o);`, this interleaving is permitted:
		  ```
		  [thread1] o->values[4];
		  [thread2] atomic_fetch_sub_explicit(&o->refs, 1, memory_order_relaxed)
		  [thread1] atomic_fetch_sub_explicit(&o->refs, 1, memory_order_relaxed)
		  [thread1] free(o);
		  [thread2] o->values[4];          <-- use after free
		  ```
		- The confusing part is that thread 2's own two statements appear **out of order** — its decrement becomes visible before its read of `values[4]`. Relaxed permits exactly that.
	- **Draft 3 — correct:**
	  ```c
	  void obj_release(shared_array *o) {
	      if (atomic_fetch_sub_explicit(&o->refs, 1, memory_order_release) != 1)
	          return;
	      atomic_thread_fence(memory_order_acquire);
	      free(o);
	  }
	  ```
		- The **release on every decrement** means *"I am done with the payload."*
		- The **acquire fence, only on the last owner**, means *"I have seen that everyone else is done."* Then `free` is safe.
		- This is §5 in eight lines: everyone publishes, one thread subscribes.
	- ### Retain is relaxed, and that is not a shortcut
	  ```c
	  shared_array *str_retain(shared_array *o) {
	      atomic_fetch_add_explicit(&o->refs, 1, memory_order_relaxed);
	      return o;
	  }
	  ```
		- **Why relaxed is sound here**: the caller **already holds a reference**, so the object cannot be freed underneath us — the last owner would need *our* reference to be gone first. No ownership is changing hands, so no ordering needs to be published or received.
	- ### The copy-on-write update — where the acquire pays off
	  ```c
	  shared_array *update(size_t idx, int value, shared_array *o) {
	      assert(idx < STR_SIZE);
	      if (atomic_load_explicit(&o->refs, memory_order_acquire) == 1) {
	          o->values[idx] = value;
	          return o;
	      }
	      shared_array *new_o = str_new();
	      if (new_o == NULL) {
	          return NULL;
	      }
	      for (int i = 0; i < STR_SIZE; i++) {
	          new_o->values[i] = o->values[i];
	      }
	      new_o->values[idx] = value;
	      obj_release(o);
	      return new_o;
	  }
	  ```
		- **Contract**: it consumes the caller's reference and returns a reference to the array holding the new value — which **may or may not be the same object**, so after calling you must not touch the pointer you passed in. **One exception**: if a copy was needed and the allocation failed, it returns `NULL` and leaves your reference to `o` untouched, so you still own it and must still release it.
		- **Why the load is an acquire.** If it reads `1` we are the only owner, and that `1` may be **the value written by the release decrement of the last other owner to drop out**. The acquire makes everything that thread did with `values` **happen-before our write**, and stops `o->values[idx] = value` from being moved above the load. No other thread holds a reference, so the in-place write races with nobody. Later, after a `retain`, other threads can see it.
		- **This is why the release does double duty**: the same release that makes `free` safe is also what makes a later **in-place write** safe.
	- ### The whole thing as a table
	- | Operation | Order used | Why |
	  |---|---|---|
	  | `atomic_fetch_sub_explicit(&refs, 1, ...)` in release | **release** | Publishes "everything I did with the payload happened before this" |
	  | `atomic_thread_fence(...)` before `free` | **acquire** | Only on the last owner: "I have seen that everyone else is done" |
	  | `atomic_fetch_add_explicit(&refs, 1, ...)` in retain | **relaxed** | Caller already holds a reference; no ownership transfer, so nothing to order |
	  | `atomic_load_explicit(&refs, ...) == 1` in update | **acquire** | Receives the last other owner's release, making the in-place write safe |
	  | `atomic_init(&o->refs, 1)` | *(not an access)* | No other thread has the pointer yet |
- ## 7. What it costs, and who actually reorders
	- **On x64, acquire and release are effectively free at the CPU**: ordinary loads already behave like acquire, ordinary stores like release.
	- **You must still write them in C anyway** — otherwise **the compiler** may reorder the payload accesses. This is the part that surprises people: on strong hardware the annotations often generate no extra instructions, yet omitting them is still a real bug, because the compiler is the reorderer you forgot about.
	- **ARM has a weaker memory model**, so acquire/release need different instructions (`ldapr`, `ldaddl`) and may incur a small performance hit.
	- Practical framing: the *hardware* cost varies by ISA; the *correctness* requirement does not.
- ## 8. Takeaways
	- **Atomicity and ordering are two different purchases.** `atomic_int` buys you "never garbage." Memory orders buy you "and here is when others see it."
	- **The whole design is a price list, and the default is the expensive one.** Sequential consistency is the default because correctness should be the easy path; relaxed and acquire/release are opt-in discounts you take where you have proved you can.
	- **Release/acquire is a full barrier cut in half by direction**, so each site pays for the one direction it needs. Release = publish (nothing earlier escapes downward). Acquire = subscribe (nothing later starts early).
	- **The pair is the unit of meaning.** Ask of any memory order not "is this strong enough?" but "**who is on the other end of this?**" An unpaired release or acquire is almost always a bug or a waste.
	- **Release protects your past; acquire inspects everyone else's.** That is why the refcount needs both: every dropper publishes, and only the destroyer must read all those publications before acting.
	- **Reordering exists because processors got wide** ([[2025-09-processors-are-getting-wider]]). The memory model is the negotiated settlement between that hardware freedom and the programmer's need to occasionally pin things down — which is also why the guarantees are stated as *what you may observe*, never as *what the machine does*.
	- Related: [[2025-09-processors-are-getting-wider]], [[2026-09-pmpp-ch1-cpu-vs-gpu-design-philosophies]]
