![RTIG Logo with strapline](images/rtig_document_header_logo.png)

# CMS to PID Interface Protocol

# Part 2 - Core Content Messages

## Status of this document

If there are any comments or feedback arising from the review or use of this document, please contact us at secretariat@rtig.org.uk

# Common Data Structures

* The following reusable data structures relate to low-level data objects. They do not constitute message types in their own right; they are used within multiple message types across multiple service types.

* While there is nothing preventing the extension of these data structures through the addition of further attributes, this is not recommended.

## Multi Language Text

* The interface supports multiple languages for all text that can be displayed on a PID using the following JSON structure:
```json
"multiLanguageText": [
	{
		"lang": "EN",
		"text": "Some text in English"
	},
	{
		"lang": "CY",
		"text": " Rhywfaint o destun yn Saesneg"
	},
	...
]
```

* Each text attribute is an array of tuple objects containing an ISO 639-1 two-character language code ("lang") and a text string ("text").

## External Reference

* Wherever references to external data sets are used, it is proposed that they are encoded using the following JSON structure that encapsulates the data source along with the reference code:
```json
"externalRef": {
	"source": "DataSetName:ObjectName.AttributeName",
	"ref": "UniqueIdentifierWithinDataSet"
}
```
* Whilst this interface specification should not be regarded as being "UK only", where it is used for bus data within the UK, the following "business rules" should be applied:

* Where an "operatorRef" is used in UK message content, this should be the National Operator Code of the operator running a given service, as defined by Traveline (see https://www.travelinedata.org.uk/traveline-open-data/transport-operations/about-2/). Where the data applies to a bus service being operated in England, the National Operator Code should appear as it is recorded in the DfT's Bus Open Data Service (BODS) for any given trip, for other administrations this will be the locally agreed code. For example:
```json
{
	"source": "NOC:NOC",
	"ref": "FSYO"
}
```
* Where a "locationRef" is used in UK bus message content, this should be an ATCO Code defined in the NaPTAN database (see https://www.gov.uk/government/publications/national-public-transport-access-node-schema). For example:
```json
{
	"source": "NaPTAN:AtcoCode",
	"ref": "490000077E"
}
```
* Or for rail in the UK it is expected that this will be the CRS code for a rail station.

* Where a "lineRef" is used in message content relating to bus services in England, this should correspond with the TransXChange Line id attribute used in BODS, for other administrations this will be the locally agreed code:
```json
{
	"source": "BODS:Service.Line.id"
	"ref": "FSYO:PB0002307:141:X5"
}
```
* Where a "directionRef" is used in message content relating to bus services in England, this should match the TransXChange direction used for a given vehicle journey in BODS for other administrations this will be the locally agreed code:
```json
{
	"source": "BODS:VehicleJourney.Direction",
	"ref": "outbound"
}
```
* Where a "vehicleJourneyRef" is used in message content, this should match the TransXChange vehicle journey id used for a given vehicle journey in BODS for other administrations this will be the locally agreed code:
```json
{
	"source": "BODS:VehicleJourney.id",
	"ref": "VJ1"
}
```
* Similar sets of "business rules" are likely to emerge for other modes and data standards (e.g. UK rail references to TIPLOC or DARWIN data sets, European references to NeTEx data, references to GTFS data); these should be formalised and applied in a similar manner.

## Vehicle Journey Definition

* An individual timetabled journey should be described using the following structure:
```json
"vehicleJourneyDefinition": {
	"lineRef": { // external reference
		"source": "BODS:Service.Line.id"
		"ref": "ABCB:PB0009999:111:123A"
		},
	"directionRef": { //external reference
		"source": "BODS:VehicleJourney.Direction",
		"ref": "outbound"
	},
	"vehicleJourneyRef": { // external reference
		"source": "BODS:VehicleJourney.id",
		"ref": "VJ1"
	},
	"operatorName": [ // multi-language text
		{
			"lang": "EN",
			"text": "The ABC Bus Company"
		}
	],
	"operatorRef": { // external reference
		"source": "NOC:NOC",
		"ref": "ABCB"
	},
	"publishedLineName": [ // multi-language text
		{
			"lang": "EN",
			"text": "123A"
		}
	"marketingLineName": [ // multi-language text
		{
			"lang": "EN",
			"text": "South Coast Express"
		}
	],
	"originRef": { // external reference
		"source": "NaPTAN:AtcoCode",
		"ref": "490000077E"
	},
	"originName": [ // multi-language text
		{
			"lang": "EN",
			"text": "Highbury"
		}
	],
	"destinationRef": { // external reference
		"source": "NaPTAN:AtcoCode",
		"ref": "490000054B"
	},
	"destinationName": [ // multi-language text
		{
			"lang": "EN",
			"text": "Paradise Park"
		}
	],
	"via": [ // multi-language text
		{
			"lang": "EN",
			"text": "City Centre, Park and Ride North"
		}
	],
	"poi": [ // multi-language text
		{
			"lang": "EN",
			"text": "Castle Ruins, All Saints Church"
		}
	]
}
```
## Location Configuration

* The location configuration data structure holds details of the localities, clusters and stop to which a PID belongs, together with the public-facing location name and geographic coordinates:
```json
"locationConfig": {
	"localities": [],
	"clusters": []
	"stops": [],
	"locationName": [ // multiLanguageText object
		{
			"lang": "EN",
			"text": "Cardiff Central"
		},
		{
			"lang": "CY",
			"text": "Caerdydd Canolog"
		}
	],
	"geographicCoordinates": {
		"longitude": -2.05467,
		"latitude": 53.53774
	}
}
```
N.B. The arrays of localities, clusters and stops within the locationConfig are not multi-lingual as they are purely used for creating MQTT topic subscriptions; they are not used for public-facing display purposes.

## Text Display Configuration

* The text display configuration data structure holds details of available character encodings and fonts, the number of lines available for text display, the number of characters available per text display line and a flag indicating whether smooth scrolling is supported:
```json
"textDisplayConfig": {
	"charEncodings": [ // array of string values
		...
	],
	"fonts": [ // array of string values
	...
	],
	"lines": 3, // integer value
	"charsPerLine": 26, // integer value
	"canScroll": true, // boolean value
}
```
* Where no character encodings are specified, UTF-8 will be the default.

* Font configuration primarily provides support for graphical displays and may be left as an empty array for text displays with no explicit font support.

## Status Report

* This provides basic information about a display's operational status.
```json
"statusReport": {
	"firmwareVersion": "XYZ.789.001.A",
	"operatingSystem": "Homebrew Linux 3.11",
	"upSince": "2020-11-17T06:30:00-00:00",
	"memoryUsage": "63%",
	"loadAverage": "78%", // one minute load average
	"diskFreeSpace": "7.6Gb"
}
```
* Vendors are free (and encouraged) to extend the status report structure to include further information as appropriate.

# Common Enumerated Types

## Reusable structures

* The following reusable data structures relate to low-level data objects. They do not constitute message types in their own right; they are used within multiple message types across multiple service types.

## Device Type
```json
[
	CMS,
	DS, // discovery service
	VSS, // vendor-specific service
	PID-LED,
	PID-LCD,
	PID-TFT,
	PID-EPAPER
]
```
## Display Status Type

* Display instructions appearing within messages should always correspond to one of the following enumerated status values:
```json
[
	OFF, // display switched off (i.e. screen blank)
	SCHED_DEP, // display should only show scheduled departures
	RT_DEP, // display should only show real-time departures
	MIXED_DEP, // display should show sched and r/t departures
	SCHED_ARR, // display should only show scheduled arrivals
	RT_ARR, // display should only show real-time arrivals
	MIXED_ARR, // display should show scheduled and r/t arrivals
	SCHED_ARRDEP, // display should show sched arr and dep
	RT_ARRDEP, // display should show real-time arr and dep
	MIXED_ARRDEP, // display should show sched and r/t arr and dep
	NO_DATA // screen remains on - e.g. to display clock
]
```
## Sensor Type

* Information messages can be displayed in a variety of different ways:
```json
[
	INT-TEMP, // internal temperature
	EXT-TEMP, // external temperature
	SHOCK,
	GPS,
	CELL-DATA,
	LIGHT-LEVEL
]
```
* This list provides for some common sensor types contained within displays. Additional sensor types may be defined by vendors in other parts for example for management or accessibility.

## Real-Time Status

* Real-time departure messages can be displayed in a variety of different ways:
```json
[
	ON-TIME,
	EARLY,
	LATE,
	DELAYED,
	CANCELLED
]
```
## Message Display Style

* Information messages can be displayed in a variety of different ways, this is managed having a Target Area and a Behaviour:

* Message Target Area
```json
[
	BOTTOM_LINE,
	FULL_SCREEN, //overrides all other content on display
	OVERLINE //full screen message leaves bottom line available for other messages.
]
```
* Message Behaviour
```json
[
	SCROLL, // default
	PAGED,
	STATIC
]
```
* Deprecated DisplayStyle to be removed in next version.
```json
[
	SCROLL, //bottom line
	PAGED, // bottom line
	STATIC, // bottom line
	FULL-SCREEN, //overrides all other content on display
	OVERLINE, // full screen message leaves bottom line available for other messages.
]
```
## Message Action

* Messages can be managed in a variety of different ways:
```json
[
	DELETE,
	REPLACE
]
```
## Message Priority Type

* Messages can be given a priority of either:
```json
[
	STANDARD,
	EMERGENCY
]
```
* Where a message is of priority STANDARD this shall be enhanced with messageActionPriority which shall be a positive integer, where 1 is the highest priority and the lowest priority being 99.

* This approach reflects the complex nature of business requirements for message management whilst retaining as much simplicity as possible.

* Where messages are received by the CMS from a third party source using SIRI SX and the ActionPriority element is set this should be mapped to messageActionPriority.

## Cleardown Reason

* Cleardown messages should specify the reason that a departure is being cleared down:
```json
[
	DEPARTED,
	CANCELLED,
	SHORT_WORKING,
	DIVERTED,
	ARRIVED
]
```
## Occupancy Level

* Occupancy levels are taken from RTIGT043, and are based on the values available in SIRI v2.1 (which has itself been aligned with GTFS-RT):
```json
[
	FULL,
	STANDINGAVAILABLE,
	SEATSAVAILABLE,
	UNKNOWN,
	EMPTY,
	MANYSEATSAVAILABLE,
	FEWSEATSAVAILABLE,
	STANDINGROOMONLY,
	CRUSHEDSTANDINGROOMONLY,
	NOTACCEPTINGPASSENGERS
]
```
## Vehicle Mode

* Vehicle Mode is taken from the SIRI v2.1 VehicleModesEnumeration:
```json
[
	Air,
	Bus,
	Coach,
	Ferry,
	Metro,
	Rail,
	Tram,
	underground
]
```
* Where vehicleMode is not provided it should be assumed to be BUS.

## Reason

* Used in multimodal data:
```json
[
	newDeparture,
	departureComment,
	platform
]
```
## Status

* Used in delivery of content:
```json
[
	DONE,
	NO_DATA
]
```
# Core Service Messages

## Unidirectional

* The following message types are applicable to all display types. In each case these are unidirectional (i.e. single phase) message types and always have the third level of topic name set to the constant value service:
	* Heartbeat
	* Display Override
	* Sensor Event

## Heartbeat

* Heartbeat messages are designed to be as small as possible. They should be published from the display using the topic:
```json
{device_type}/heartbeat/service
```
* They should carry the payload:
```json
{
	"heartbeat": {
		"vendorId": "VENDOR0001", // unique within project
		"deviceId": "ABCD1234567890", // unique for the vendor
		"timestamp": "2020-12-17T17:30:00-00:00"
		"vendorData": {
			"object1": 0,
			"object2": "0.2.0"
		}
	}
}
```
* Where vendorData is provided this shall be minimised to control the volume of data transmitted and agreed by the implementing project to ensure that the CMS is able to support the inclusion of each object.

## Display Override

* Where the CMS wishes to override the current display status of a PID, it should publish a Display Override message to a topic that corresponds with one or more registered PIDs:
```json
{displayType}/displayOverride/ /{vendor_id}/{device_id}
{
	"displayOverride": {
		"status": "OFF", // display status type enumerated value
		"timestamp": "2020-12-17T17:30:00-00:00"
	}
}
```
* If a display override is active, it is expected that the displayOverride attribute in the status response from the display is set, so that the CMS is aware of the current displayOverride status.

## Sensor Event

* Modern PIDs often contain sensors that measure and monitor environmental conditions that can be reported back to the CMS - for example internal temperature, external temperature, shock detection, etc. In accordance with MQTT best practice, it is proposed that each sensor type should be given its own topic, thereby allowing subscribers to filter out messages from sensor types that are not relevant to them.

* Sensor Event messages should use the following topic name:
```json
{device_type}/sensorEvent/service/{type}
```
* It is anticipated that the message content for the majority of sensor content will follow the following convention:
```json
"sensorEvent": {
	"vendorId": "VENDOR0001", // unique within project
	"deviceId": "ABCD1234567890", // unique for the vendor
	"type": "INT-TEMP", // from sensor type enumeration
	"value": "24.7",
	"unit": "degreeCelsius",
	"data": {
		"vendor-specific-data-1": "value-1",
		"vendor-specific-data-2": "value-2",
		...
	}
}
```
* At present, it is recommended that the sensor type is duplicated in the topic name and the message payload - allowing (future) devices and services to subscribe to sensor events of a particular type. In each case, the type should be taken from the sensor type enumeration.

 N.B. The "data" attribute is designed to provide unlimited extensibility, depending on the requirements of PID vendors. It might be necessary to place some constraints on the content that is permitted here. This will require further discussion with PID vendors.

N.B. Specific event types for accessibility (e.g. Bluetooth beacon activations, physical push button requests, etc.) will be added in a later document part.

# Core Content Messages.

* The following message types are applicable to all display types. In each case these are unidirectional (i.e. single phase) message types and always have the third level of topic name set to the constant value content:
	* Scheduled Departure
	* Real-Time Departure
	* Information Message

## Scheduled Departure

* Scheduled departure messages should use the following topic name structure:
```json
CMS/scheduledDeparture/content/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
```
* Loosely based on the content of a SIRI-ST <TimetabledStopVisit> message, the following message structure has been expanded to include multi-language text and external references. In addition, support has been added for "via" text, points of interest and operator brand name:
```json
{
	"scheduledDepartures": [
		{
			"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
			"vehicleMode": "BUS",
			"targetedVehicleJourney": { // vehicle journey definition
				"lineRef": { // external reference
					"source": "BODS:Service.Line.id",
					"ref": "ABCB:PB0009999:111:123A"
				},
				"directionRef": { // external reference
					"source": "BODS:VehicleJourney.Direction",
					"ref": "outbound"
				},
				"vehicleJourneyRef": { // external reference
					"source": "BODS:VehicleJourney.id",
					"ref": "VJ1"
				},
				"operatorName": [ // multi-language text
					{
						"lang": "EN",
						"text": "The ABC Bus Company"
					}
				],
				"operatorRef": { // external reference
					"source": "NOC:NOC",
					"ref": "ABCB"
				},
				"publishedLineName": [ // multi-language text
					{
						"lang": "EN",
						"text": "123A"
					}
				],
				"marketingLineName": [ // multi-language text
					{
						"lang": "EN",
						"text": "South Coast Express"
					}
				],
				"originRef": { // external reference
					"source": "NaPTAN:AtcoCode",
					"ref": "490000077E"
				},
				"originName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Highbury"
					}
				],
				"destinationRef": { // external reference
					"source": "NaPTAN:AtcoCode",
					"ref": "490000054B"
				},
				"destinationName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Paradise Park"
					}
				],
				"via": [ // multi-language text
					{
						"lang": "EN",
						"text": "City Centre, Park and Ride North"
					}
				],
				"poi": [ // multi-language text
					{
						"lang": "EN",
						"text": "Castle Ruins, All Saints Church"
					}
				]
			},
			"targetedCall": {
				"visitNumber": "1",
				"quay": "7", // Platform, stand etc - Quay is the Transmodel name
				"aimedArrivalTime": "2022-02-17T09:40:46-00:00",
				"aimedDepartureTime": "2022-02-17T09:42:47-00:00",
				"expectedArrivalTime": null,
				"expectedDeparturetime": null
			}
		},
		...
	]
}
```
N.B. At the highest level, "scheduledDepartures" is defined as an array, allowing multiple scheduled departures to be sent in a single message payload. In practice it may be necessary to limit the number of scheduled departures in a single message to avoid excessive payload sizes (as these may be limited by MQTT brokers or bridges on the network).

* expectedArrivalTime and expectedDepartureTime are nullable optional elements. If the element is null or not present, then this signifies that real time is not available. Do not populate with times if real time updates are not available. Real time updates should normally be provided using the realTimeDeparture topic.

* An example of a rail departure would be:
```json
{
	"scheduledDepartures": [
		{
			"departureId": "db654e8a-4e9c-465d-326b-6bf2fb379ab7",
			"vehicleMode": "RAIL",
			"targetedVehicleJourney": { // vehicle journey definition
				"lineRef": { // external reference
					"source": "Darwin4Trains.Entity.ViewModels.TrainAmqViewModel",
					"ref": "DARWIN"
				},
				"directionRef": { // external reference
					"source": "Darwin4Trains.Entity.ViewModels.JourneyStatusLocationViewModel",
					"ref": "outbound"
				},
				"vehicleJourneyRef": { // external reference
					"source": "Darwin4Trains.Entity.ViewModels.JourneyStatusLocationViewModel",
					"ref": "VJ1"
				},
				"operatorName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Transport for Wales"
					}
					{
						"lang": "CY",
						"text": "Trafnidiaeth Cymru"
					}
				],
				"operatorRef": { // external reference
					"source": "NOC:NOC",
					"ref": "TFW"
				},
				"publishedLineName": [ // multi-language text
					{
						"lang": "EN",
						"text": "CDF - BYI"
					}
					{
						"lang": "CY",
						"text": "CDF - BYI"
					}
				],
				"marketingLineName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Barry Island"
					}
					{
						"lang": "CY",
						"text": "Ynys y Barri"
					}
				],
				"originRef": { // external reference
					"source": "Darwin4Trains.Entity.ViewModels.JourneyStatusLocationViewModel",
					"ref": "CDF"
				},
				"originName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Cardiff"
					}
					{
						"lang": "CY",
						"text": "Caerdydd"
					}
				],
				"destinationRef": { // external reference
					"source": "Darwin4Trains.Entity.ViewModels.JourneyStatusLocationViewModel",
					"ref": "BYI"
				},
				"destinationName": [ // multi-language text
					{
						"lang": "EN",
						"text": "Barry Island"
					}
					{
						"lang": "CY",
						"text": " Ynys y Barri "
					}
				],
				"via": [ // multi-language text
					{
						"lang": "EN",
						"text": "Grangetown (1559),Cogan (1603),Eastbrook (1605),Dinas Powys (1607),Cadoxton (1611),Barry Docks (1614),Barry (1618)"
					}
					{
						"lang": "CY",
						"text": "Grangetown (1559),Cogan (1603),Eastbrook (1605),Dinas Powys (1607),Tregatwg (1611),Dociau'r Barri (1614),Y Barri (1618)"
					}
				],
				"poi": [ // multi-language text
					{
						"lang": "EN",
						"text": "Barry Docks"
					}
					{
						"lang": "CY",
						"text": "Dociau'r Barri"
					}
				]
			},
			"targetedCall": {
				"visitNumber": "1",
				"quay": "8", // Platform, stand etc - Quay is the Transmodel name
				"aimedArrivalTime": "2022-02-17T09:40:46-00:00",
				"aimedDepartureTime": "2022-02-17T09:42:47-00:00",
				"expectedArrivalTime": null,
				"expectedDeparturetime": null
			}
		},
		...
	]
}
```
* The amount of scheduled data published in advance will be a project-specific requirement, but it is strongly recommended that a minimum 24 hour lookahead is provided.

## Real Time Departure

* Real-time departure messages should use the following topic name structure:
```json
CMS/realTimeDeparture/content/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
```

* Loosely based on the content of a SIRI-SM <MonitoredStopVisit> message, the payload of real-time departure messages has a lot in common with that of a scheduled departure message:
```json
{
	"realTimeDeparture": [
		{
			"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
			"predictionTime": "2004-12-17T09:25:46-05:00",
			"vehicleMode": "BUS",
			"monitoredVehicleJourney": { // vehicle journey definition
				"lineRef": { // external reference
					"source": "BODS:Service.Line.id"
					"ref": "ABCB:PB0009999:111:123A"
				},
				"directionRef": { // external reference
				"source": "BODS:VehicleJourney.Direction",
				"ref": "outbound"
			},
			"vehicleJourneyRef": { // external reference
				"source": "BODS:VehicleJourney.id",
				"ref": "VJ1"
			},
			"operatorName": [ // multi-language text
				{
					"lang": "EN",
					"text": "The ABC Bus Company"
				}
			],
			"operatorRef": { // external reference
				"source": "NOC:NOC",
				"ref": "ABCB"
			},
			"publishedLineName": [ // multi-language text
				{
					"lang": "EN",
					"text": "123A"
				}
			],
			"marketingLineName": [ // multi-language text
				{
					"lang": "EN",
					"text": "South Coast Express"
				}
			],
			"originRef": { // external reference
				"source": "NaPTAN:AtcoCode",
				"ref": "490000077E"
			},
			"originName": [ // multi-language text
				{
					"lang": "EN",
					"text": "Highbury"
				}
			],
			"destinationRef": { // external reference
				"source": "NaPTAN:AtcoCode",
				"ref": "490000054B"
			},
			"destinationName": [ // multi-language text
				{
					"lang": "EN",
					"text": "Paradise Park"
				}
			],
			"via": [ // multi-language text
				{
					"lang": "EN",
					"text": "City Centre, Park and Ride North"
				}
			],
			"poi": [ // multi-language text
				{
					"lang": "EN",
					"text": "Castle Ruins, All Saints Church"
				}
			],
			"progressStatus": "ON-TIME", // enum value
			"occupancyLevel": "SEATS-AVAILABLE", // enum value
			"monitoredCall": {
				"monitored": "true"
				"vehicleAtStop": "true",
				"quay": "7", // Platform, stand etc - Quay is the Transmodel name
				"aimedArrivalTime": "2022-02-17T09:39:00-00:00",
				"expectedArrivalTime":"2022-02-17T09:39:53-00:00:00",
				"aimedDepartureTime": "2022-02-17T09:40:00-00:00",
				"expectedDepartureTime":"2022-02-17T09:40:25-00:00:00",
				"departureStatus": "ON-TIME", // real-time status
				"arrivalStatus": "ON-TIME", // real-time status
			}

		}
	},
	...
	]
}
```
* Where monitored is false the expectedArrivalTime should be presented to the passenger.

* Where no valid prediction is available then expectedArrivalTime and / or (as appropriate) expectedDepartureTime must not be provided or set to null, and monitored should be false.

* Where no expected time was being provided and is subsequently provided this should trigger a prediction to be shown, and where expected time has been provided and is no longer provided it should trigger a prediction to be removed.

* The departure id is used for three purposes:
	* To match the prediction to the scheduled departure
	* to replace earlier predictions that relate to the same departure
	* to support rapid cleardown of the departure

## Information Message

* Information messages should use the following topic name structure:
```json
CMS/informationMessage/content/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}/{message_id}
```
* The inclusion of message_id allows for multiple messages to be set for any given device_id,

* Information messages can be given different priorities and different display styles - so, for example, the supplied message text could be displayed as a low priority scrolling message about future disruption or a full-screen emergency message.

```json
{
	"informationMessage": {
		"messageText": [
			{
				"lang": "EN",
				"text": "Service 123A suspended between 10am and 5pm today"
			},
			{
				"lang": "CY",
				"text": " Gwasanaeth 123A wedi'i atal rhwng 10am a 5pm heddiw"
			}
		],
		"earliestDisplayTime": "2022-02-17T10:00:00-00:00",
		"latestDisplayTime": "2022-02-17T17:00:00-00:00",
		"messagePriority": "STANDARD", // message priority type enumerated value
		"messageActionPriority": 5, // positive integer 1 = high
		"messageTargetArea": "SCROLL", //was displayTargetArea in previous versions
		"messageBehaviour": "SCROLL", // message display style enumerated value, was displayStyle in previous versions
		"messageId": UUID,
		"messageAction": "DELETE" // optional action to delete
	}
}
```
* There may be a requirement to delete an informationMessage from the display that has been sent previously. The optional attribute messageAction can be added to the informationMessage structure to replace or delete the message on the display with the messageId UUID provided when messageAction is set to DELETE only the messageId attribute is required. If a message with the provided UUID is not already present on the display, then no action should take place.

## Journey Message

* Journey messages should use the following topic name structure:
```json
CMS/journeyMessage/content/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}/{departure_Id}
```
* The inclusion of departure_Id allows for a message to be targeted for a specific journey (departure) from a specific (original) stop.

* Journey messages can be given different priorities and different display styles - so, for example, the supplied message text could be displayed as a low priority scrolling message, a static or paged message.
```json
{
	"journeyMessage": {
		"messageText": [
			{
				"lang": "EN",
				"text": "Cancelled"
			},
			{
				"lang": "CY",
				"text": "Canslo"
			}
		]
		"earliestDisplayTime": "2022-02-17T10:00:00-00:00",
		"latestDisplayTime": "2022-02-17T17:00:00-00:00",
		"messagePriority": "STANDARD", // message priority type enumerated value
		"messageActionPriority": 5, // positive integer 1 = high
		"messageBehaviour": "SCROLL", // message display style enumerated value, was displayStyle in previous versions
		"messageId": UUID,
		"messageAction": "DELETE" // optional action to delete
	}
}
```
* There may be a requirement to delete an journeyMessage from the display that has been sent previously. The optional attribute messageAction can be added to the journeyMessage structure to replace or delete the message on the display with the messageId UUID provided when messageAction is set to DELETE only the messageId attribute is required. If a message with the provided UUID is not already present on the display, then no action should take place.

# Bidirectional Communication Messages

## Bidirectional

* The protocol defines a number of scenarios which require bidirectional request/response style communication patterns:
	* Display Capability
		* Request from any device (but usually CMS)
		* Response from PID
	* Device discovery
		* Initiated by PID
		* Response from a discovery service (could be CMS or vendor-specific service)
		* Acknowledged by PID
	* Departure cleardown
		* Requested by CMS
		* Acknowledged by PID
	* Status reporting
		* Request from any device (but usually CMS)
		* Response by PID
	* Display Snapshots
		* Request from any device (but usually CMS)
		* Response by PID

## Display Capability

* A display capability request will typically be sent by a CMS to a single PID but could also be sent by another service on the MQTT network and/or sent to multiple PIDs using a topic name constructed as follows:
```json
{device_type}/displayCapability/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"displayCapabilityRequest": {
		"requestId": "4e7580d3-a6df-480b-b837-f8ad160f5a5a"
	}
}
```
* The response will be broadcast by the PID to any display capability topic subscribers using the topic name:
```json
{device_type}/displayCapability/response
```
* The message payload includes the PID's vendor and device ids, a copy of the request id (so that the response can be matched to the request - or ignored by any other devices that subscribe to display capability responses) and a display configuration object.
```json
{
	"displayCapabilityResponse": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "4e7580d3-a6df-480b-b837-f8ad160f5a5a",
		"rtigVersion": "2", // optional,if not present 1 assumed
		"textDisplayConfig": { // text display config
			"charEncdings": [ // array of string values
				"UTF-8"
			],
			"lines": 3, // integer value
			"charsPerLine": 26, // integer value
			"canScroll": true // boolean value
			"canPage": false, // boolean value
			"maxScrollChar": 150, // integer value
		}
	}
}
```
* The elements canScroll and canPage describe the capability of a display. maxScrollChar is the maximum message length which is capable of being scrolled by the display.

## Device Discovery

* The interface supports and promotes the concept of "zero configuration" PID installations using a device discovery procedure. There are two pre-conditions:
	* Each PID must have a unique device identifier and a vendor identifier.
	* Where PSK-level security is enabled on the MQTT broker, each PID must have knowledge of the pre-shared key that will allow it to communicate over the MQTT network; where HTTPS is enabled on the MQTT broker, each PID must have the capability to process Transport Layer Security (TLS).

* To carry out configuration automatically using the device discovery procedure, it is proposed that at first power up, the PID should send a message announcing its arrival on the MQTT network with the topic:
```json
{device_type}/discovery/request
```
* with a simple payload containing its unique vendor and device identifiers and a discovery request id:
```json
{
	"discoveryRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"deviceModel": "ABC DISPLAY", // optional string with model
		"requestId": "1ca491a6-3355-462e-81b5-cac9bbf4941c",
		"tenantId": "UUID" // optional identify customer tenant
	}
}
```
* Where an encrypted device discovery process is being used the discovery request payload has an additional attribute to pass the public key of the display specific key to the CMS. This is used to pass security sensitive credential information back to the display via a RSA or EC public key encryption scheme. It can also be potentially used to share a symmetric signing key.
```json
displayPublicKey: "string representing the PEM format public key"
```
* The displayPublicKey is RSA 2048 bit and transmitted as a CSR in PEM format, without the begin and end lines, and as one string with no line breaks. When creating the CSR set the OU=vendorId/CN=deviceId in the X509 name attributes so that it can be identified easily should it be necessary.

* New versions of this specification will update key length and type as required.

* A discovery server on the network (this could be the CMS or the PID supplier's own discovery service) should then respond with a message with the topic:
```json
{device_type}/discovery/response/{vendor_id}/{device_id}
```
* The vendorId identifier should be the name of display manufacturer. This should be all capitals and be letters and / or numbers, not contain spaces but underscores are allowed.

* The use of deviceId reduces data volumes as without it all devices will receive discovery data they are not interested in.

* The response contains a unique response id in addition to the original request id:
```json
{
	"discoveryResponse": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "1ca491a6-3355-462e-81b5-cac9bbf4941c",
		"responseId": "1bb1caf4-6866-441d-a946-83cf830fdc6d",
		"locationConfig": { // locationConfiguration object
			"localities": [],
			"clusters": []
			"stops": ["4900128738"],
			"locationName": [ // multiLanguageText object
				{
					"lang": "EN",
					"text": "Cardiff Central"
				},
				{
					"lang": "CY",
					"text": "Caerdydd Canolog"
				}
			]
		},
		"vendorConfig": {
			// vendor-specific object
			// this could be provided via the CMS or directly
			// by the vendor's own separate discovery service
		}
	}
}
```
* The discoveryResponse message has an optional attribute to pass to the device the URL for a file containing a CA bundle file in PEM format.

* When the client connects to the MQTT broker it should initially suppress CA validation and ignore any expired or invalid CAs. Once successfully connected a replacement CA bundle shall be provided. This attribute is used to communicate to the device the authorised CAs that should be downloaded and installed, prior to opening a TLS connection with the username/password or client certificate.
```json
caBundle: "https://s3.cms-supplier.com/ca-bundle.pem"
```
* The discoveryResponse message has an additional optional attribute to pass the username and password for the MQTT connection of the display to the CMS, because we need to know this to be able to connect with the appropriate credentials for MQTT topic security. The content of this is [user,pass] encrypted using the public key extracted from the displayPublicKey CSR and encoded as base64.

* The CMS should only return one of mqttUserPass or mqttClientCertificate. Which one is used, will be a project specific requirement.
```json
mqttUserPass: "nZDLA6/eN8jH3tWQg7x0X4CyxrMBohSJHRK"

or

mqttClientCertificate: "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA2RwFFVfEcv1biTouMfPKIYkawjxNEPilRn7WKuQTsOXoVw0kWyWJqnh++H7ht1b7QckgkoSNvbmZvlboNBAmRLiStfgg6U4cOKZzFJrdPytgt6eCnZDLA6/eN8jH3tWQg7x0X4CyxrMBohSJHRK1bg86080Kze1M9muHITRsqNKSv9xv5l1F0ce1OsAVv+FYM1DB+zjtKmg3By3vWYpsAXkPI6wSQ43XbiZ6Z5RZvwF9XKD/nM2/s8pPZXbnNEWVTUI/b1Z6pUIf2BCovHCJLUdXV+apkvmUEsf86Pv7ZCgHWrDDfF+93n7XfttTVeVQdbzs/PbhQ1Alee1QkojW6wIDAQAB"
```
* Finally, the PID can acknowledge that the configuration has been applied by confirming the response id to the discovery service:
```json
{device_type}/discovery/acknowledge
	{
		"discoveryAcknowledge": {
			"discoveryAcknowledge": {
				"vendorId": "VENDOR0001VENDOR0001",
				"deviceId": "ABCD1234567890",
				"responseId": "1bb1caf4-6866-441d-a946-83cf830fdc6d"
			}
	}
}
```
## Request for Content Delivery

* Where a display requires the delivery of a fresh set of content it should use the following requests. Scenarios where this would be used include:
	* The display has completed discovery, started up for the first time and has not yet received any content to display.
	* The display has received a data reset request from the CMS
	* The display has restarted from a power outage
	* The display has received a restart request from the CMS
	* The display needs content provided by a platform other than the primary CMS and the display needs to inform the platform that content should be sent.

* The display will publish to the following topics depending on the data required.
```json
CMS/scheduledDeparture/request
CMS/realtimeDeparture/request
CMS/informationMessage/request
CMS/journeyMessage/request
```
* The payload of this request will be as follows for the request of scheduled data:
```json
{
	"scheduledDepartureRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "ad3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"timestamp": "2022-01-12T10:32:14-00:00",
		"locationConfig": {
			"localities": [],
			"clusters": [],
			"stops": ["4900128738"],
			"locationName": [ // multiLanguageText object
				{
					"lang": "EN",
					"text": "Cardiff Central"
				},
				{
					"lang": "CY",
					"text": "Caerdydd Canolog"
				}
			]
		}
	}
}
```
* The payload of this request will be as follows for the request of real time data:
```json
{
	"realtimeDepartureRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "ad3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"timestamp": "2022-01-12T10:32:14-00:00",
		"locationConfig": {
			"localities": [],
			"clusters": [],
			"stops": ["4900128738"],
			"locationName": [ // multiLanguageText object
				{
					"lang": "EN",
					"text": "Cardiff Central"
				},
				{
					"lang": "CY",
					"text": "Caerdydd Canolog"
				}
			]
		}
	}
}
```
* The payload of this request will be as follows for the request of information messages:
```json
{
	"informationMessageRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "ad3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"timestamp": "2022-01-12T10:32:14-00:00",
		"locationConfig": {
			"localities": [],
			"clusters": [],
			"stops": ["4900128738"],
			"locationName": [ // multiLanguageText object
				{
					"lang": "EN",
					"text": "Cardiff Central"
				},
				{
					"lang": "CY",
					"text": "Caerdydd Canolog"
				}
			]
		}
	}
}
```
* The payload of this request will be as follows for the request of information messages:
```json
{
	"journeyMessageRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "ad3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"timestamp": "2022-01-12T10:32:14-00:00",
		"locationConfig": {
			"localities": [],
			"clusters": [],
			"stops": ["4900128738"],
			"locationName": [ // multiLanguageText object
				{
					"lang": "EN",
					"text": "Cardiff Central"
				},
				{
					"lang": "CY",
					"text": "Caerdydd Canolog"
				}
			]
		}
	}
}
```
* The Display will subscribe to these topics:
```json
CMS/scheduledDeparture/response/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
CMS/realtimeDeparture/response/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
CMS/informationMessage/response/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
CMS/journeyMessage/response/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
```
* When the request has been processed, the CMS or the third-party content provider will send a response of the form
```json
{
	"realtimeDepartureResponse": {
		"requestId": "ad3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"status": "DONE",
		"timestamp": "2022-01-12T10:32:14-00:00"
	}
}
```
* The element status signifies DONE when fresh content has been delivered to the topic(s) or NO_DATA when there is no content available for the requested location.

* This mechanism ensures the CMS has visibility of the data exchange and can to a limited extent track performance of third-party content providers.

## Departure Cleardown

* Timely cleardown of expired departures is critical to the success of any RTPI system.

* The protocol therefore requires cleardown request and response messages to contain a timestamp, so that the CMS (or other MQTT-connected reporting services) can measure the latency between a CMS requesting a cleardown and a PID confirming that it has actioned the request.

* Requests should be sent using the following topic name:
```json
CMS/cleardown/request/{locality_id}/{cluster_id}/{stop_id}/{vendor_id}/{device_id}
```
* The payload is very simple and only contains a departure id, a cleardown reason and a timestamp:
```json
{
	"cleardownRequest": {
		"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
		"cleardownReason": "DEPARTED",
		"timestamp": "2022-01-12T10:32:14-00:00"
	}
}
```
* The response is sent using the topic name:
```json
{device_type}/cleardown/response
	{
		"cleardownResponse": {
			"vendorId": "VENDOR0001",
			"deviceId": "ABCD1234567890",
			"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
			"timestamp": "2022-01-12T10:32:19-00:00"
		}
	}
```
## Status Reporting

* Status requests and responses follow a similar pattern to display capability requests and responses; the request topic is as follows:
```json
{device_type}/status/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"statusRequest": {
		"requestId": "a2047a18-75aa-4bd1-a281-5fbae13d6cf1"
	}
}
```
* The response will be broadcast by the PID to any status topic subscribers using the topic name:
```json
{device_type}/status/response
```
* The message payload includes the PID's vendor and device ids, a copy of the request id (so that the response can be matched to the request - or ignored by any other devices that subscribe to status responses) and a status report object:
```json
{
	"statusResponse": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": " a2047a18-75aa-4bd1-a281-5fbae13d6cf1", // taken from request
		"statusReport": { // status report object
			"firmwareVersion": "XYZ.789.001.A",
			"operatingSystem": "Homebrew Linux 3.11",
			"upSince": "2020-11-17T06:30:00-00:00",
			"memoryUsage": "63%",
			"loadAverage": "78%", // one minute load average
			"displayOverride": "RT_DEP" // optional, if a display override is active, set to enum
		}
	}
}
```
## Text Snapshot

* Status requests and responses follow a similar pattern to display capability and status reporting requests and responses; the request topic is as follows:
```json
{device_type}/textSnapshot/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"textSnapshotRequest": {
		"requestId": "ae1c5b90-c0f1-4b64-8493-eea6db12eeea"
	}
}
```
* The response will be broadcast by the PID to any text snapshot topic subscribers using the topic name:
```json
{device_type}/textSnapshot/response
```
* The message payload includes the PID's vendor and device ids, a copy of the request id (so that the response can be matched to the request - or ignored by any other devices that subscribe to text snapshot responses) and a line-by-line representation of the text that is currently being displayed:
```json
{
	"textSnapshotResponse": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "ae1c5b90-c0f1-4b64-8493-eea6db12eeea", // taken from request
		"lines": [ // array of string values
			"123A East Killingham 1025",
			"X17 Batchampton 7min",
			"No delays reported "
		]
	}
}
```
# Use Cases

## Message handling behaviour

* There are separate enumerations for Message display style and Message priority type in the protocol, handling of these requires some interpretation. This section outlines the assumptions in behaviour made by the working group.

* Full-Screen Message
	* Full-screen messages take precedence over any other content currently displayed on the screen. This means that any real-time information or other media scheduled for display will be temporarily overridden by the message while it is active, ensuring it is the sole content visible.
	* In cases where the entire message cannot be accommodated due to character limitations on the display, it will be paged through until the complete message has been shown. Once displayed in full, the message will cyclically loop from the beginning until its scheduled expiry time or until it is manually cancelled.
	* When multiple messages are sent to the display, the highest-priority message will take precedence. If two active messages share the same priority, the display will alternate between the first and second messages until one of the messages concludes or is cancelled. If the other message is still active, it will remain on display.
	* Emergency full screen messages are displayed exclusively, any lower priority messages are not shown at all, and bottom line messages are not shown. In the event of multiple active emergency messages, they will be paged in the order received by the display

* Overline
	* These behave as full screen with the exception an overline message leaves bottom line available for other messages where screens do not otherwise differentiate, for example many LED displays.

* Bottom Line Message
	* Each display can be customised to showcase a specific maximum number of messages simultaneously. This setting dictates the maximum number of messages, regardless of priority level, that can appear on the display at any given time.
	* Emergency Messages: When emergency messages are dispatched, they are exclusively displayed on the bottom line of the screen. In the event of multiple active emergency messages, they will be paged in the order received by the display, adhering to the configured active message limit.

* Message Priority
	* Message priority is set through the use of messagePriority.
	* Messages with a messagePriority of EMERGENCY will take predicence over all other messages.
	* Messages with a messagePriority of STANDARD are enhanced with messageActionPriority which shall be a positive integer, where 1 is the highest priority and the lowest priority being 99.
	* In the absence of emergency messages, messages are displayed based on their relative messageActionPriority and displayed in the order they are received.
	* If there are no active emergency messages and the number of messageActionPriority=1 messages haven't reached a configured limit (if available), messageActionPriority=2 messages will be displayed up a configured message limit (if available). These will be displayed alongside any messageActionPriority=1 messages, with messageActionPriority=1 messages appearing first within the message cycle.
	* This process continues for messages with higher messageActionPriority until the configurable limits for number of messages displayed is reached.
	* This system ensures that messages are displayed according to their priority level while adhering to any message limit set for each display unit.

## Quay reallocation

* Where a Quay is reallocated in real time the working group recommends the following behaviour.

* For example a departure planned for quay 7 is reallocated to quay 8.

* In this situation it is expected that a new Monitored Call is pushed to stop_id for quay 8 and the existing Monitored Call call for the original stop_id for quay 7 is updated to be quay 8 as well. This allows a message to be displayed against the departure on quay 7.

* A journeyMessage can then be used to display a message on the display for quay 7 "This service now departs from stand 8".





