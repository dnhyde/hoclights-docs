Concept
=======

Hoclights is a web-app to control smart lights from in the same local netowrk. We chose to enforce a "same network" policy in order to mitigate security risks of an endpoint exposed through the internet.

The smart lights are tied to a specific location: our laboratory. For this reason for now there is no need to open a remote control. In the future, having a remote endpoint can be useful for monitoring the lights status, but such a system exceeds the purpose of our web-app

.. _terminology-ref:

Terminology
-----------

For the sake of clarity we use the following terms to describe entities that come into play in the hoclights ecosystem (*LOW LEVEL --> HIGH LEVEL*):

	* **smart-object** (*LOWEST*): any physical thing that can controlled by the hoclights system. For example a dimmable light.
	* **responder** (*LOW*): a device that controls one or more smart-objects. Its role is to translate hoclights system commands into low-level signals to trigger some behaviour in the smart-object. For example a wemos board that controls some dimmable lights or leds.
	* **middleware** (*MIDDLE*): a device that has the role to forward/transform/translate the requests coming from higher level devices into understandable requests for the responders. For example forward an http request from the smartphone to the wemos. 
	* **web-app** (*HIGH*): the frontend that users employ to send command to the smart-objects. The commands are sent under the form of http requests. They can be send to a responder directly or to a middleware.


Requirements
------------

Hoclights must:

	* be accessible to all devices under a predefined network;
	* be accessible from different devices (e.g. smartphones, tablets, computers);
	* communicate with a middleware that controls the smart objects;
	* respond to user inputs in real time.
