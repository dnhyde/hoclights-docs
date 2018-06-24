.. role:: bash(code)
   :language: bash

Implementation
==============

This chapter goes through hoclights implementation in details.


Which technlogies and why?
--------------------------

Hoclights went through a set of technological approaches. First it was implemented as a native mobile application using `React Native <https://facebook.github.io/react-native/>`_, then we attempted of building an Android Native Application written in Java. In the end we opted for a mobile web-app for one main reason: its portability. Indeed by creating a web-app we are OS-independant. iOS development without an account is limited to only 3 devices for testing purposes per year, and to our opinion it is a very severe limitation.


Frontend
~~~~~~~~

We discovered `Vue <https://vuejs.org/>`_, an open source javascript framework that makes code writing seamless and clear. It is so modular that maintaining the code base is almost a pleasure. For this reason we opted to develop the web-app in javascript, integrating Vue framework and following its best practices.

The style of our web-app is created using a *pure CSS* frameword named `Bulma <https://bulma.io/>`_. Bulma is a young framework based on flexbox. At the beginning its primitive may seem unfamiliar, but in very few time you can get used to them and leverage all Bulma's power.

A complete JSDoc style documentation of the frontend is availabe in the project repository.


Backend
~~~~~~~

Since we needed to host our web-app somewhere a backend was necessary. Instead of useing a cloud provider, we decided to flash raspian stretch on a raspberry pi 3 model B. The main reason for such a choice is that the rpi is advertized on the local network and has access to a very useful feature for hoclights: `the mDNS and DNSsd feature <./implementation.html#zeroconf-bonjour-mdns>`_.

We chose to implement our backend in Node.js to use a single language (javascript) for full-stack development. At the same time Node.js offers a rich package manager such as `npm` and has very useful plugins such as `Express <http://expressjs.com/>`_ that bootstrap server-side code implementation.
The full documentation for the backend can be found in the project repository.


Raspberry Pi Settings
---------------------

The rpi 3 Model B is equipped with:

	* Raspian Stretch v4.14 (2018-04-18)
	* Node.js v8.11.3 LTS
	* Nginx 1.10.3

Node act as middleware exposing two main endpoinds: :code:`/node/mdns` and :code:`/node/forward/:paramResponderIpAddr`. The former response is a list of responders that were discovered through the mdns search, the latter is a forwarder that takes into account to send the requests to the target responder based on the provided ip address as paramter.


**Why an endpoint to forward such requests?**

Short answer: `CORS <https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS>`_. Some responders (e.g. wemos) work at low level and should not bear the burden of http madness. For this reason we delegate the http traffic to our Node middleware.


**Why Nginx then?**

Node is listening on port :code:`3000` (or any custom port specified by the developer), anyway having Node to listen on port :80 is not so easy. The best approach is to use a `reverse proxy <https://www.nginx.com/resources/glossary/reverse-proxy-server/>`_. We configured Nginx to listen on port :code:`:80` and host our web-app.

Meaning that if you visit the rpi ip address (or its local dns - in our case raspberrylights.local) in the same local network you will see hoclights Vue frontend. i.e. if you go the root endpoint you see the web-app :code:`/`. For example by visiting :code:`192.168.0.2` or equivalently :code:`http://raspberrylights.local/`.
To reach our backend at the specified ip we had to specify a `static ip address <https://www.raspberrypi.org/learning/networking-lessons/rpi-static-ip-address/>`_ that is assigned to our rpi in the network. We preferred the static ip address to the local dns name because some browsers (chrome and firefox) may use an internal dns resolver instead of the OS one to resolve :code:`.local` domains.

To configure nginx we changed its configurtation under :bash:`/etc/nginx/sites-available/default`. Tested the config with :bash:`nginx -t` and reload nginx with :bash:`sudo service nginx reload`. `This repository <https://github.com/pdroll/Raspberry-Pi-Node-Server>`_ is quite old, but explains step by step most of the work to setup the rpi with node and nginx.


**Starting Node**

Once that our node backend is in place it can be started with the classical node application command: :bash:`node index.js`. To start node automatically we had to create a service. We created the hoclights service following `this guide <https://www.paulaikman.co.uk/nodejs-services-raspberrypi/>`_. Hoclights service is stored under :bash:`/lib/systemd/system`. Please notice that any change to the service file, :bash:`hoclights.service`, require a root restart of the daemon systemctl with :bash:`sudo systemctl daemon-reload`.

The Node backend is reachable from its dedicated endpoint: :code:`/node`


Backend Archtecture
-------------------

The backend has Nginx as entry point that acts as reverse proxy with two endpoints:
	* root :code:`/`: host the production static build of the web-app.
	* node :code:`/node`: sneds requests to the node.js middleware running behind the nginx reverse proxy. 

We could have hosted our Vue web-app directly on node server using `Express static function <http://expressjs.com/en/starter/static-files.html>`_. But for a much ordered architecture we decided to host our web-app on nginx and use ngnx as reverse proxy to avoid CORS. The backend architectural choice is reported in the figure below.

.. figure:: _img/hoclights-backend-architecture.png
    :align: center
    :alt: alternate text
    :figclass: align-center

    Hoclights backend implementation.

For development we duplicated an identical architecture on the dev machine. Using this approach we only need to change one single endpoint from dev to prod (i.e. :code:`192.168.0.2/node` to :code:`localhost:8081/node`). During development we have the Vue app running on :code:`localhost:8080` reverse proxied by nginx running on :code:`localhost:8081` and our node server runnning on :code:`localhost:3000` reverse proxied by nginx through :code:`localhost:8081/node`.

ZeroConf Bonjour mDNS
---------------------

Differently from the middleware (rpi 3 with Node) we cannot ask a static ip address for every responder. In case of many smart-objects controlled by many responders our system would not be scalable.
For this reason we implemented the multicast DNS discovery, a feature that allows devices to broadcast their dns information. On top of it, the dns-sd: dns-service_discovery, allows to advertise a specific service that is offered by the respnder. For example Google Chromecasts use this feature to "show" themselves to nearby smartphones.

In our particular case we used the following `npm package <https://www.npmjs.com/package/dnssd>`_.

.. warning:: the dnssd package works flawlwssly on rpi. However it is not able to finish the mdns discovery directly on OSX (probably because it requires to have mdns installed on the OS stack.)

.. tip:: There is a workaround the the above warning. The dnssd is able to perform the discovery if another device doing the mdns reqests is present on the same network.











