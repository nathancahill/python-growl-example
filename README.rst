==========================
Using Growl with apiweb.io
==========================

In search for an apartment in Boulder, I needed a way to be notified instantly
when new places are listed on Craigslist. Growl is the obvious choice on my Mac, 
and apiweb.io provides an easy to use API for Craigslist.

Requires growl and requests. Install them with pip:

.. code-block:: bash

    $ pip install requests
    $ pip install growl

Here's the script. 

.. code-block:: python

    import time
    import requests
    from growl import Registration, Notification, GROWL_UDP_PORT
    from socket import socket, AF_INET, SOCK_DGRAM

    prev_id = ''
    addr = ('localhost', GROWL_UDP_PORT)
    s = socket(AF_INET, SOCK_DGRAM)

    r = Registration('Craigslist Watcher')
    s.sendto(r.payload(), addr)

    while True:
        r = requests.get('http://apiweb.io/api/nathancahill/craigslist/boulder/apa')
        j = r.json()

        if not prev_id == j['results'][0]['id']:
            title = 'New apartment in ' + j['results'][0]['location']
            description = j['results'][0]['price'] + ': ' + j['results'][0]['title']

            n = Notification('Craigslist Watcher', 'Found Results', title, description)
            s.sendto(n.payload(), addr)

            prev_id = j['results'][0]['id']

        time.sleep(60)


How to use it
=============

You can change the apiweb.io URL to any Craigslist city, and the "apa" part to 
any section of craigslist.

On your first use, you need to send the Registration packet. On subsequent uses,
you can comment those lines out.
