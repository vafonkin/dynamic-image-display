# dynamic-image-display

A Lightning Component to respond to real time Platform Events to have a real-time display of status.

## Sample use case

After a routine servicing of a connected machine, an access panel is left open. The machine's sensors notice the door is not fully closed, and sends out a status that the door is open. IoT Explorer receives that message, and changes the status of the Asset in SF that the door is open. On the Asset record in SF, the image of the machine is dynamically updated, without having to refresh the page or record, that the door is open.

## Use

1. Install the component, custom metadata type, and streamer component from the repository
1. Create (or re-use) a Platform Event
  * Find Platform Events under **Setup > Data > Platform Events**
  * Two fields are required to be added
    * REQUIRED message field, **Long Text Area**
    * OPTIONAL key field, **Text**
1. Extend the Target Object
  * Add a new field to track the status
    * Might be a **Text** or **Long Text Area**
  * Stores the state of the various layers from the configuration
    * Is a JSON Object, of name-value pairs; name is the layername, value is whether to hide the layer or not
    * Ex:
    ```
    {
      "error": "show",
      "warning": "hide"
    }
    ```
1. Add a new entry to the Custom Metadata Type **Image List**
  * Repository already contains the **Image List** custom metadata type
  * **Image List Name** is the developer name for this entry, used in the setting of the component
  * **Image Config** is the actual set of layers to use in the component
    * Is a JSON Object Array, each JSON Object represents the layer and associated image to load into that layer
    * Can also leverage an archive, see example below (resource name if no archive, with archive you need a root '/' before path and name)
    * Ex:
    ```
    [
      {
        "layername": "base",
        "image": "image_base"
      },
      {
        "layername": "error",
        "image": "/image_error",
        "archive": "images"
      },
      {
        "layername": "warning",
        "image": "/image_warning",
        "archive": "images"
      }
    ]
    ```
    * **base** is a special layer name, typically always shown, the only layer that is defaulted to "show", all others are defaulted to "hide"
1. Modify the Record Page for the target object from above to show this component.
  * Suggested first use is on the right sidebar
  * Channel is of the form "/event/Event__E"
  * Use the target object developer field name from above (e.g. DisplayStatus__c)
  * As appropriate, use the filter field info from the message and target object
## Testing

Sample configuration and data are provided to show how this all works together.

### Dynamically change the image

In the Dev Console, under Debug > Open Execute Anonymous Window, run this code:

```
List<TestEvent__e> evts = new List<TestEvent__e>();
evts.add(new TestEvent__e(
    Message__c = '{ "left": "hide", "right": "show" }'
	)
);
EventBus.publish(evts);
```

### Import the data

```
sfdx force:data:tree:import -p ./data/Image_Status__c-plan.json
```

### Export the data

```
sfdx force:data:tree:export -q ./data/imagestatus.soql -p -d ./data/
```

## Resources

1. Streaming component leverages work from Andrew Fawcett at https://github.com/afawcett/streamingcomponent
