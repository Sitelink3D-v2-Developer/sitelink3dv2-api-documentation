# Chapter 6 SmartView
SmartView is an umbrella term for one of the live streaming technologies offered by Sitelink3D v2. Compared with MFK Live (Machine Forward Kinematics), SmartView involves less programming investment to start using and is capable of providing information at various resolutions and with configurable server side filtering. These features facilitate a set of live streaming capabilities that can be used to implement expressions that make sense to the end user.

## SmartView Microservice
The ```smart_view``` microservice manages requests for SmartView data. It streams data using SSE (server sent events) and manages the lifetimes of SmartApps discussed below.

## SmartApps
SmartView delivers functionality by means of SmartApps. A SmartApp is simply a piece of serverside JavaScript that provides information about a specific bounded context of site operations that a user may wish to subscribe to. Each SmartApp has a particular purpose and is only concerned with providing information within its domain. SmartApp examples include:

- Machine locations and state.
- Weight payloads sent between machines.
- Haul activity.

A SmartApp is identified by three settings: 

- Key which is a string like ‘topcon/machines/machine_list’.
- Version. 
- Start time.

To subscribe to a SmartApp, you will need to know the following settings:
1. Site of interest: you need the site identifier (hereafter called SITE).
2. SmartApp of interest: SmartApps have two distinguishing parts of their identity as discussed above:
 * Key.
 * Version.
3. Settings that relate to the SmartApp specifically.
4. The server URL.
5. An authorization token. See the token example [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples/blob/main/components/tokens/get_token.py) for more information.

These get put together into a GET request like this: 

1. SITE is left as a SHA-256 value. 
2. APP_B64 is formed from the base 64 encoding of 
 - ```key:version``` to start a specific version.
 - ```key``` to start the most recent version.
3. START_B64 is formed from the base 64 encoding of {"from":"continuous"} which provides a continuous stream of data.
4. Any other settings passed as parameters.

### Querying Deployed SmartApps

The currently deployed SmartApps along with their versions and some basic usage information can be viewed in the QA environment at https://qa-api.sitelink.topcon.com/smart_view/unstable/smartapps/index.html

### Connecting to a SmartApp

Using SmartView is as simple as performing a GET to establish a subscription. The "x-topcon-hashes" value must be base64 encoded. The event stream will be returned.

METHOD : GET

URL : {SERVER_URL}/smart_view/v2/sites/{SITE}/apps/{APP_B64}/start/{START_B64}

HEADERS: 
```
{
  "Authorization": "Bearer {TOKEN}"
  "Content-Type" : "text/event-stream"
  "x-topcon-hashes" : { "key-1" : "crc32-value-for-key-1", "..." : "..." }
}
```

The response code is 200 on success and an event stream is made available for reading.

### Consuming a SmartApp Event Stream

As described [here](https://html.spec.whatwg.org/multipage/server-sent-events.html#event-stream-interpretation), the event stream consists of lines that are: 
- blank.
- comments (start with ‘:’).
- events (start with ‘event:’).
- data (start with ‘data:’).

Language specific handling is required to process the event streams: 
- Python requests library requires a stream=True parameter.
- Javascript requires an EventSource.

JSON frames below always come one per line; whitespace added for clarity

#### Loading
While the SmartApp is loading, you will receive loading events, which look like:
```
: 2020-05-07T06:01:04Z
event: loading
data: {"at": "2020-05-07T06:01:04Z", "data": {"count": 0, "elapsed_ms": 0, "progress": 0}, "event": "loading"}
: 2020-05-07T06:01:05Z
event: loading
data: {"at": "2020-05-07T06:01:05Z", "data": {"count": 6119, "elapsed_ms": 1000, "progress": 0.8734971408493168}, "event": "loading"}
: 2020-05-07T06:01:06Z
event: loading
data: {"at": "2020-05-07T06:01:06Z", "data": {"count": 18019, "elapsed_ms": 2000, "progress": 0.9482007370291521}, "event": "loading"}
```
Here the progress goes from 0 to 1. So progress*100 is the percent complete.

#### Loading Complete

```
{
  {
  "from": "shift_plan",
  "shift_plan": "shift_plan_default"
  },
  "version": "0.0"
}
```
Most of these are values simply confirm how the server was configured. 

Then there are timestamp events:
```
: 2020-04-27T05:40:37Z
```
and updates, which will be a data frame of the form:
```
{
"key" : "value"
}
```

For example our Machine List app will return frames like

```
: 2020-04-27T05:40:47Z
event: update
data: 
{
    "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ABF-010": 
    {
        "asset_class": "truck",
        "at": 1588831155590,
        "device": 
        {
            "ac": "iOS",
            "lease_start": 1588822275587,
            "name": "ABF-010",
            "type": "device",
            "urn": "urn:X-topcon:device:ac:iOS:oem:Apple:model:x86_64:sn::id:ABF-010",
            "uuid": "7bc93adf-413c-3b7c-8f48-7398a57a16bd"
        },
        "event": 
        {
            "at": 1588830851488,
            "quantity": 8,
            "region": "Airport - Dump Fill Dirt",
            "region_uuid": "657cbef6-3e47-49c8-8761-eacff5666e36",
            "type": "dump",
            "uuid": "bc97ca4b-fd5b-446a-a4c3-ff6c786b371b"
        },
        "lease_start": 1588822275587,
        "machine": 
        {
            "asset_class": "truck",
            "name": "ABF-010",
            "oem": "nonesuch",
            "type": "machine"
        },
        "state": 
        {
            "activity": "Hauling",
            "activity_uuid": "a419d0aa-78bd-11ea-9f5e-323b13daa92d",
            "material": "Fill Dirt",
            "material_uuid": "6d099e50-53b4-4000-8a02-d1814be60fe6",
            "operator": "George Jefferson",
            "operator_uuid": "92a5d225-2b09-472b-aaeb-a6fc50724f83",
            "task": "Haul Fill Dirt",
            "task_uuid": "6a6d6e7a-6a38-4ccb-ba2a-48736ca7e32a"
        },
        "urn": "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ABF-010"
    },
    "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ABF-010.position": 
    {
        "h": "7f6d0db5",
        "v": 
        {
            "altitude": 236.40848213614407,
            "at": 1588831258534,
            "latitude": 37.34936439543866,
            "longitude": -95.53161495041546,
            "urn": "urn:X-topcon:machine:ac:truck:oem:nonesuch:model::sn::id:ABF-010"
        }
    }
}

```

#### Close Events 

When something goes wrong internally, you will get a close event. Generally, given one of these, our code will simply retry. If you didn’t get any data, you should check that the message doesn’t indicate a bad request.
```
event: closed
data: 
{
    "message": "bad status 404 Not Found from ...: Not found"
}
```


## Examples
The directories nested under this file provide specific information on the operation of the available SmartApp examples available in our example code repository.