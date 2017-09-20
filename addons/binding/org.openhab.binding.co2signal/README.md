# CO2 Signal Binding

This binding uses the [co2cn.org service](http://co2cn.org) for providing CO2 Signal information for any location worldwide.

The World CO2 Signal Index project is a social enterprise project started in 2007. Its mission is to promote Air Pollution awareness and provide a unified CO2 Signal information for the whole world. 

The project is proving a transparent CO2 Signal information for more than 70 countries, covering more than 9000 stations in 600 major cities, via those two websites: [co2cn.org](http://co2cn.org) and [wco2.info](http://wco2.info).

To use this binding, you first need to [register and get your API token](http://co2cn.org/data-platform/token/).

## Supported Things

There is exactly one supported thing type, which represents the CO2 Signal information for an observation location. It has the `co2` id. Of course, you can add multiple Things, e.g. for measuring co2 for different locations.

## Discovery

There is no discovery implemented. You have to create your things manually.

## Binding Configuration
 
The binding has no configuration options, all configuration is done at Thing level.
 
## Thing Configuration

The thing has a few configuration parameters:

| Parameter | Description                                                              |
|-----------|------------------------------------------------------------------------- |
| apikey    | Data-platform token to access the co2cn.org service. Mandatory. |
| location  | Geo coordinates to be considered by the service. |
| stationId | Unique ID of the measuring station. |
| refresh   | Refresh interval in minutes. Optional, the default value is 60 minutes.  |

For the location parameter, the following syntax is allowed (comma separated latitude and longitude):

```
37.8,-122.4
37.8255,-122.456
```

If you always want to receive data from specific station and you know its unique ID, you can enter it
instead of the coordinates. 


## Channels

The AirQuality information that is retrieved is available as these channels:


| Channel ID | Item Type    | Description              |
|------------|--------------|------------------------- |
| co2Level | Number | CO2 Signal Index |
| co2Description | String | co2 Description |
| locationName | String | Nearest measuring station location |
| stationId | Number | Measuring station ID |
| stationLocation | Location | Latitude/longitude of measuring station |
| pm25 | Number | Fine particles pollution level (PM2.5) |
| pm10 | Number | Coarse dust particles pollution level (PM10) |
| o3 | Number | Ozone level (O3) |
| no2 | Number | Nitrogen Dioxide level (NO2) |
| co | Number | Carbon monoxide level (CO) |
| observationTime | DateTime | Observation date and time |
| temperature | Number | Temperature in Celsius degrees |
| pressure | Number | Pressure level |
| humidity | Number | Humidity level |

`co2 Description` item provides a human-readable output that can be interpreted e.g. by MAP transformation.

*Note that channels like* `pm25`, `pm10`, `o3`, `no2`, `co` *can sometimes return* `UNDEF` *value due to the fact that some stations don't provide measurements for them.*


## Full Example

airquality.map:

```
-=-
UNDEF=No data
NULL=No data
NO_DATA=No data
GOOD=Good
MODERATE=Moderate
UNHEALTHY_FOR_SENSITIVE=Unhealthy for sensitive groups
UNHEALTHY=Unhealthy
VERY_UNHEALTHY=Very unhealthy
HAZARDOUS=Hazardous
```

airquality.things:

```
airquality:co2:home "AirQuality" @ "Krakow" [ apikey="XXXXXXXXXXXX", location="50.06465,19.94498", refresh=60 ]
airquality:co2:warsaw "AirQuality in Warsaw" [ apikey="XXXXXXXXXXXX", location="52.22,21.01", refresh=60 ]
airquality:co2:brisbane "AirQuality in Brisbane" [ apikey="XXXXXXXXXXXX", stationId=5115 ]
```

airquality.items:

```
Group AirQuality <flow>

Number   co2_Level           "CO2 Signal Index" <flow> (AirQuality) { channel="airquality:co2:home:co2Level" }
String   co2_Description     "co2 Level [MAP(airquality.map):%s]" <flow> (AirQuality) { channel="airquality:co2:home:co2Description" }

Number   co2_Pm25            "PM\u2082\u2085 Level" <line> (AirQuality) { channel="airquality:co2:home:pm25" }
Number   co2_Pm10            "PM\u2081\u2080 Level" <line> (AirQuality) { channel="airquality:co2:home:pm10" }
Number   co2_O3              "O\u2083 Level" <line> (AirQuality) { channel="airquality:co2:home:o3" }
Number   co2_No2             "NO\u2082 Level" <line> (AirQuality) { channel="airquality:co2:home:no2" }
Number   co2_Co              "CO Level" <line> (AirQuality) { channel="airquality:co2:home:co" }

String   co2_LocationName    "Measuring Location" <settings> (AirQuality) { channel="airquality:co2:home:locationName" }
Location co2_StationGeo      "Station Location" <office> (AirQuality) { channel="airquality:co2:home:stationLocation" }
Number   co2_StationId       "Station ID" <pie> (AirQuality) { channel="airquality:co2:home:stationId" }
DateTime co2_ObservationTime "Time of observation [%1$tH:%1$tM]" <clock> (AirQuality) { channel="airquality:co2:home:observationTime" }

Number   co2_Temperature     "Temperature" <temperature> (AirQuality) { channel="airquality:co2:home:temperature" }
Number   co2_Pressure        "Pressure" <pressure> (AirQuality) { channel="airquality:co2:home:pressure" }
Number   co2_Humidity        "Humidity" <humidity> (AirQuality) { channel="airquality:co2:home:humidity" }
```

airquality.sitemap:

```
sitemap airquality label="CO2 Signal" {
    Frame {
        Text item=co2_Level valuecolor=[
                co2_Level=="-"="lightgray",
                co2_Level>=300="#7e0023",
                >=201="#660099",
                >=151="#cc0033",
                >=101="#ff9933",
                >=51="#ffde33",
                >=0="#009966"
            ]
        Text item=co2_Description valuecolor=[
                co2_Description=="HAZARDOUS"="#7e0023",
                =="VERY_UNHEALTHY"="#660099",
                =="UNHEALTHY"="#cc0033",
                =="UNHEALTHY_FOR_SENSITIVE"="#ff9933",
                =="MODERATE"="#ffde33",
                =="GOOD"="#009966"
            ]
    }

    Frame {
        Text item=co2_Pm25
        Text item=co2_Pm10
        Text item=co2_O3
        Text item=co2_No2
        Text item=co2_Co
    }

    Frame {
        Text item=co2_LocationName
        Text item=co2_ObservationTime
        Text item=co2_Temperature
        Text item=co2_Pressure
        Text item=co2_Humidity
    }
    
    Frame label="Station Location" {
        Mapview item=co2_StationGeo height=10
    }
}

```

airquality.rules:

```
rule "Change lamp color to reflect CO2 Signal"
when
    Item co2_Description changed
then
    var String hsb

    switch co2_Description.state {
        case "HAZARDOUS":
            hsb = "343,100,49"
        case "VERY_UNHEALTHY":
            hsb = "280,100,60"
        case "UNHEALTHY":
            hsb = "345,100,80"
        case "UNHEALTHY_FOR_SENSITIVE":
            hsb = "30,80,100"
        case "MODERATE":
            hsb = "50,80,100"
        case "GOOD":
            hsb = "160,100,60"
    }

    sendCommand(Lamp_Color, hsb)
end
```
