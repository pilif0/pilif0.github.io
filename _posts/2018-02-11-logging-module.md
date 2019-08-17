---
layout: post
title: Logging Module
categories: [Open Sea]
tags: [C++, logging, Boost.Log]
date: 2018-02-11 15:43
---
First part of Open Sea is the logging module, which is a layer of abstraction over the [Boost.Log](http://www.boost.org/doc/libs/1_66_0/libs/log/doc/html/index.html) library.
It sets up easy logging into a file with a fallback being the console.
The whole module is contained in the `open_sea::log` namespace.

__Update:__ The relevant pull request is available [here](https://github.com/pilif0/open-sea/pull/1).

To initialize logging, all you need to do is call the `init_logging` function which sets up the file sink with proper formatting, filtering and attributes.
If the file sink initialization fails for some reason (e.g. target file exists but do not have write permissions), the console is used instead.
You can also add the console sink manually by calling the `add_console_sink` function.

To log an actual message, you require a logger object.
This can be retrieved using the `get_logger` functions.
You can either provide it a name of the module (to easily identify where the log entry is from), or not provide anything and then “No Module” will be used instead.
The intention is to use one logger for the whole scope of a function or for a class.
To log an actual message, you call the function `log`, providing it with the logger to use, severity level of the message, and the actual message.
Severity levels are taken from the `severity_level` enum in the same namespace and they are declared in ascending order of severity (relevant to filtering).
By default, only messages with severity warning or higher will be logged into the file.
If `OPEN_SEA_DEBUG` is defined, all messages are logged into the file.

The aim of this module is to make logging messages as effortless as possible while logging as much information as possible.
This should result in easier debugging of later additions and changes to the project.

One other addition that piggybacked on logging is [Doxygen](http://www.stack.nl/~dimitri/doxygen/) documentation.
A doc target has been added to the CMake configuration that builds the documentation.
All documentation specific code (so far only the relevant `CMakeLists.txt`) is in the `doc` folder.
