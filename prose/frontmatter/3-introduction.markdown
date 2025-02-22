## Introduction {-}

Type-level programming is an uncommon calling. While most programmers are
concerned with getting more of their code to compile, we type-level programmers
are trying our best to *prevent* code from compiling.

Strictly speaking, the job of types is twinfold---they prevent (wrong) things
from compiling, and in doing so, they help guide us towards more elegant
solutions. For example, if there are ten solutions to a problem, and nine of
them be poorly-typed, then we need not look very hard for the right answer.

But make no mistake---this book is primarily about reducing the circumstances
under which a program compiles. If you're a beginner Haskell programmer who
feels like GHC argues with you too often, who often finds type errors
inscrutable, then this book is probably not for you. Not yet.

So whom *is* this book for? The target audience is people who are
intermediate-to-proficient with the language. They're capable of solving real
problems in Haskell, and doing it without too much hassle. They need not have
strong opinions on `ExceptT` vs throwing exceptions in `IO`, nor do they need to
know how to inspect generated Core to find performance bottlenecks.

But the target reader should have a healthy sense of unease about the programs
they write. They should look at their comments saying "don't call this function
with $n=5$ because it will crash," and wonder if there's some way to teach the
compiler about that. The reader should nervously eyeball their calls to `error`
that they're convinced can't possibly happen, but are required to make the
type-checker happy.

In short, the reader should be looking for opportunities to make *less* code
compile. This is not out of a sense of masochism, anarchy, or any such thing.
Rather, this desire comes from a place of benevolence---a little frustration
with the type-checker now is preferable to a hard-to-find bug making its way
into production.

Type-level programming, like anything, is best in moderation. It comes with its
own costs in terms of complexity, and as such should be wielded with care. While
it's pretty crucial that your financial application handling billions of dollars
a day runs smoothly, it's a little less critical if your hobbyist video game
draws a single frame of gameplay incorrectly. In the first case, it's probably
worthwhile to use whatever tools you have in order to prevent things from going
wrong. In the second, these techniques are likely too heavy-handed.

Style is a notoriously difficult thing to teach---in a very real sense, style
seems to be what's left after we've extracted from a subject all of the things
we know how to teach. Unfortunately, when to use type-level programming is
largely a matter of style. It's easy to take the ball and run with it, but
discretion is divine.

When in doubt, err on the side of *not* doing it at the type-level. Save these
techniques for the cases where it'd be catastrophic to get things wrong, for the
cases where a little type-level stuff goes a long way, and for the cases where
it will drastically improve the API. If your use-case isn't obviously one of
these, it's a good bet that there is a cleaner and easier means of doing it with
values.

But let's talk more about types themselves.

As a group, I think it's fair to say that Haskellers are contrarians. Most of
us, I'd suspect, have spent at least one evening trying to extol the virtues of
a strong type system to a dynamically typed colleague. They'll say things along
the lines of "I like Ruby because the types don't get in my way." Though our
first instinct, as proponents of strongly typed systems, might be to forcibly
connect our head to the table, I think this is a criticism worth keeping in
mind.

As Haskellers, we certainly have strong opinions about the value of types. They
*are* useful, and they *do* carry their weight in gold when coding, debugging
and refactoring. While we can dismiss our colleague's complaints with a wave of
the hand and the justification that they've never seen a "real" type system
before, we are doing them and ourselves both a disservice. Such a flippant
response is to ignore the spirit of their unhappiness---types *often do* get in
the way.  We've just learned to blind ourselves to these shortcomings, rather
than to bite the bullet and entertain that maybe types aren't always the
solution to every problem.

But there are plenty of error-free programs that are ruled out by a type system.
Consider, for example, the following program which has a type-error, but never
actually evaluates it:

```haskell
fst ("no problems", True <> 17)
```

Because the type error gets ignored lazily by `fst`, evaluation of such an
expression will happily produce `"no problems"` at runtime. Despite the fact
that we consider it to be ill-typed, it is in fact, well-behaved. The usefulness
of such an example is admittedly low, but the point stands; types often do get
in the way of perfectly reasonable programs.

Sometimes such an obstruction comes under the guise of "it's not clear what type
this thing should have." One particularly poignant case of this is C's `printf`
function:

```c
int printf (const char *format, ...)
```

If you've never before had the pleasure of using `printf`, it works like this:
it parses the `format` parameter, and uses its structure to pop additional
arguments off of the call-stack. You see, it's the shape of `format` that
decides what parameters should fill in the `...` above.

For example, the format string `"hello %s"` takes an additional string and
interpolates it in place of the `%s`. Likewise, the specifier `%d` describes
interpolation of a signed decimal integer.

The following calls to `printf` are all valid:

* `printf("hello %s", "world")`, producing "hello world",
* `printf("%d + %d = %s", 1, 2, "three")`, producing "1 + 2 = three",
* `printf("no specifiers")`, producing "no specifiers".

Notice that, as written, it seems impossible to assign a Haskell-esque type
signature to `printf`. The additional parameters denoted by its ellipsis are
given types by the value of its first parameter---a string. Such a pattern is
common in dynamically-typed languages, and in the case of `printf`, it's
inarguably useful.

The documentation for `printf` is quick to mention that the format string must
not be provided by the user---doing so opens up vulnerabilities in which an
attacker can corrupt memory and gain access to the system. Indeed, this is
hugely widespread problem---and crafting such a string is often the first
homework in any university lecture on software security.

To be clear, the vulnerabilities in `printf` occur when the format string's
specifiers do not align with the additional arguments given. The following,
innocuous-looking calls to `printf` are both malicious.

* `printf("%d")`, which will probably corrupt the stack,
* `printf("%s", 1)`, which will read an arbitrary amount of memory.

C's type system is insufficiently expressive to describe `printf`. But because
`printf` is such a useful function, this is not a persuasive-enough reason to
exclude it from the language. Thus, type-checking is effectively turned off for
calls to `printf` so as to have ones cake and eat it too.  However, this opens a
hole through which type errors can make it all the way to runtime---in the form
of undefined behavior and security issues.

My opinion is that preventing security holes is a much more important aspect of
the types, over "`null` is the billion dollar mistake" or whichever other
arguments are in vogue today. We will return to the problem of `printf` in
chapter 9.

With very few exceptions, the prevalent attitude of Haskellers has been to
dismiss the usefulness of ill-typed programs. The alternative is an
uncomfortable truth: that our favorite language can't do something useful that
other languages can.

But all is not lost. Indeed, Haskell *is* capable of expressing things as
oddly-typed as `printf`, for those of us willing to put in the effort to learn
how. This book is *the* comprehensive manual for getting you from here to there:
from a competent Haskell programmer to one who convinces the compiler to do
their work for them.

