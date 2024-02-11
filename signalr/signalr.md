# SignalR

SignalR solves the problem of real time updates (i.e. pushing data to the browser) and reliable delivery is _not_ a feature of this tool. There's a video of a talk by David Fowler and some other engineer on the SignalR team where they mention that at one point SingalR had some logic for retransmittning messages to subscribers that temporarliy lost their connection but that feature is gone.

A lot the info on this page comes from [this podcast episode](https://topenddevs.com/podcasts/adventures-in-net/episodes/mastering-signalr-net-176).

## Connections

The SignalR npm module(?) establishes one connection for each hub. Also each tab in the browser has it's own connections. So it's a good practise to have as few hubs as possible.

There are events the tell you when the connection has been lost.

## Scaling issues

Each client connects to a hub and the hub keeps track of which clients should be updated when a message is sent to a group. This makes things a bit complicated when you have to scale the hub and load balance the traffic to the hubs. How does one hub know about the connections made to another hub? This problem could be solved by using a db such as Redis _or_ by using a SignalR resource in Azure.

## Azure

### Pricing

According to the aforementioned podcast episode, Azure charges you per day when it comes to SignalR resources.

### Auth

Clients can connect using a connection string which contains some key or they can use a bearer token.

TODO: Find out exactly how you setup an Azure SingalR resource to do JWT authorization. I've recently seen requests being denied with 401 Unauthorized but could not figure out how this was setup.
