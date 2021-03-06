Debugging
=========

.. function:: tap(value, label=None)

    Prints value and then returns it. Useful to tap into some functional pipeline for debugging::

        fields = (f for f in fields_for(category) if section in tap(tap(f).sections))
        # ... do something with fields

    If ``label`` is specified then it's printed before corresponding value::

        squares = {tap(x, 'x'): tap(x * x, 'x^2') for x in [3, 4]}
        # x: 3
        # x^2: 9
        # x: 4
        # x^2: 16
        # => {3: 9, 4: 16}


.. decorator:: log_calls(print_func, errors=True, stack=True)
               print_calls(errors=True, stack=True)

    Will log or print all function calls, including arguments, results and raised exceptions. Can be used as decorator or tapped into call expression::

        sorted_fields = sorted(fields, key=print_calls(lambda f: f.order))

    If ``errors`` is set to ``False`` then exceptions are not logged. This could be used to separate channels for normal and error logging::

        @log_calls(log.info, errors=False)
        @log_errors(log.exception)
        def some_suspicious_function(...):
            # ...
            return result

    :func:`print_calls` always prints everything, including error stack traces.


.. decorator:: log_enters(print_func)
               print_enters
               log_exits(print_func, errors=True, stack=True)
               print_exits(errors=True, stack=True)

    Will log or print every time execution enters or exits the function. Should be used same way as :func:`log_calls` and :func:`print_calls` when you need to track only one event per
    function call.


.. decorator:: log_errors(print_func, label=None, stack=True)
               print_errors(label=None, stack=True)

    Will log or print all function errors providing function arguments causing them. If ``stack``
    is set to ``False`` then each error is reported with simple one line message.

    Can be combined with :func:`silent` or :func:`ignore` to trace occasionally misbehaving function::

        @silent
        @log_errors(logging.warning)
        def guess_user_id(username):
            initial = first_guess(username)
            # ...

    Can also be used as context decorator::

        with print_errors('initialization', stack=False):
            load_this()
            load_that()
            # ...
        # SomeException: a bad thing raised in initialization


.. decorator:: log_durations(print_func, label=None)
               print_durations(label=None)

    Will time each function call and log or print its duration::

        @log_durations(logging.info)
        def do_hard_work(n):
            samples = range(n)
            # ...

        # 121 ms in do_hard_work(10)
        # 143 ms in do_hard_work(11)
        # ...

    A block of code could be timed with a help of context manager::

        with print_durations('Creating models'):
            Model.objects.create(...)
            # ...

        # 10.2 ms in Creating models
