Evaluation
==========

We evaluate hoclights requests mostly to play with datavisualization libraries.

Data format
-----------

With hoclights we can gather only basic data (we do not track which user changes the lights for now). The data that we store have the following format:

=========  =========  ===========  ================
timestamp  responder  smartobject  value
=========  =========  ===========  ================
123456789  wemos_123  lamp1  	   {"brightness":0}
123456789  wemos_123  lamp1  	   {"brightness":0}
=========  =========  ===========  ================
