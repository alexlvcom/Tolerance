*Behavioural* runners
=====================

These operation runners decorates an existing one to add extra *behaviour*:

- The `retry runner`_ will retry the operation until it is successful or considered as failing too much.
- The `buffered runner`_ will buffer operations until you decide the run them.

.. note::

    The `Throttling component <../throttling/intro.html>`_ also come with a `Rate Limited Operation Runner <../throttling/integrations.html#operation-runner>`_


Retry runner
------------

This runner will retry to run the operation until it is successful or the wait strategy decide to fail.

.. code-block:: php

    use Tolerance\Operation\Runner\CallbackOperationRunner;
    use Tolerance\Operation\Runner\RetryOperationRunner;
    use Tolerance\Waiter\Waiter\SleepWaiter;
    use Tolerance\Waiter\Waiter\ExponentialBackOff;

    // Creates the strategy used to wait between failing calls
    $waitStrategy = new CountLimited(
        new ExponentialBackOff(
            new SleepWaiter(),
            1
        ),
        10
    );

    // Creates the runner
    $runner = new RetryOperationRunner(
        new CallbackOperationRunner(),
        $waitStrategy
    );

    $result = $runner->run($operation);


Buffered runner
---------------

This runner will buffer all the operations to post-pone their execution.

.. code-block:: php

    use Tolerance\Operation\Buffer\InMemoryOperationBuffer;
    use Tolerance\Operation\Runner\BufferedOperationRunner;

    $buffer = new InMemoryOperationBuffer();
    $bufferedRunner = new BufferedOperationRunner($runner, $buffer);

    // These 2 operations will be buffered
    $bufferedRunner->run($firstOperation);
    $bufferedRunner->run($secondOperation);

Once you've decided that you want to run all the operations, you need to call the :code:`runBufferedOperations` method.

.. code-block:: php

    $results = $bufferedRunner->runBufferedOperations();

The :code:`$results` variable will be an array containing the result of each ran operation.

.. tip::

    The Symfony Bridge automatically run all the buffered operations when the kernel terminates. Checkout the
    `Symfony Bridge documentation <../bridges/symfony-bundle/intro.html>`_
