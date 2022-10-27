# Network Tracing in the (Chromium) Browser

Type `chrome://net-export/` or `edge://net-export/` in the address bar to get the UI for capturing network traffic. If you have a lot of tabs open you might want to start a incognito tab first.

All the information you need is in the UI but this is the workflow:

1. Pick one of the options using the radio buttons
1. Click "Start logging to disk" and chose where you want to save your file
1. Open a new tab and trigger the trafic you want to debug
1. Go back and click "Stop Logging"
1. Click link to view the trace in netlog_viewer

"Events"

Here's [a youtube video](https://www.youtube.com/watch?v=2RGdZbGgskk&ab_channel=EricLawrence) that shows how to use the feature.

Bonus TIL: Type `chrome://chrome-urls/` to get a list of all nifty tools that are built into Chrome.
