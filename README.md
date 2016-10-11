# Network Traffic App for Splunk

## Overview
Very often, network traffic events can provide a lot of information about misconfigurations, potential attacks, and user activity. This app provides searches and dashboards based on the [Splunk Common Information Model](https://docs.splunk.com/Documentation/CIM/latest/User/Overview) to help provide insight into your network traffic.

## A note on Splunk Data Model Acceleration and Disk Space

This app requires data model acceleration, which will use additional disk space. If you are using the Splunk App for Enterprise Security, this is already enabled, and should have been factored into your retention policies. If not, you should review the documentation on data model acceleration, how it uses disk space, and how to plan for it. This documentation can be found in the [Splunk Enterprise Knowledge Manager Manual](http://docs.splunk.com/Documentation/Splunk/latest/Knowledge/Acceleratedatamodels#Data_model_summary_size_on_disk).

## A note on the Splunk Common Information Model

As mentioned above, the app uses the CIM for network traffic events. The CIM allows you to take events from a number of sources or products, and report on them in one cohesive manner, using a common set of names for fields and event types.

## A note on the Network Traffic Data Model, src, dest, src_ip, and dest_ip

The `Network Traffic` data model includes both `src`, `dest` fields, and `src_ip`, `dest_ip` fields. For this app, I have opted to use the `_ip` versions of these fields, in case hostnames are being used for the other fields. Make sure your field extractions are correctly populating these fields.

## Available Dashboards
Below are the dashboards available within the app.

### Overview
Provides a general overview of the network traffic events.

### IP Profile
Provides information around an IP address (both `dest_ip` and `src_ip`), including traffic from, to, and possible open ports.

### Transport Information
Information around the `transport` field of events (TCP, UDP, ICMP, etc.).

### Port Information
This form provides information based on the destination port (`dest_port`) field of events, such as the traffic over time, conversations, and sources.

### Internal and External Traffic
Currently just top destinations (external and from external to internal). The determination on internal vs. external is configured by macros. See the [App Configuration Macros](#app-configuration-macros) section of this document.

### Scanning Activity
This dashboard provides the top potential scanners (both host and port scanners) based on network traffic.

### Geographic Information
This is based on the geo-ip information provided by the built-in IP location from Splunk. Internal traffic is excluded from this page. __Note__: The searches on this page may take a while to load.

### Network Traffic Search
A form which allows for searching network traffic events based on a few different parameters.

### Sourcetype Information

__Note__: This dashboard is not shown in the navigation bar. To view this dashboard, go to `Settings` -> `User Interface` -> `Views` -> and select the `Open` option next to the `sourcetype_information` item in the list.

## Prerequisites

### Splunk Versions
This app has been tested with Splunk versions 6.4.x. This app should be installed on the same search head on which the `network_traffic` data model has been accelerated.

### Splunk Common Information Model Add-on
This app depends on data models included in the Splunk Common Information Model Add-on, specifically the `NetworkTraffic` data model. Information on installing and using the Splunk Common Information Model Add-on can be found in the [Common Information Model Add-on Manual](http://docs.splunk.com/Documentation/CIM/latest/User/Install), as well as [tag and field requirements for the Network Traffic data model](http://docs.splunk.com/Documentation/CIM/4.5.0/User/NetworkTraffic).
Information on configuring the acceleration on the data model can be found in the [Knowledge Manager Manual](http://docs.splunk.com/Documentation/Splunk/latest/Knowledge/Acceleratedatamodels#Enable_persistent_acceleration_for_a_data_model).

The Splunk Common Information Model Add-on can be downloaded from [Splunkbase](https://apps.splunk.com/app/1621/). This app has been tested with versions 4.5 of the CIM add-on.

### Data model Acceleration on the Network Traffic data model
In order to make the app respond and load quickly, accelerated data models are used to provide summary data. For this data to be available, the `Network_Traffic` data model must be accelerated. Information on how to enable acceleration for the `Network_Traffic` data model can be found in the [Knowledge Manager Manual](http://docs.splunk.com/Documentation/Splunk/latest/Knowledge/Managedatamodels#Enable_data_model_acceleration).

## Installation
This app should be installed on a search head where the `Network_Traffic` data model has been accelerated. More information on installing or upgrading Splunk apps can be found here: <http://docs.splunk.com/Documentation/Splunk/latest/Admin/Wheretogetmoreapps>

__Note__: This app will require a restart of Splunk.

## Simple Installation Process
1. Make sure the field extractions and tags on your network traffic events are correct.
2. Install the Splunk Common Information Model Add-on (skip if you are installing on an ES search head).
4. Enable accelerations on the `Network_Traffic` data model (skip if you are installing on an ES search head).
3. Install the Network Traffic App for Splunk.
4. Restart Splunk.
5. Continue with App Configuration.

## App Configuration

This app may require some configuration before it will work properly (outside of the configuration of the Data Model Acceleration). In particular, you may need to edit the configuration macros, as well as the dropdown which populates the `Device` dropdown found on many of the dashboards.


### App Configuration Macros

#### network_traffic_dvcs
This macro contains the search which is used to populate the `Devices` dropdown found on many of the dashboards. By default this is the auto-generated lookup, however, the macro can be edited to point to another lookup as needed.

#### network_traffic_dest_external
This macro contains a partial search which determines when a network traffic event's destination IP address is external to your network. By default this will exclude private IP addresses, but can be edited to reflect your own network configuration. __Note__: this search snippet is used in searches using the accelerated data models and the [tstats](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/Tstats) command. While the normal SPL does support CIDR, `tstats` does not. Make sure your search syntax will work with the `tstats` command.

#### network_traffic_src_external
This macro contains a partial search which determines when a network traffic event's source IP address is external to your network. By default this will exclude private IP addresses, but can be edited to reflect your own network configuration. __Note__: this search snippet is used in searches using the accelerated data models and the [tstats](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/Tstats) command. While the normal SPL does support CIDR, `tstats` does not. Make sure your search syntax will work with the `tstats` command.

### Device Dropdown Lookup

For the "Device" dropdown, present on many of the dashboards, you can use the auto-generated lookup, which runs every morning at 2 am.
If your `Network_Traffic` data model is populated, you can run the saved search `network_traffic_dvc_auto_gen` to populate the dropdown.

The lookup has two fields: `dvc` and `device_name`. `device_name` can be a description of the device. `dvc` can be wild-carded (not CIDR, as that is not available in `tstats` searches used with the accelerated data models). The search used to populate this dropdown can be configured using the `network_traffic_dvcs` macro.

## Future Plans

* Performance improvements (using base searches for dashboards)
* Options for the exclusion of internal traffic from dashboard views
* More, based on your feedback

## About support for this app
Support for this app is provided on a best-effort basis. We have released this app for free, and want to help solve issues, and add features, but we also have day-jobs.

Need help? Use the Splunk community resources! I can be found on many of them:

* [Splunk Answers](https://answers.splunk.com/)
* [#splunk on Efnet IRC](https://wiki.splunk.com/Community:IRC)
* [Splunk Slack channel](http://splunk402.com/chat/)

The git repo for this app is located [here](https://github.com/automine/network_traffic).

## Credits

- This app was created by David Shpritz of [Aplura, LLC](https://www.aplura.com)
- Thanks to the members of #splunk on IRC, and the Splunk Community Slack channel
- Thanks to other members of the Splunk Professional Services group for input on panels
- Thanks to Yorokobi for name suggestions
- Icons made by [Madebyoliver](http://www.flaticon.com/authors/madebyoliver) from [www.flaticon.com](http://www.flaticon.com) is licensed by [CC 3.0 BY](http://creativecommons.org/licenses/by/3.0/)

## Release Notes

### 1.1
* Removed default csv file for device dropdown
* Edited README.md to fix section on the device dropdown

### 1.0

First release.
