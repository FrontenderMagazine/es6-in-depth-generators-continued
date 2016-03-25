# ES6 в деталях: Генераторы, продолжение

*[ES6 в деталях][1] — это цикл статей о новых возможностях языка программирования JavaScript, появившихся в 6 редакции стандарта ECMAScript, кратко — ES6.*

Добро пожаловать обратно в ES6 в деталях! Я надеюсь, время нашего летнего
перерыва вы провели так же весело, как и я. Но жизнь программиста не может
состоять из одних только феерверков и лимонада. Пришло время вернуться к тому,
где мы остановились. И у меня заготовлена тема, которая для этого идеально
походит.

Ещё в мае я писал про генераторы, новый вид функций, появившихся в ES6. Я назвал
их самой волшебной функциональностью ES6. Я говорил о том, как они могут
изменить будущее асинхронного программирования. И затем я написал это:

> О генераторах ещё можно многое рассказать… Но я считаю, что эта статья уже
> достаточно длинная, и из неё и так можно узнать много нового. Как и
> генераторы, мы пока приостановимся и закончим позднее.

Вот и пришло время.

[Вы можете найти первую часть этой статьи здесь.][2] Наверное, было бы лучше
сначала прочесть её перед тем, как приступать к этой. Давайте, это весело.
Она… немного длинная и мудрёная. Но там есть говорящий кот!

## Краткий обзор

В прошлый раз мы заострили внимание на базовом поведении генераторов. Оно
немного *странное*, наверное, но не сложное для понимания. Функция-генератор
во многом похожа на обычную функцию. Основное отличие в том, что тело
функции-генератора не выполняется всё сразу. Она работает понемногу раз за
разом, приостанавливая выполнение всякий раз, как достигнет выражения `yield`.

В [первой части][3] было подробное объяснение, но мы так и не рассмотрели ни
одного примера того, как все эти части взаимодействуют. Давайте этим и займёмся.

    function* someWords() {
      yield "hello";
      yield "world";
    }

    for (var word of someWords()) {
      alert(word);
    }

Скрипт довольно простой. Но если бы вы могли наблюдать всё то, что происходит,
как если бы различные куски кода были персонажами пьесы, *тот* сценарий был бы
совсем другим. Он мог бы выглядеть примерно так:

    СЦЕНА - ВНУТРЕННОСТИ КОМПЬЮТЕРА, ДЕНЬ

    ЦИКЛ FOR стоит в одиночестве на сцене, на голове каска, в руках
    доска с зажимом, очень деловитый.

                              ЦИКЛ FOR
                             (вызывает)
                            someWords()!

    Входит ГЕНЕРАТОР: высокий латунный механический джентльмен.
    Выглядит дружелюбно, но неподвижен будто статуя.

                              ЦИКЛ FOR
                           (потирая руки)
                          Итак, приступим!
                            (генератору)
                              .next()!

    ГЕНЕРАТОР оживает.

                             ГЕНЕРАТОР
                   {value: "hello", done: false}

    Генератор застывает в дурацкой позе.

                              ЦИКЛ FOR
                               alert!

    Сломя голову вбегает ALERT, с широко распахнутыми глазами,
    запыхавшийся. Складывается ощущение, что он такой всегда.

                              ЦИКЛ FOR
                      Скажи пользователю "hello".

    ALERT разворачивается и рывком убегает за сцену.

                               ALERT
                       (из-за кулис, кричит)
                           Всё остановить!
              Страница на hacks.mozilla.org сообщает:
                              "hello"!

    Пауза несколько секунд, затем ALERT мчится обратно, подбегает
    к ЦИКЛУ FOR и с юзом останавливается.

                               ALERT
                      Пользователь говорит, OK

                              ЦИКЛ FOR
                           (потирая руки)
                          Итак, приступим!
                    (поворачивается к ГЕНЕРАТОРУ)
                              .next()!

    ГЕНЕРАТОР снова оживает.

                             ГЕНЕРАТОР
                   {value: "world", done: false}

    Он застывает в другой дурацкой позе.

                              ЦИКЛ FOR
                               alert!

                               ALERT
                            (уже бежит)
                              Я мигом!
                       (из-за кулис, кричит)
                           Всё остановить!
              Страница на hacks.mozilla.org сообщает:
                              "world"!

    Снова пауза, затем ALERT плетётся обратно на сцену, неожиданно
    расстроенный.

                               ALERT
                 Пользователь снова говорит, OK, но...
                но пожалуйста, запрети этой странице
                     показывать диалоговые окна.

    Уходит с недовольным видом.

                              ЦИКЛ FOR
                           (потирая руки)
                          Итак, приступим!
                    (поворачивается к ГЕНЕРАТОРУ)
                              .next()!

    ГЕНЕРАТОР оживает в третий раз.

                              ГЕНЕРАТОР
                          (с достоинством)
                   {value: undefined, done: true}

    Его голова опускается на грудь, его глаза угасают.
    Он больше никогда не двинется.

                              ЦИКЛ FOR
                    Время для обеденного перерыва.

    Уходит.

    Через какое-то время входит СБОРЩИК МУСОРА, поднимает
    безжизненного ГЕНЕРАТОРА и уносит за кулисы.

Ну да, не то чтобы прямо *Гамлет*. Но вы поняли.

Как вы можете увидеть в этой пьесе, когда объект генератора впервые появляется,
он приостановлен. Он просыпается и выполняет небольшую часть себя каждый раз,
когда вызывается его метод `.next()`.

Это действие синхронное и однопоточное. Заметьте, в любой момент времени что-то
делает только один из персонажей. Персонажи никогда не прерывают и не
перекрикивают друг друга. Они говорят по очереди, и кто бы ни говорил, он может
продолжать столько, сколько хочет. (Точно как у Шекспира!)



And some version of this drama unfolds each time a generator is fed to a `for`
`of` loop. There is always this sequence of `.next()` method calls that do not
appear anywhere in your code. Here I’ve put it all onstage, but for you and your
programs, all this will happen behind the scenes, because generators and the
`for`–`of` loop were designed to work together, via [the iterator interface][4]

So to summarize everything up to this point:

*   Generator objects are polite brass robots that yield values.
*   Each robot’s programming consists of a single chunk of code: the body of
    the generator function that created it.
   

### How to shut down a generator

Generators have several fiddly extra features that I didn’t cover in part 1:

*   `generator.return()`
*   the optional argument to `generator.next()` 
*   `generator.throw(error)`
*   `yield*`

I skipped them mainly because without understanding *why* those features exist
, it’s hard to care about them, much less keep them all straight in your head. 
But as we think more about how our programs will use generators, we’ll see the 
reasons.

Here’s a pattern you’ve probably used at some point:


`
<pre>
function doThings() {
  setup();
  try {
    // ... do some things ...
  } finally {
    cleanup();
  }
}

doThings();
</pre>
`

The cleanup might involve closing connections or files, freeing system
resources, or just updating the DOM to turn off an “in progress” spinner. We 
want this to happen whether our work finishes successfully or not, so it goes in
a`finally` block.

How would this look in a generator?


`
<pre>
function* produceValues() {
  setup();
  try {
    // ... yield some values ...
  } finally {
    cleanup();
  }
}

for (var value of produceValues()) {
  work(value);
}
</pre>
`

This looks all right. But there is a subtle issue here: the call `work(value)`
isn’t inside the`try` block. If it throws an exception, what happens to our
cleanup step?

Or suppose the `for`–`of` loop contains a `break` or `return` statement. What
happens to the cleanup step then?

It executes anyway. ES6 has your back.

When we first discussed [iterators and the `for`–`of` loop][5], we said the
iterator interface contains an optional`.return()` method which the language
automatically calls whenever iteration exits before the iterator says it’s done.
Generators support this method. Calling`myGenerator.return()` causes the
generator to run any`finally` blocks and then exit, just as if the current 
`yield` point had been mysteriously transformed into a `return` statement.

Note that the `.return()` is not called automatically by the language in *all*
contexts, only in cases where the language uses the iteration protocol. So it is
possible for a generator to be garbage collected without ever running its
`finally` block.

How would this feature play out on stage? The generator is frozen in the middle
of a task that requires some setup, like building a skyscraper. Suddenly someone
throws an error! The`for` loop catches it and sets it aside. She tells the
generator to`.return()`. The generator calmly dismantles all its scaffolding
and shuts down. Then the`for` loop picks the error back up, and normal
exception handling continues.

### Generators in charge

So far, the conversations we’ve seen between a generator and its user have
been pretty one-sided. To break with the theater analogy for a second:


![(A fake screenshot of iPhone text messages between a generator and its user, with the user just saying 'next' repeatedly and the generator replying with values.)][6]

The user is in charge. The generator does its work on demand. But this isn’t
the only way to program with generators.

In part 1, I said that generators could be used for asynchronous programming.
Things you currently do with asynchronous callbacks or promise chaining could be
done with generators instead. You may have wondered how exactly that is supposed
to work. Why is the ability to yield (which after all is a generator’s only 
special power) sufficient? After all, asynchronous code doesn’t just yield. It*
makes stuff happen.* It calls for data from files and databases. It fires off
requests to servers. And then it returns to the event loop to wait for those 
asynchronous processes to finish. How exactly will generators do this? And 
without callbacks, how does the generator receive data from those files and 
databases and servers when it comes in?

To start working toward the answer, consider what would happen if we just had a
way for the`.next()` caller to pass a value back into the generator. With just
this one change, we could have a whole new kind of conversation:


![(A fake screenshot of iPhone text messages between a generator and its caller; each value the generator yields is an imperious demand, and the caller passes whatever the generator wants as an argument the next time it calls .next().)][7]

And a generator’s `.next()` method does in fact take an optional argument, and
the clever bit is that the argument then appears to the generator as the value 
returned by the`yield` expression. That is, `yield` isn’t a statement like 
`return`; it’s an expression that has a value, once the generator resumes.

    var results = yield getDataAndLatte(request.areaCode);
    

This does a lot of things for a single line of code:

*   It calls `getDataAndLatte()`. Let’s say that function returns the string 
    `"get me the database records for area code..."` that we saw in the
    screenshot.
   
*   It pauses the generator, yielding the string value.
*   At this point, any amount of time could pass.
*   Eventually, someone calls `.next({data: ..., coffee: ...})`. We store that
    object in the local variable
   `results` and continue on the next line of code.

To show that in context, here’s code for the entire conversation shown above:


`
<pre>
function* handle(request) {
  var results = yield getDataAndLatte(request.areaCode);
  results.coffee.drink();
  var target = mostUrgentRecord(results.data);
  yield updateStatus(target.id, "ready");
}
</pre>
`

Note how `yield` still just means exactly what it meant before: pause the
generator and pass a value back to the caller. But how things have changed! This
generator expects very specific supportive behavior from its caller. It seems to
expect the caller to act like an administrative assistant.

Ordinary functions are not usually like that. They tend to exist to serve their
caller’s needs. But generators are code you can have a conversation with, and 
that makes for a wider range of possible relationships between generators and 
their callers.

What might this administrative assistant generator-runner look like? It doesn
’t have to be all that complicated. It might look like this.


`
<pre>
function runGeneratorOnce(g, result) {
  var status = g.next(result);
  if (status.done) {
    return;  // phew!
  }

  // The generator has asked us to fetch something and
  // call it back when we're done.
  doAsynchronousWorkIncludingEspressoMachineOperations(
    status.value,
    (error, nextResult) => runGeneratorOnce(g, nextResult));
}
</pre>
`

To get the ball rolling, we would have to create a generator and run it once,
like this:

    runGeneratorOnce(handle(request), undefined);
    

In May, I mentioned `Q.async()` as an example of a library that treats
generators as asynchronous processes and automatically runs them as needed.
`runGeneratorOnce` is that sort of thing. In practice, generator will not yield
strings spelling out what they need the caller to do. They will probably yield 
Promise objects.

If you already understand promises, and now you understand generators, you
might want to try modifying`runGeneratorOnce` to support promises. It’s a
difficult exercise, but once you’re done, you’ll be able to write complex 
asynchronous algorithms using promises as straight-line code, not a`.then()` or
a callback in sight.

### How to blow up a generator

Did you notice how `runGeneratorOnce` handles errors? It ignores them!

Well, that’s not good. We would really like to report the error to the
generator somehow. And generators support this too: you can call
`generator.throw(error)` rather than `generator.next(result)`. This causes the
`yield` expression to throw. Like `.return()`, the generator will typically be
killed, but if the current yield point is in a`try` block, then `catch` and 
`finally` blocks are honored, so the generator may recover.

Modifying `runGeneratorOnce` to make sure `.throw()` gets called appropriately
is another great exercise. Keep in mind that exceptions thrown inside generators
are always propagated to the caller. So`generator.throw(error)` will throw 
`error` right back at you unless the generator catches it!

This completes the set of possibilities when a generator reaches a `yield`
expression and pauses:

*   Someone may call `generator.next(value)`. In this case, the generator
    resumes execution right where it left off.
   
*   Someone may call `generator.return()`, optionally passing a value. In this
    case, the generator does not resume whatever it was doing. It executes
   `finally` blocks only.
*   Someone may call `generator.throw(error)`. The generator behaves as if the
   `yield` expression were a call to a function that threw `error`.
*   Or, maybe nobody will do any of those things. The generator might stay
    frozen forever. (Yes, it is possible for a generator to enter a
   `try` block and simply *never* execute the `finally` block. A generator can
    even be reclaimed by the garbage collector while it’s in this state.
    )

This is not much more complicated than a plain old function call. Only 
`.return()` is really a new possibility.

In fact, `yield` has a lot in common with function calls. When you call a
function, you’re temporarily paused, right? The function you called is in 
control. It might return. It might throw. Or it might just loop forever.

### Generators working together

Let me show off one more feature. Suppose we write a simple generator-function
to concatenate two iterable objects:


`
<pre>
function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}
</pre>
`

ES6 provides a shorthand for this:


`
<pre>
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}
</pre>` 

A plain `yield` expression yields a single value; a `yield*` expression
consumes an entire iterator and yields*all* values.

The same syntax also solves another funny problem: the problem of how to call a
generator from within a generator. In ordinary functions, we can scoop up a 
bunch of code from one function and refactor it into a separate function, 
without changing behavior. Obviously we’ll want to refactor generators too. But 
we’ll need a way to call the factored-out subroutine and make sure that every 
value we were yielding before is still yielded, even though it’s a subroutine 
that’s producing those values now.`yield*` is the way to do that.


`
<pre>
function* factoredOutChunkOfCode() { ... }

function* refactoredFunction() {
  ...
  yield* factoredOutChunkOfCode();
  ...
}
</pre>
`

Think of one brass robot delegating subtasks to another. You can see how
important this idea is for writing large generator-based projects and keeping 
the code clean and organized, just as functions are crucial for organizing 
synchronous code.

### Exeunt

Well, that’s it for generators! I hope you enjoyed that as much as I did, too
. It’s good to be back.

Next week, we’ll talk about yet another mind-blowing feature that’s totally
new in ES6, a new kind of object so subtle, so tricky, that you may end up using
one without even knowing it’s there. Please join us next week for a look at ES6 
proxies in depth.<section class="about">

[More articles by Jason Orendorff…][8]</section></article>

 [1]: https://hacks.mozilla.org/category/es6-in-depth/

 [2]: https://hacks.mozilla.org/2015/05/es6-in-depth-generators/ "ES6 In Depth: Generators"
 [3]: https://hacks.mozilla.org/2015/05/es6-in-depth-generators/

 [4]: http://www.ecma-international.org/ecma-262/6.0/index.html#sec-iterator-interface

 [5]: https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/
 [6]: img/generator-messages-small-250x375.png
 [7]: img/generator-messages-2-small-250x375.png
 [8]: https://hacks.mozilla.org/author/jorendorffmozillacom/