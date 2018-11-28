python-ELM
==========

A python emulator for the ELM327 OBD-II adapter. Built for testing [python-OBD](https://github.com/brendanwhitfield/python-OBD).

This emulator can process basic examples in [*python-OBD* documentation](https://python-obd.readthedocs.io/en/latest/) and reproduces a limited message flow
generated by a Toyota Auris Hybrid car, including some custom messages.

Run with:

```shell
python -m elm
```

Tested with Python 3.7. Python 2 is not supported.

The serial port to be used by the application interfacing the emulator is displayed in the first output line. E.g.,:

    Running on /dev/pts/1

A [dictionary](https://docs.python.org/3.7/tutorial/datastructures.html#dictionaries) is used to define commands and pids. The dictionary includes more sections (named scenarios):

- `'AT'`: supported AT commands
- `'default'`: supported default pids
- `'test'`: different values for some of the default pids
- any additional custom section can be used to define specific scenarios

Default settings include both the 'AT' and the 'default' scenarios.

The dictionary used to parse each ELM command is dynamically built as a union of three defined scenarios in the following order: 'default', 'AT', custom scenario (when applied). Each subsequent scenario redefines commands of the previous scenarios. In principle, 'AT scenario is added to 'default' and, if a custom scenario is used, this is also added on top, and all equal keys are replaced. Then the Priority key defines the precedence to match elements.

If `emulator.scenario` is set to a string different from *default*, the custom scenario set by the string is applied; any key defined in the custom scenario replaces the default settings ('AT' and 'default' scenarios).

Allowed keys to be used in the dictionary:

- `'Pid'`: short pid description (also used by the `emulator.answer` dictionary)
- `'Descr'`: string describing the pid
- `'Exec'`: command to be executed
- `'Log'`: *logging.debug* argument
- `'ResponseFooter'`: run a function and returns a footer to the response (a lambda function can be used)
- `'ResponseHeader'`: run a function and returns a header to the response (a lambda function can be used)
- `'Response'`: returned data
- `'Action'`: can be set to 'skip' in order to skip the processing of the pid
- `'Header'`: not used
- `'Priority'=number`: when set, the key has higher priority than the default (highest number = 1, lowest = 10 = default)

The emulator provides a monitoring front-end, supporting commands. The monitoring front-end controls the backend thread executing the actual process.

At the prompt '*CMD> *', the emulator accepts the following commands:

- `q` = quit
- `c` = print the number of each executed PID (upper case names), the values associated to some 'AT' PIDs (*cmd_...*), the unknown requests, the emulator response delay, the total number of executed commands (*commands*) and the current scenario (*scenario*)
- `p` = pause
- `i` = toggle prompt off/on
- `r` = resume
- `t <n>` = delay each emulator response of `<n>` seconds (floating point number; default is 0.5 seconds)
- `w <n>` = delay the execution of the next command of `<n>` seconds (floating point number; default is 10 seconds)
- `o` = switch to 'engineoff' scenario
- `s <scenario>` = switch to `<scenario>` scenario; if the scenario is missing or invalid, defaults to `'test'`
- `d` = reset to 'default' scenario
- `reset` = reset the emulator (counters and variables)
- any other Python command can be used to query/configure the backend thread

The command prompt also allows configuring the `emulator.answer` dictionary, which has the goal to force answers for specific Pids (`'Pid': '...'`). Its syntax is:

```
emulator.answer = { 'pid' : 'answer', 'pid' : 'answer', ... }
```

Example:

```
emulator.answer = { 'SPEED': 'NO DATA\r', 'RPM': 'NO DATA\r' }
```

The above example forces SPEED and RPM pids to always return "NO DATA".

To reset the *emulator.answer* string to its default value:

```
emulator.answer = {}
```

The dictionary can be used to build a workflow.

The front-end can also be controlled by an external piped automator.

Logging is controlled through `elm.yaml` (in the current directory by default). Its path can be set through the *ELM_LOG_CFG* environment variable.

The logging level can be dynamically changed by referencing `emulator.logger`. For instance, if the logging configuration has *stdout* as the first handler, the following commands will change the logging level:

```
emulator.logger.handlers[0].setLevel(logging.DEBUG)
emulator.logger.handlers[0].setLevel(logging.INFO)
emulator.logger.handlers[0].setLevel(logging.ERROR)
```
