# SOS Observation Injector

The SOS Observation Injector is a Java toolkit that can be used to enter sensor observations
into an [IOOS customized](https://code.google.com/p/ioostech/) 
[52&deg;North SOS](http://52north.org/communities/sensorweb/sos/).

Applications using this toolkit should provide implementations of StationRetriever and ObservationRetriever 
that properly construct data objects (SosStation, SosSensor, ObservationCollection, etc) for the targeted
sensor provider. Alternatively, ObservationCollections can be constructed and inserted manually.

## Installation
The only requirements for this project are that Java 1.6 or greater is installed and Maven is used to build 
the project. All other dependencies are downloaded via Maven.

## Maven Projects
To add this project to a Maven project insert the below code into your pom.xml file. 

    ....
    <repositories>
      <repository>
        <id>axiom_public_releases</id>
        <name>Axiom Releases</name>
        <url>http://nexus.axiomalaska.com/nexus/content/repositories/public/</url>
      </repository>
    </repositories>
    ....
    <dependencies>
      ....
      <dependency>
        <groupId>com.axiomalaska</groupId>
        <artifactId>sos-injection</artifactId>
        <version>VERSION_HERE</version>
      </dependency>
      ....
    </dependencies>

## Data Classes

### ObservationUpdater
The ObservationUpdater has one method called “update”. This method uses all of the SosStations pulled 
from the StationRetriever to request observations from the ObservationRetriever object. These observations 
are then used to update the SOS with the use of an ObservationSubmitter object. The ObservationSubmitter 
object formats and submits the data to the SOS. 

### StationRetriever
The StationRetriever object is used to fetch the SosStations that are needed to update the SOS. The interface 
for a StationRetriever is a single method called “getStations” which returns a List of SosStations. An 
example of a StationRetriever is located at com.axiomalaska.sos.cnfaic.CnfaicStationRetriever. From looking 
at the CnfaicStationRetriever class, we can see that each SosStation is hard-coded and built in this file. 
This interface would typically be used to pull the station’s data from a database. The SosStations would 
then be built from this data. 

### ObservationRetriever
This interface contains a single method called “getObservationCollection” which is passed the following 
parameters: SosStation, SosSensor, SosPhenomenon, and a start date. With these parameters an 
ObservationCollection will be returned containing observations newer than the start date. An example of an 
ObservationRetriever is located at com.axiomalaska.sos.cnfaic.CnfaicObservationRetriever.java. This class 
pulls the updated real-time observations from the www.cnfaic.org website. 

### SosStation
The SosStation interface represents a location which has one or more SosSensors. The following must be 
provided to this interface: Location, ID, SosSource, feature of interest name, if the station is moving, and 
a list of Sensors. The Location object stores a latitude and longitude in double format. A requirement for 
the ID is that when it is combined with the Source ID it needs to be unique. An example of an ID is 11111. 
The feature of interest description describes where the station is located; for example, this could be the 
name of the bay the station is in.

### SosSensor
The SosSensor contains a collection of SosPhenomenon and an ID. In the “getSensors” method in the 
com.axiomalaska.sos.cnfaic.CnfaicStationRetriever class there are three example SosSensors: Air Temperature, 
Relative Humidity, and Wind. An SosSensor can have more than one phenomena. For example, the Wind sensor has 
three phenomena: Wind Speed, Wind Direction, and Wind Gust. An SosSensor can also have a single phenomenon as 
seen in the Air Temperature SosSensor.

### SosSource
The SosSource contains information about the owner of a collection of SosStations. View the 
com.axiomalaska.sos.cnfaic.CnfaicStationRetriever “getStations” method to see an example of an SosSource. 

### SosPhenomenon
An SosPhenomenon is one specific value type that is read from a sensor. For example, a wind sensor would have 
three phenomena: Wind speed, Wind direction, and Wind Gust. For an air temperature sensor there would only be 
one phenomenon, Air Temperature. An SosPhenomenon contains the following: name (e.g. Air Temperature), tag 
(e.g urn:x-ogc:def:phenomenon:IOOS:0.0.1:air\_temperature ), and units (e.g. C). Across all the stations the 
phenomena must be the same. For example, there cannot exist a phenomenon from one station for air temperature 
with units of “C” and another with the units of “F”. The spelling must be the same as well; two 
stations with air temperature phenomenon cannot have units of “C” for one and “Celsius” for the 
other. 

### ObservationCollection
The ObservationCollection class is used to hold the sensor observations that are collected. This object holds 
three lists: numeric values, dates, and locations. The values, dates, and locations are related by their 
position in the list. For example, the first items in the three lists are associated to each other, the 
second items  in each list are associated to each other, and so on. The ObservationCollection also contains 
the SosPhenomenon, SosSensor, and SosStation which these observations are associated to. The “location” 
list is only used if the associated station is moving; if the station is not moving, the location list is 
empty. 

### ObservationSubmitter
This class is the core worker for inputting stations, sensors, phenomena, and observations into the SOS. 
There are two ways to use this class to add observations to the SOS. The first way is to push observations 
and the second is to pull observations. To push observations, call the method update(ObservationCollection 
observationCollection, PublisherInfo publisherInfo) with a prebuilt ObservationCollection. This will only add 
the observations that have not already been added to the SOS. To pull observations call the method

    update(List<Station> stations, ObservationRetriever observationRetriever, PublisherInfo publisherInfo)

or the method

    update(Station station, ObservationRetriever observationRetriever, PublisherInfo publisherInfo)

These methods take one or more stations and an ObservationRetriever. These methods go through 
all of the combinations of a station’s sensor and a sensor’s phenomena to pull observations from the 
ObservationRetriever object which are then placed in the SOS. The “pull observations” method is used by 
the ObservationUpdater in the provided example.

### PublisherInfo
The PublisherInfo interface contains information about the organization that is hosting the SOS. This 
includes the organization’s name, country of origin, email address, and web address. 
