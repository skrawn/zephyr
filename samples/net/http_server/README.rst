HTTP Server
###########

Overview
********

The HTTP Server sample application for Zephyr implements a basic TCP server
on top of the HTTP Parser Library that is able to receive HTTP 1.1 requests,
parse them and write back the responses.

This sample  code generates HTTP 1.1 responses dynamically
and does not serve content from a file system. The source code includes
examples on how to write HTTP 1.1 responses: 200 OK, 400 Bad Request,
403 Forbidden, 404 Not Found and soft HTTP errors like 200 OK with a 404
Not Found HTML message.

The source code for this sample application can be found at:
:file:`samples/net/http_server`.

Requirements
************

- Linux machine with wget and the screen terminal emulator
- Freedom Board (FRDM-K64F)
- LAN for testing purposes (Ethernet)

Building and Running
********************

Currently, the HTTP Server application is configured to listen at port 80,
This value is defined in the :file:`samples/net/http_server/src/config.h`
file:

.. code-block:: c

	#define ZEPHYR_PORT		80

Open the project configuration file for your platform, for example the
:file:`prj_frdm_k64f.conf` file is the configuration file for the
:ref:`frdm_k64f` board. For IPv4 networks, set the following variables:

.. code-block:: console

	CONFIG_NET_IPV4=y
	CONFIG_NET_IPV6=n

IPv6 is the preferred routing technology for this sample application,
if CONFIG_NET_IPV6=y is set, the value of CONFIG_NET_IPV4=y is ignored.

This sample code only supports static IP addresses that are defined in the
project configuration file:

.. code-block:: console

	CONFIG_NET_SAMPLES_MY_IPV6_ADDR="2001:db8::1"
	CONFIG_NET_SAMPLES_MY_IPV4_ADDR="192.168.1.101"

Adding URLs
===========

To define a new URL or to change how a URL is processed by the HTTP server,
open the :file:`samples/net/http_server/src/main.c` file and locate the
following lines:

.. code-block:: c

	http_url_default_handler(http_write_soft_404_not_found);
	http_url_add("/headers", HTTP_URL_STANDARD, http_write_header_fields);
	http_url_add("/index.html", HTTP_URL_STANDARD, http_write_it_works);

The first line defines how Zephyr will deal with unknown URLs. In this case,
it will respond with a soft HTTP 404 status code, i.e. an HTTP 200 OK status
code with a 404 Not Found HTML body.

The second line must be interpreted as follows: requests to /headers,
/headers/index.html and in general to /headers/xxx, will trigger the
http_write_header_fields routine that prints the received HTTP
Header Fields. In this case, "xxx" must be understood as any resource
under the /headers/ URL.

The third line will trigger a routine that prints an HTML It Works!
message when the /index.html or /index.html/xxx URLs are found.

To build this sample on your Linux host computer, open a terminal window,
locate the source code of this sample application and type:

.. code-block:: console

	make BOARD=frdm_k64f

The FRDM K64F board is detected as a USB storage device. The board
must be mounted (i.e. to /mnt) to 'flash' the binary:

.. code-block:: console

    $ cp outdir/frdm_k64f/zephyr.bin /mnt

On Linux, use the 'dmesg' program to find the right USB device for the
FRDM serial console. Assuming that this device is ttyACM0, open a
terminal window and type:

.. code-block:: console

    $ screen /dev/ttyACM0 115200

Once the binary is loaded into the FRDM board, press the RESET button.

Refer to the board documentation in Zephyr, :ref:`frdm_k64f`,
for more information about this board and how to access the FRDM
serial console under other operating systems.


Sample Output
=============

Assume that this HTTP server is configured to listen at 192.168.1.101 port 80.
On your Linux host computer, open a terminal window and type:

.. code-block:: console

	wget 192.168.1.101/index.html

wget will show:

.. code-block:: console

	--2017-01-17 00:37:44--  http://192.168.1.101/
	Connecting to 192.168.1.101:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: unspecified [text/html]
	Saving to: ‘index.html’

The HTML file generated by Zephyr and downloaded by wget is:

.. code-block:: html

	<html>
	<head>
	<title>Zephyr HTTP Server</title>
	</head>
	<body><h1><center>It Works!</center></h1></body>
	</html>

The screen application will display the following information:

.. code-block:: console

	[dev/eth_mcux] [INF] eth_0_init: Enabled 100M full-duplex mode.
	[dev/eth_mcux] [DBG] eth_0_init: MAC 00:04:9f:c9:29:6e
	Zephyr HTTP Server
	Address: 192.168.1.101, port: 80

	----------------------------------------------------
	[print_client_banner:42] Connection accepted
	Address: 192.168.1.10, port: 54327
	[http_ctx_get:268] Free ctx found, index: 0
	[http_write:59] net_nbuf_get_tx, rc: 0 <OK>
	[http_write:82] net_context_send: 0 <OK>
	[http_rx_tx:86] Connection closed by peer


To obtain the HTTP Header Fields web page, use the following command:

.. code-block:: console

	wget 192.168.1.101/headers -O index.html

wget will show:

.. code-block:: console

	--2017-01-19 22:09:55--  http://192.168.1.101/headers
	Connecting to 192.168.1.101:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: unspecified [text/html]
	Saving to: ‘index.html’

This is the HTML file generated by Zephyr and downloaded by wget:

.. code-block:: html

	<html>
	<head>
	<title>Zephyr HTTP Server</title>
	</head>
	<body>
	<h1>Zephyr HTTP server</h1>
	<h2>HTTP Header Fields</h2>
	<ul>
	<li>User-Agent: Wget/1.16 (linux-gnu)</li>
	<li>Accept: */*</li>
	<li>Host: 192.168.1.101</li>
	<li>Connection: Keep-Alive</li>
	</ul>
	<h2>HTTP Method: GET</h2>
	<h2>URL: /headers</h2>
	<h2>Server: arm</h2>
	</body>
	</html>

To test the 404 Not Found soft error, use the following command:

.. code-block:: console

	wget 192.168.1.101/not_found -O index.html

Zephyr will generate an HTTP response with the following header:

.. code-block:: console

	HTTP/1.1 200 OK
	Content-Type: text/html
	Transfer-Encoding: chunked

and this is the HTML message that wget will save:

.. code-block:: html

	<html>
	<head>
	<title>Zephyr HTTP Server</title>
	</head>
	<body><h1><center>404 Not Found</center></h1></body>
	</html>

Known Issues and Limitations
============================

- Currently, this sample application only generates HTTP responses in
  chunk transfer mode.
- Clients must close the connection to allow the HTTP server to release
  the network context and accept another connection.
