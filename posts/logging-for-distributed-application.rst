.. title: Centralized logging for distributed applications with pyzmq
.. slug: logging-for-distributed-application
.. date: 2013/08/14 01:13:00
.. tags: python
.. link: 
.. description: 
.. type: text


Simpler distributed applications can take advantage of centralized logging. 
PyZMQ, a Python bindings for ØMQ provides log handlers for the python logging module and can be easily used for this purpose. 
Log handlers utilizes ØMQ Pub/Sub pattern and broadcasts log messages through a PUB socket. 
It is quite easy to construct the message collector and write messages to a central location. 

::

	+-------------+
	|Machine1:App1+-------------------------
	+-------------+                        |
	                                +---------------+
	+-------------+.................|Machine3:Logger|
	|Machine1:App2|                 +---------------+
	+-------------+                       |
	                                      |
	       +-------------+                |
	       |Machine2:App1|-----------------
	       +-------------+

*Client Application*

To start with, we will need pyzmq library and support for logging library. 

client logger: usual imports

.. code-block:: python

	import logging
	import random
	import time
	import zmq
	from zmq.log.handlers import PUBHandler

Useful format that identifies where the logs are emanating from. 

.. code-block:: python

	LOG_LEVELS = (logging.DEBUG, logging.INFO, logging.WARN, logging.ERROR, logging.CRITICAL)
	 
	formatters = {
	        logging.DEBUG: logging.Formatter("%(filename)s:%(lineno)d | %(message)s\n"),
	        logging.INFO: logging.Formatter("%(filename)s:%(lineno)d | %(message)s\n"),
	        logging.WARN: logging.Formatter("%(filename)s:%(lineno)d | %(message)s\n"),
	        logging.ERROR: logging.Formatter("%(filename)s:%(lineno)d | %(message)s\n"),
	        logging.CRITICAL: logging.Formatter("%(filename)s:%(lineno)d | %(message)s\n")
	        }
	interval = 1
	port = 5558


And finally the log handler that allows publication of messages over a PUB zmq socket. 

.. code-block:: python

	ctx = zmq.Context()
	pub = ctx.socket(zmq.PUB)
	pub.connect('tcp://127.0.0.1:%i' % port)
	logger = logging.getLogger("clientapp1")
	logger.setLevel(level)
	handler = PUBHandler(pub)
	handler.formatters = formatters
	logger.addHandler(handler)
	while True:
	        level = random.choice(LOG_LEVELS)
	        logger.log(level, "subtopic.subsub::Hello from %i" % os.getpid())
	        time.sleep(interval)


You may have also notice the use of specific style of message that helps you provide a specific subtopic which is useful for logging structure. 
Finally, we will implement the centralized logger.

*Centralized logger* with usual imports and parameters.

.. code-block:: python

	import zmq.green as zmq
	import logging
	import logging.handlers
	 
	LOG_LEVELS = {'DEBUG': logging.DEBUG,
	              'INFO': logging.INFO,
	              'WARN': logging.WARN,
	              'ERROR': logging.ERROR,
	              'CRITICAL': logging.CRITICAL
	              }
	port = 5558

The centralized logger implements the SUB pattern (of PUB/SUB) to subscribe to published messages and log the messages to a file. The published messages could emanate from different applications on different machines and provides for centralized logging. 

.. code-block:: python

	logger = logging.getLogger()
	context = zmq.Context(context)
	socket_fd = context.socket(zmq.SUB)
	socket_fd.bind("tcp://localhost:%s" % port)
	socket_fd.setsockopt(zmq.SUBSCRIBE, "")
	filehandler = logging.handlers.TimedRotatingFileHandler('log file', 'midnight',1)
	logger.setLevel(logging.DEBUG)
	filehandler.setLevel(logging.DEBUG)
	formatter = logging.Formatter('%(asctime)s | %(levelname)s | %(message)s')
	filehandler.setFormatter(formatter)
	logger.addHandler(filehandler)
	while True:
	        topic, message = socket_fd.recv_multipart()
	        pos = topic.find('.')
	        level = topic
	        if pos > 0: level = topic[:pos]
	        if message.endswith('\n'): message = message[:-1]
	        log_msg = getattr(logging, level.lower())
	        if pos > 0: message = topic[pos+1:] + " | " + message
	        log_msg(message)



