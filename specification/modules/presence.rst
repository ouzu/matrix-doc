.. Copyright 2016 OpenMarket Ltd
..
.. Licensed under the Apache License, Version 2.0 (the "License");
.. you may not use this file except in compliance with the License.
.. You may obtain a copy of the License at
..
..     http://www.apache.org/licenses/LICENSE-2.0
..
.. Unless required by applicable law or agreed to in writing, software
.. distributed under the License is distributed on an "AS IS" BASIS,
.. WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
.. See the License for the specific language governing permissions and
.. limitations under the License.

Presence
========

.. _module:presence:

Each user has the concept of presence information. This encodes:

* Whether the user is currently online
* How recently the user was last active (as seen by the server)
* Whether a given client considers the user to be currently idle
* Arbitrary information about the user's current status (e.g. "in a meeting").

This information is collated from both per-device (``online``, ``idle``,
``last_active``) and per-user (status) data, aggregated by the user's homeserver
and transmitted as an ``m.presence`` event. This is one of the few events which
are sent *outside the context of a room*. Presence events are sent to all users
who subscribe to this user's presence through a presence list or by sharing
membership of a room.

A presence list is a list of user IDs whose presence the user wants to follow.
To be added to this list, the user being added must be invited by the list owner
who must accept the invitation.

User's presence state is represented by the ``presence`` key, which is an enum
of one of the following:

- ``online`` : The default state when the user is connected to an event
  stream.
- ``unavailable`` : The user is not reachable at this time e.g. they are
  idle.
- ``offline`` : The user is not connected to an event stream or is
  explicitly suppressing their profile information from being sent.

Events
------

{{presence_events}}

Client behaviour
----------------

Clients can manually set/get their presence/presence list using the HTTP APIs
listed below.

{{presence_cs_http_api}}

Server behaviour
----------------

Each user's homeserver stores a "presence list" per user. Once a user accepts
a presence list, both user's HSes must track the subscription.

Last active ago
~~~~~~~~~~~~~~~
The server maintains a timestamp of the last time it saw a pro-active event from
the user. A pro-active event may be sending a message to a room or changing
presence state to ``online``. This timestamp is presented via a key called
``last_active_ago`` which gives the relative number of milliseconds since the
pro-active event.

To reduce the number of presence updates sent to clients the server may include
a ``currently_active`` boolean field when the presence state is ``online``. When
true, the server will not send further updates to the last active time until an
update is sent to the client with either a) ``currently_active`` set to false or
b) a presence state other than ``online``. During this period clients must
consider the user to be currently active, irrespective of the last active time.

The last active time must be up to date whenever the server gives a presence
event to the client. The ``currently_active`` mechanism should purely be used by
servers to stop sending continuous presence updates, as opposed to disabling
last active tracking entirely. Thus clients can fetch up to date last active
times by explicitly requesting the presence for a given user.

Idle timeout
~~~~~~~~~~~~

The server will automatically set a user's presence to ``unavailable`` if their
last active time was over a threshold value (e.g. 5 minutes). Clients can
manually set a user's presence to ``unavailable``. Any activity that bumps the
last active time on any of the user's clients will cause the server to
automatically set their presence to ``online``.

Security considerations
-----------------------

Presence information is shared with all users who share a room with the target
user. In large public rooms this could be undesirable.
