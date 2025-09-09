![RTIG Logo with strapline](images/rtig_document_header_logo.png)

# CMS to PID Interface Protocol

## Part 3 - Message Types for Graphical Displays


## Status of this document

This document is Published.

If there are any comments or feedback arising from the review or use of this document, please contact us at secretariat@rtig.org.uk


# Message Types for Graphical Displays

## Multimedia References

* It is the intention that binary data (audio, video, images, etc) required by more advanced PIDs will not be encoded directly into MQTT message payloads. Instead, it is proposed that external URLs are provided as part of the message payload and that a PID subsequently retrieves multimedia data from the internet (or private network) directly using the following JSON structure:
```json
{
"multimediaRef": {
	"contentType": "video/mp4",
	"url": "https://suppliername.com/content/video/test.mp4",
	"md5": "79054025255fb1a26e4bc422aef54eb4",
	"contentExpiry": "2022-03-01T00:00:00-00:00"
}
```
* Content types must be standard MIME types, from the list maintained by IANA (RFC6838 and RFC4855).

* The MD5 checksum should be used to verify that the content has not been corrupted during transfer.

* It should be noted that while MD5 is an old algorithm that is no longer secure for encryption purposes (having been compromised by multiple vulnerabilities and subsequently superseded for this purpose by algorithms such as SHA-256), it is still widely used and accepted as the de facto standard for file hashing, with implementations readily available in most environments.

* If the device referencing the multimedia content has already downloaded the same content previously, it need only be re-download if the original content expiry timestamp has passed.

* Known media data should be synchronised on display startup or playlists may be downloaded in the background as required.

* Assets may be display type specific. The display needs to request assets for that display. (as an example, e-paper typically may use svg assets not png).

## Graphical Snapshot

* Based on the concepts of text snapshots (see RTIGT047 Part 2) and multimedia references (above), the graphical snapshot message pattern is defined as follows.

* Request topic:
```json
{device_type}/graphicalSnapshot/request/{vendor_id}/{device_id}
```
* Request payload:
```json
{
	"graphicalSnapshotRequest": {
		"requestId": " 7d79665a-ea06-4648-810b-9f689bf87a14",
		"uploadUrl": "https://suppliername.com/content/snapshot/202404201556/ABCD1234567890.png?signed=1231231234123d"
	}
}
```
* Upon receiving a graphical snapshot request, the PID will upload a snapshot image to an internet-accessible location defined by uploadUrl and then issue a response to any graphical snapshot topic subscribers using the topic name:
```json
{device_type}/graphicalSnapshot/response
```
* The message payload includes the PID’s vendor and device ids, a copy of the request id (so that the response can be matched to the request – or ignored – by any other devices that subscribe to graphical snapshot responses) and a multimedia reference to the snapshot image that has been uploaded to an internet-accessible location:
```json
{
"graphicalSnapshotResponse": {
	"vendorId": "VENDOR0001",
	"deviceId": "ABCD1234567890",
	"requestId": "7d79665a-ea06-4648-810b-9f689bf87a14",
	"multimediaRef": {
		"contentType": "image/png",
		"url": "https://suppliername.com/content/snapshot/202112171556/A BCD1234567890.png",
		"md5": "79054025255fb1a26e4bc422aef54eb4",
		"contentExpiry": "2021-12-18T00:00:00-00:00"
	}
}
```
## Playlists

* A playlist is a list of media elements (usually of the same media type, although mixed media types are permitted) that should be rendered sequentially *within a single display component*.

* Playlists are specified as an array of elements containing a media type, a display duration and a display order:
```json
[
	{
		"multimediaRef": {...}, // multimedia ref object
		"duration": 25, // integer number of seconds
		"order": 1 // integer
	},
	{
		"multimediaRef": {...}, // multimedia ref object
		"duration": 10, // integer number of seconds
		"order": 2 // integer
	},
	...
]
```
## Graphical Display Templates

* A template defines a full-screen layout and must contain details of every display component (e.g. departures, advertising, message ticker, etc) that is included in the layout.

* All PIDs must support the capability to use templates

* Each component must display content of one of the following media types:
```json
[
IMAGE,
VIDEO,
PLAYLIST,
displayStatusType, // e.g. RT_DEP, MIXED_ARRDEP, etc
INFO_MSG,
RSS_FEED
]
```
* Each component is defined using the following basic data structure:
```json
"graphicalDisplayComponent": {
	"top": 0, // integer value in pixels
	"left": 0, // integer value in pixels
	"bottom": 800, // integer value in pixels
	"right": 1200, // integer value in pixels
	"mediaType": "RT_DEP", // enum value
	"contentRaw": { ... }, // playlist or null
	"contentRefs": [] // array of multimediaRef
}
```
* Where a component is of media type PLAYLIST, contentRaw must be a playlist object (as defined above) and contentRef must be an empty array; for all other types, contentRaw must be null and contentRef must not be empty. Where a component is of type IMAGE, VIDEO or RSS_FEED, contentRef must contain a single multimedia reference; for other media types, multimedia references to an arbitrary combination of HTML, JavaScript and CSS files can be supplied. It should be noted that successful rendering of a particular referenced file type will always be subject to a PID’s capabilities – see Graphical Display Capability below.

* It is recommended that all required files must be specifically included in the contentRef array, and that a PID must not attempt to follow any external URLs embedded directly within any source files, in order to provide a basic level of security against the injection of malicious code.

* A template is defined using the following data structure:
```json
"graphicalDisplayTemplate": {
	"templateId": "09d57be3-253a-4670-8513-3bb4c2ba2883",
	"components": [
		{ ... }, // graphicalDisplayComponent
		{ ... }, // graphicalDisplayComponent
		...
	]
}
```
* The CMS (or another service on the MQTT network) can request definitions of the templates currently being used by a PID using the following request topic:
```json
{device_type}/getAllTemplates/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"getAllTemplatesRequest": {
		"requestId": "5412b85a-1083-4eea-bf54-26c7bca709c1"
	}
}
```
* In response, the PID should broadcast to any subscribers using the topic name:
```json
{device_type}/getAllTemplates/response
```
* The message payload is an array of graphical display template definitions, together with a reference to the request id:
```json
{
	"getAllTemplatesResponse": {
		"requestId": "5412b85a-1083-4eea-bf54-26c7bca709c1",
		"templates": [
			"graphicalDisplayTemplate": { ... },
			"graphicalDisplayTemplate": { ... },
			...
		]
	}
}
```
* The CMS can create/replace definitions of the templates currently being used by a PID using the following request topic:
```json
{device_type}/setAllTemplates/request/{vendor_id}/{device_id}
```
* The payload is the same as a getAllTemplatesResponse:
```json
{
	"setAllTemplatesRequest": {
		"requestId": "7192044e-37ec-4a3c-a4ad-652b780d3c0b",
		"templates": [
			"graphicalDisplayTemplate": { ... },
			"graphicalDisplayTemplate": { ... },
			...
		]
	}
}
```
* Note that the payload must contain the full set of all required templates for the PID as it will overwrite all previous template definitions.

* In response, the PID should send a confirmation that the templates were received and updated successfully, using the topic:
```json
{device_type}/setAllTemplates/response
```
* The payload is a repetition of the accepted template ids. It is the responsibility of the PID to reject unsupported templates by excluding their ids from the response payload:
```json
{
	"setAllTemplatesResponse": {
		"requestId": "7192044e-37ec-4a3c-a4ad-652b780d3c0b",
		"templateIds": [
			"d1613512-2baa-427e-b0d4-c5baa665ef1f",
			"defa79c9-93f9-4f40-b61e-9f473e1b9d59",
			...
		]
	}
}
```
* The CMS (or another service on the MQTT network) can request the definition of the active template for a PID using the following request topic:
```json
{device_type}/getActiveTemplate/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"getActiveTemplateRequest": {
		"requestId": "20cda746-b9cc-4202-860b-c96e5717c609"
	}
}
```
* In response, the PID should broadcast to any subscribers using the topic name:
```json
{device_type}/getActiveTemplate/response
```
* The message payload is a single graphical display template definition, together with a reference to the request id:
```json
{
	"getActiveTemplateResponse": {
		"requestId": "20cda746-b9cc-4202-860b-c96e5717c609",
		"graphicalDisplayTemplate": { ... }
	}
}
```
* The CMS can set which template is currently being used by a PID using the following request topic:
```json
{device_type}/setActiveTemplate/request/{vendor_id}/{device_id}
```
* The payload consists of a template id and a request id:
```json
{
	"setActiveTemplateRequest": {
		"requestId": "7d3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"templateId": "d1613512-2baa-427e-b0d4-c5baa665ef1f"
	}
}
```
* In response, the PID should confirm whether or not the active template was updated, using the topic:
```json
{device_type}/setActiveTemplate/response
```
* The payload is a repetition of the received template ids:
```json
{
	"setActiveTemplateResponse": {
		"requestId": "7d3ccebe-0611-4d4c-8ab3-7e1e6882b08b",
		"result": "OK" // enum value
	}
}
```
* In each case, the result should be a value from the following enumeration:
```json
[
OK,
NOT_SUPPORTED,
NOT_FOUND,
ERROR
]
```
## Carousels

* A carousel is effectively a playlist of templates. A carousel is defined in a similar manner to a playlist – i.e. as an array of elements – the key difference being that each element in the array is a template identifier rather than a multimedia reference:
```json
[
	{
		"templateId": "09d57be3-253a-4670-8513-3bb4c2ba2883",
		"duration": 30, // integer number of seconds
		"order": 1 // integer
	},
	{
		"templateId": "e8681125-b196-4034-86e0-8f0c50d12a87",
		"duration": 30, // integer number of seconds
		"order": 2 // integer
	},
	...
]
```
* It is not necessary for all connected PIDs to support the capability to use carousels; this is likely to be a key differentiating factor between PID models and suppliers. Where carousel support is provided, a further differentiating factor will be how transitions between carousels are managed. For example, is the PID able to maintain the current state of a scrolling text ticker or media playlist that is common to sequential templates within a carousel – or will the content for such components be restarted whenever the template changes?

* The CMS (or another service on the MQTT network) can request definitions of the carousels currently being used by a PID using the following request topic:
```json
{device_type}/getAllCarousels/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"getAllCarouselsRequest": {
		"requestId": "7ebdcb4c-cad0-4202-a59e-5a0bfd24f5c7"
	}
}
```
* In response, the PID should broadcast to any subscribers using the topic name:
```json
{device_type}/getAllCarousels/response
```
* The message payload is an array of graphical display carousel definitions, together with a reference to the request id:
```json
{
	"getAllCarouselsResponse": {
		"requestId": "7ebdcb4c-cad0-4202-a59e-5a0bfd24f5c7",
		"carousels": [
			"graphicalDisplayCarousel": { ... },
			"graphicalDisplayCarousel": { ... },
			...
		]
	}
}
```
* The CMS can create/replace definitions of the carousels currently being used by a PID using the following request topic:
```json
{device_type}/setAllCarousels/request/{vendor_id}/{device_id}
```
* The payload is the same as a getAllCarouselsResponse:
```json
{
	"setAllCarouselsRequest": {
		"requestId": "e7034aa3-f7f0-4a59-b486-493eb368672d",
		"carousels": [
			"graphicalDisplayCarousel": { ... },
			"graphicalDisplayCarousel": { ... },
			...
		]
	}
}
```
* Note that the payload must contain the full set of all required carousels for the PID as it will overwrite all previous carousel definitions.

* In response, the PID should send a confirmation that the carousels were received and updated successfully, using the topic:
```json
{device_type}/setAllCarousels/response
```
* The payload is a repetition of the received carousel ids:
```json
{
	"setAllCarouselsResponse": {
		"requestId": "e7034aa3-f7f0-4a59-b486-493eb368672d",
		"carouselIds": [
			"4b4eb833-808a-435f-ac64-4a20cdf041d8",
			"a910c96b-d599-4ed6-9de1-1b4f91eede6c",
			...
		]
	}
}
```
* The CMS (or another service on the MQTT network) can request the definition of the active carousel for a PID using the following request topic:
```json
{device_type}/getActiveCarousel/request/{vendor_id}/{device_id}
```
* The message payload simply contains a request id:
```json
{
	"getActiveCarouselRequest": {
		"requestId": "f9ec6b7b-9809-48c6-a5d2-9924d9b87ccf"
	}
}
```
* In response, the PID should broadcast to any subscribers using the topic name:
```json
{device_type}/getActiveCarousel/response
```
* The message payload is a single graphical display carousel definition, together with a reference to the request id:
```json
{
	"getActiveCarouselResponse": {
		"requestId": "f9ec6b7b-9809-48c6-a5d2-9924d9b87ccf",
		"graphicalDisplayCarousel": { ... }
	}
}
```
* The CMS can set which carousel is currently being used by a PID using the following request topic:
```json
{device_type}/setActiveCarousel/request/{vendor_id}/{device_id}
```
* The payload consists of a carousel id and a request id:
```json
{
	"setActiveCarouselRequest": {
		"requestId": "10af973f-3213-436e-9db0-99b7a82f12d8",
		"carouselId": "4b4eb833-808a-435f-ac64-4a20cdf041d8"
	}
}
```
* In response, the PID should confirm whether or not the active carousel was updated, using the topic:
```json
{device_type}/setActiveCarousel/response
```
* The payload is a repetition of the received carousel ids:
```json
{
	"setActiveCarouselResponse": {
		"requestId": "10af973f-3213-436e-9db0-99b7a82f12d8",
		"result": "OK" // enum value from [ OK, NOT_FOUND, ERROR ]
	}
}
```
## Graphical Display Capability

* The graphical display capability data structure type includes details of supported media types, screen dimensions and orientation, using the following data structure:
```json
"graphicalDisplayConfig": {
	"screenDimensions": {
		"x": 1920, // integer value in pixels
		"y": 1080 // integer value in pixels
	},
	"screenOrientation": LANDSCAPE, // enum [LANDSCAPE, PORTRAIT]
	"screenAspectRatio": "16:9", // string value using colon
	"iconsSupported": true, // boolean value
	"imagesSupported": true, // boolean value
	"videosSupported": true, // boolean value
	"playlistsSupported": true, // boolean value
	"carouselsSupported": true, // boolean value
	"componentContinuitySupported": true, // boolean value
	"htmlSupported": true, // boolean value
	"javaScriptSupported": true, // boolean value
	"cssSupported": true // boolean value
}
```
* In theory, only one of screenDimensions, screenOrientation and screenAspectRatio should be required, but including all three is recommended for end-user readability. Where there is conflict between these values, screenDimensions shall always take precedence.

* componentContinuitySupported relates to the ability to preserve the state of a component between template changes (regardless of whether a new template has been specifically requested or has been changed as part of a carousel). For example, where sequential templates contain a scrolling text or media playlist component (in the same location on the screen) will the component state be preserved during a template transition or will the ticker text or media playlist be restarted.

* As described in RTIGT047 Part 2, a display capability request will typically be sent by a CMS using a topic name constructed as follows:
```json
{device_type}/displayCapability/request/{vendor_id}/{device_id}
```
* As before, the message payload simply contains a request id:
```json
{
	"displayCapabilityRequest": {
		"requestId": "f3d68878-3b5a-4f6f-ba08-b4ffd5b0f1d7"
	}
}
```
* Once again, the response will be broadcast by the PID to any display capability topic subscribers using the topic name:
```json
{device_type}/displayCapability/response
```
* However, for a multimedia display, the message payload includes a graphicalDisplayConfig object in addition to the contents of the equivalent message for a text display:
```json
"displayCapabilityResponse": {
	"vendorId": "VENDOR0002",
	"deviceId": "ABCD1234567890",
	"requestId": "f3d68878-3b5a-4f6f-ba08-b4ffd5b0f1d7",
	"textDisplayConfig": { ... },
	"graphicalDisplayConfig": { ... }
}
```
## Graphical Assets Request

* Icon media files can be downloaded by the display using a request response mechanism at display startup and dynamically as required.

* It is expected that the CMS will supply the correctly sized and formatted icon files associated with the display that requested it. This may include SVG format files for e-paper displays. Bitmaps for LED displays, and PNG or JPG of appropriate resolution for TFT displays.

* Graphical assets are stored as iconType and iconRef. The iconType enumeration may contain the following values
```json
[
	OPERATOR, // operator icon
	LINE, // line icon
	ROUTE, // route icon
	POI, // Point of Interest icon
	FEATURE // Feature icon
]
```
* A graphicalAssetsRequest is sent using a topic name constructed as follows
```json
{device_type}/graphicalAssets/request/{vendor_id}/{device_id}
```
* The request contains the body as follows
```json
{
	"graphicalAssetsRequest": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"requestId": "1ca491a6-3355-462e-81b5-cac9bbf4941c",
		"iconType": "" //optional
		"iconRef": "" //optional
	}
}
```
* The iconType and the iconRef are optional. If they are not supplied in the request (key not present, blank or set to null) then a full set of icons appropriate to that display is delivered.

* The iconType may be supplied on its own without an iconRef. For example iconType is set to OPERATOR to receive a full set of operator logos.

* The iconRef must be unique to the project, and may not be used more than once across different iconType

* The iconRef may be requested on its own if the display finds that a specific icon is missing.

* The response topic is constructed as follows
```json
{device_type}/graphicalAssets/response/{vendor_id}/{device_id}
```
* The response contains the body as follows
```json
{
	"graphicalAssetsResponse": {
		"vendorId": "VENDOR0001",
		"deviceId": "ABCD1234567890",
		"responseId": "1bb1caf4-6866-441d-a946-83cf830fdc6d",
		"icons": [
			{
				"iconType": "OPERATOR",
				"iconRef": "SCE",
				"multimediaRefs": [
					{
						"contentType": "image/png",
						"url": "https://suppliername.com/content/images/sce-en.png",
						"md5": "79054025255fb1a26e4bc422aef54eb4",
						"contentExpiry": "2022-03-01T00:00:00-00:00",
						"lang": "EN"
					},
					{
						"contentType": "image/png",
						"url": "https://suppliername.com/content/images/sce-cy.png",
						"md5": "79054025255fb1a26e4bc422aef54eb4",
						"contentExpiry": "2022-03-01T00:00:00-00:00",
						"lang": "CY"
					}
				]
			},
			{
				"iconType": "OPERATOR",
				"iconRef": "NXEM",
				"multimediaRefs": [
					{
						"contentType": "image/png",
						"url": "https://suppliername.com/content/images/sce-en.png",
						"md5": "79054025255fb1a26e4bc422aef54eb4",
						"contentExpiry": "2022-03-01T00:00:00-00:00",
						"lang": "EN"
					}
				]
			},
			{
				"iconType": "POI",
				"iconRef": "railway",
				"multimediaRefs": [
					{
						"contentType": "image/png",
						"url": "https://suppliername.com/content/images/railway-en.png",
						"md5": "79054025255fb1a26e4bc422aef54eb4",
						"contentExpiry": "2022-03-01T00:00:00-00:00",
						"lang": "EN"
					}
				]
			}
		]
	}
}
```
* On receiving the icon multimediaRefs, the display should download the referenced files and make them available for use.

* When the contentExpiry date and time is reached, the display should request a new set of graphical Assets.

## Icon Support

* Since graphical displays are able to display images, the scheduledDeparture, realtimeDeparture and informationMessage message types (as defined in RTIGT047 Part 2) are extended to include support for the addition of one or more icons for a given departure or message, using the following array structure:
```json
icons:
[
	{
		"iconRef": "NXEM", // iconRef
		"priority": 1 // integer - optional
	},
	{
		"iconRef": "railway", // iconRef
		"priority": 2 // integer - optional
	},
	...
]
```
* In each case, an instance of an icon is defined via an iconRef and an optional priority. As with all icon references, the intention is that the asset is downloaded and cached until the expiry time is reached – thereby avoiding repeated downloads each time an icon needs to be displayed.

* The display is aware of the type of icon because this was provided as part of the graphicalAssetsResponse structure, so it is only necessary to provide an iconRef and optional priority in the content message.

* Example of a scheduled departure message containing icons, showing the icon information for each departure alongside the targeted vehicle journey and targeted call definitions:
```json
{
	"scheduledDepartures": [
		{
			"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
			"vehicleMode": "BUS",
			"targetedVehicleJourney": { ... }, // journey definition
			"targetedCall": { ... }, // targeted call definition
			"icons": [
				{
					"iconRef": "NXEM", // iconRef
					"priority": 1 // integer - optional
				},
				{
					"iconRef": "railway", // iconRef
					"priority": 2 // integer - optional
				},
				...
			],
		},
		...
	]
}
```
* The agreed logos are synchronised during display startup via the graphicalAsset request mechanism.

* Operator and service logos are handled in the display template – if the display can handle the graphic and the logo file is available then it shall be used.

* Example of a real-time departure message containing icons, showing the icon information for each departure alongside the monitored vehicle journey and monitored call definitions:
```json
{
	"realTimeDeparture": [
		{
			"departureId": "fc867e8a-4e9c-465d-847d-6bf2fb37977f",
			"predictionTime": "2004-12-17T09:25:46-05:00",
			"vehicleMode": "BUS",
			"monitoredVehicleJourney": { ... }, // journey definition
			"progressStatus": "ON-TIME", // enum value
			"occupancyLevel": "SEATS-AVAILABLE", // enum value
			"monitoredCall": { ... }, // monitored call definition
			"icons": [
				{
					"iconRef": "NXEM", // iconRef
					"priority": 1 // integer
				},
				{
					"iconRef": "railway", // iconRef
					"priority": 2 // integer
				},
				...
			],
		},
		...
	]
}
```
* Example of an information message containing icons, alongside the message text definition:
```json
{
	"informationMessage": {
		"messageText": [ ... ],
		"earliestDisplayTime": "2022-02-17T10:00:00-00:00",
		"latestDisplayTime": "2022-02-17T17:00:00-00:00",
		"messagePriority": "MEDIUM", // message priority enum
		"displayStyle": "SCROLL", // message display style enum
		"messageId": UUID,
		"icons": [
			{
				"iconRef": "NXEM", // iconRef
				"priority": 1 // integer
			},
			{
				"iconRef": "railway", // iconRef
				"priority": 2 // integer
			},
			...
		]
	}
}
```
* An information messageText may contain placeholders within the text to reference icons which can be displayed inline. This is structured as %icon:iconRef% within the text. An example would be
```json
"messageText": "Change here for %icon:railway% to continue your journey"
```
* The messageText in a journeyMessage may also contain placeholders for icon substitution

* No limit is imposed on icon dimensions, number of colours or file size by the protocol, but it should be recognised that icons are only intended for use within text fields; where larger images (e.g. corporate logos or advertising) are required, image (or playlist) components should be used within a template.


