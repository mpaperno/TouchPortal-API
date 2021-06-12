# [TouchPortal](https://touch-portal.com)-API for Python
Easy way to Build Plugins for TouchPortal with little understanding of Python.

## Installation
Simply run this in your command line `pip install TouchPortal-API` if your not able to you can download [here](https://pypi.org/project/TouchPortal-API/#files) and do `pip install [fileyoudownloaded]`
Make Sure your on latest version of TouchPortal current Version of TouchPortal API supports TP V2.3

## Usage
```python
import TouchPortalAPI # Import the api

TPClient = TouchPortalAPI.Client('YourPluginID') # Initiate the client (replace YourPluginID with your ID)

@TPClient.on('info')  # This Will run once You've connected to TouchPortal
def OnStart(client, data):
    # You must provide 2 parm in the function or else it will give error
    print('I am Connected!', data)


    TPClient.stateUpdate("(Your State ID)", "State Value") # This if you want to update a dymic states in TouchPortal

    updateStates = [
        {
            "id": "(Your State ID)",
            "value": "(The Value You wanted)"
        },
        {
            "id": "(Your 2nd State ID)",
            "value": "(The Value You wanted)"
        }
    ]
    TPClient.stateUpdateMany(updateStates) # Or You can create an list with however many state you want and use this function to send them all

@TPClient.on('action')  # This manages when you press a button in TouchPortal it will send here in json format
def Actions(client, data):
    print(data)

@TPClient.on('settings') # This Function will get called Everytime when someone changes something in your plugin settings
def Settings(client, data):
    print('received data from settings!')

@TPClient.on('closePlugin') # When TouchPortal sends close Plugin message it will run this function
def shutDown(client, data):
    print('Received shutdown message!')
    TPClient.disconnect() # This is how you disconnect once you received the closePlugin message


TPClient.connect() # Connect to Touch Portal

```

## Example Plugin
Make a Folder in `%appdata%/TouchPortal/plugins/` called `ExamplePlugin`
and make a file called entry.tp and paste this json data inside.
```json
{
  "sdk": 3,
  "version": 100,
  "name": "Example Plugin",
  "id": "ExamplePlugin",
  "configuration": {
    "colorDark": "#222423",
    "colorLight": "#020202"
  },
  "categories": [
    {
      "id": "Main",
      "name": "Example Plugin",
      "actions": [
      	{
      	  "id": "ExampleAction",
          "name": "This is Example Action",
          "prefix": "plugin",
          "type": "communicate",
          "tryInline": true,
          "format": "Print({$ExampleTextData$})",
          "data": [
       	 	  {
        	    "id": "ExampleTextData",
              "type": "text",
              "label": "text",
              "default": "Hello World"
		        }
	         ]
	      }
      ],
      "events": [],
      "states": [
	      {
          "id": "ExampleStates",
          "type": "text",
          "desc": "Example States",
          "default": "None"
        }
      ]
    }
  ]
}
```

Save this somewhere and also Make sure you've Setup the entry.tp file as well then reboot TouchPortal
you should see your plugin. Without This script the Plugin wont do anything right? lets run this file
and Use one of the action! Note This is just a Example Plugin
```python
import TouchPortalAPI
from TouchPortalAPI import TYPES

# Setup callbacks and connection
TPClient = TouchPortalAPI.Client("ExamplePlugin")

@TPClient.on(TYPES.onConnect) # Or replace TYPES.onConnect with 'info'
def onStart(client, data):
    print("Connected!", data)

@TPClient.on(TYPES.onAction) # Or 'action'
def Actions(client, data):
    print(data)
    if data['actionId'] == "ExampleAction": # we filter the Action we wanted to tigger when User Press it
        print(data['data'][0]['value']) # We grab the Value from the Action and print out
    TPClient.stateUpdate("ExampleStates", data['data'][0]['value']) # We can also update our ExampleStates with the Action Value

@TPClient.on(TYPES.onShutDown) # or 'closePlugin'
def shutDown(client, data):
    print("Got Shutdown Message! Shutting Down the Plugin!")
    TPClient.disconnect() # This stops the connection to TouchPortal

# After Callback setup like we did then we can connect
TPClient.connect()
```


## TouchPortalAPI.Tools
`from TouchPortalAPI import Tools`
- Tools.convertImage_to_base64() # This can be both Url based Image or Local Image from your pc
- Tools.updateCheck(githubUser, githubRepoName, CurrentVersion)

## TouchPortalAPI.TYPES
- TYPES.onHold_up  - "up"
- TYPES.onHold_down  - "down"
- TYPES.onConnect  - "info"
- TYPES.onAction  - "action"
- TYPES.onListChange  - "listChange"
- TYPES.onShutdown  - "closePlugin"
- TYPES.onBroadcast  - "broadcast"
- TYPES.onSettingUpdate  - "settings"
- TYPES.allMessage  - "message" # This one is not build in to TouchPortal it's a custom one that it send at any message from TP

## List of Methods
- isActionBeingHeld(actionId)
  - This returns `True` or `False` for an Action ID. If you have an Action that can be held, this nethod would return `True` while it is being held, and `False` otherwise.
- createState(stateId, description, value)
  - This will create a TP State at runtime. `stateId`, `description`, and `value` are all required (`value` becomes the State's default value).
  If the State already exists, it will be updated with `value` instead of being re-created.
- createStateMany(stateId, states:list)
  - Convenience function to create several States at once. `states` should be an iteratable of `dict` types in the form of `{'id': "StateId", 'desc': "Description", 'value': "Default Value"}`.
- removeState(stateId)
  - This removes a State that has been created at runtime. `stateId` needs to be a string.
- removeStateMany(states)
  - Convenience function to remove several States at once. `states` should be an iteratable of state ID strings.
- choiceUpdate(stateId, values)
  - This updates the list of choices in a previously-declared TP State with id `stateId`. See TP API reference for details on updating list values.
- choiceUpdateSpecific(stateId, values, instanceId)
  - This updates a list of choices in a specific TP Item Instance, specified in `instanceId`. See TP API reference for details on updating specific instances.
- settingUpdate(settingName, settingValue)
  - This updates a value in your plugin's Settings.
- stateUpdate(stateId, stateValue)
  - This updates a value in ether a pre-defined static State or a dynamic State created in runtime.
- stateUpdateMany(states)
  - Convenience function to update serveral states at once. `states` should be an iteratable of `dict` types in the form of `{'id': "StateId", 'value': "The New Value"}`.
- updateActionData(instanceId, stateId, minValue, maxValue)
  - This allows you to update Action Data in one of your Action. Currently TouchPortal only supports changing the minimum and maximum values in numeric data types.
- send(data)
  - This will try to send any arbitrary Python object in `data` (presumably something `dict`-like) to TouchPortal after serializing it as JSON and adding a `\n`.
  Normally there is no need to use this method directly, but if the Python API doesn't cover something from the TP API, this could be used instead.
- connect()
  - Call this method to connect to TouchPortal after all your setups are complete. Normally this is used at the end of your script.
  Does nothing if the client is already connected.
- disconnect()
  - Trigger the client to disconnect from TouchPortal. Normally this is used in `@TPClient.on("closePlugin")` callback but it can be used any way you like only
  after you've connected to TouchPortal. Does nothing if client is not currently connected.

## Touch Portal api documentation
https://www.touch-portal.com/api

## Bugs
If theres are any bugs, Issues or if you need help feel free use Issue tab

## Contribute
Feel Free to suggest a pull request or Fork
