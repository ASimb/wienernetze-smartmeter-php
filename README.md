# wienernetze-smartmeter-php
Read energy-consumption from Wiener Netze Smartmeters.

## Object constructor:
__new ViennaSmartmeter(username, password, [debug=true/false])__

## Available functions:
- __login()__  
  does the login with Wiener Netze webpage credentials
  returns "true" if successful, otherwise "false"
- __getProfile()__
  get your profile info  
  returns:
  ```
  stdClass Object
      (
      [id] => 00000
      [salutation] => Herr
      [lastname] => Mustermann
      [firstname] => Max
      [email] => max.mustermann@mustermail.com
      [defaultGeschaeftspartnerRegistration] => stdClass Object
          (
          [id] => 000000
          [registrationKey] => 000000000000
          [zaehlpunkt] => AT0000000000000000000000000000000   // meterpoint
          [status] => CONFIRMED
          [geschaeftspartner] => 0000000000                   // customerid
          [completedAt] => 2000-12-24T18:00:00.000Z
          )
      [isZpsDonee] => 
      [registration] => stdClass Object
          (
          [zaehlpunkt] => AT0000000000000000000000000000000
          )
      )
  ```
 - __welcome()__  
   get all Infos on the welcome-page  
   returns:
   ```
   stdClass Object
       (
       [meterReadings] => Array
           (
           [0] => stdClass Object
               (
               [value] => 0               // current value of smartmeter
               [type] => 1-2:1.8.0
               [validated] => 1
               [date] => 2024-12-24T18:00:00.000Z
               )
           )
       )
    ```
- __getConsumptionData(meterpoint, customerid, start_time, end_time, role, aggregate)__  
  get energy-consumption for the period between start_time and end_time  
  _start_time/end_time_ must have format yyyy-mm-dd hh:mm:ss (e.g. 2024-12-24 18:30:00)  
  _aggregate_ has the possible values NONE (period is 15-minutes), SUM_PER_DAY (period is 1 day)   
  _role_ has the possible values V001 (period as selected for aggregate), V002 (period is 15-minutes anyway)   
  returns an object, which consists of two parts:
     
  _part 1: "descriptor"_
  ```
  stdClass Object
      (
      [geschaeftspartnernummer] => 0000000000                   // customerid
      [zaehlpunktnummer] => AT0000000000000000000000000000000   // meterpoint
      [rolle] => V001
      [aggregat] => NONE
      [granularitaet] => D
      [einheit] => KWH
      )
    ```
    _part 2: "values"_  
    including an array of objects. Each object represents the measurements of the required period:
    ```
    stdClass Object
        (
        [wert] => 0.043                           // value of consumption in kWh for the period
        [zeitpunktVon] => 2024-12-27T08:00:00Z
        [zeitpunktBis] => 2024-12-27T08:15:00Z
        [geschaetzt] => 
        )
    ``` 
- __getConsumptionByDay(meterpoint, customerid, startingday, [startingtime])__  
  get energy-consumption by startingday  
  _startingday_ must have the format yyyy-mm-dd  
  _startingtime_ must have the format hh:mm (parameter is optional)  
  **IMPORTANT:** this function is obsolete and is implemented for compatibility reasons only. Please use getConsumptionData()  
  returns an array of objects. Each object represents the measurements of the required period:
  ```
  stdClass Object
      (
      [value] => 304                           // value of consumption in Wh (!) for the period
      [timestamp] => 2024-12-27T08:00:00Z
      )
  ```
- __getMeasurements(profile, start_date, end_date, type)__  
  get energy-consumption for the period between start_day and end_day  
  _start_date/end_date_ must have the format yyyy-mm-dd  
  _type_ has the possible values QUARTER_HOUR, DAY, METER_READ  
  returns:
  ```
  stdClass Object
       (
       [zaehlwerke] => Array
           (
           [0] => stdClass Object
                (
                [obisCode] => 1-1:1.9.0
                [einheit] => WH
                [messwerte] => Array
                    (
                    [0] => stdClass Object
                         (
                         [messwert] => 113
                         [zeitVon] => 2024-12-21T23:00:00.000Z
                         [zeitBis] => 2024-12-21T23:15:00.000Z
                         [qualitaet] => VAL
                         )
                    [1] => stdClass Object
                         (
                         [messwert] => 144
                         [zeitVon] => 2024-12-21T23:15:00.000Z
                         [zeitBis] => 2024-12-21T23:30:00.000Z
                         [qualitaet] => VAL
                         )
                    [3] => ........
                    )
                )
            )
        [zaehlpunkt] => AT0000000000000000000000000000000   // meterpoint
    )
  ```
- __getEvents(meterpoint, start, end)__  
  get events limited by start and end parameters  
  _start/end_ must have the format yyyy-mm-dd hh:mm:ss  
  returns an array of events
- __createEvent(meterpoint, name, start, end)__  
  create an event
- __deleteEvent(id)__
  deletes an event by id. The id is returned with getEvents().
- __getLimits()__
  get limits set by the user   
  returns an array of all created limits. Each element of this list has an entry like this:  
  ```
  [resourceLocation] => https://service.wienernetze.at/rest/smp/1.0/m/radar/benachrichtigung/12345678
  ```
  The number at the end of this url is the ID
- __createLimit(name, end, period, threshold, type, meterpoint, customerid)__
  create new Limit  
  _end_ must have the format yyyy-mm-dd hh:mm:ss  
  _period_ can take d or m for day or month  
  _threshold_ in Watt per Hour, not kWh  
  _type_ can take lt ( less than ) and gt ( greater than)
- __deleteLimit(id)__  
  delete limit. The id is returned with getLimits() or createLimit()
- __getNotifications(limit, order)__  
  gets notifications limited by _limit_ and ordered by _order_.
- __getMeterPoints()__  
  gets all Meterpoints assinged to your account ( full detail ).  
  returns an array containing an object for each meterpoint with all available data


## Usage
```php
<?php
	require_once("smartmeter-vienna.class.php");
	$sm = new ViennaSmartmeter("[yourusername]", "[yourpassword]", $debug=false);
	if ($sm->login()) {
	    $me = $sm->getProfile();
	    print_r($me);
	    $yesterday = date('Y-m-d H:i:00', strtotime("-1 days"));
	    $daybefore = date('Y-m-d H:i:00', strtotime("-2 days"));
	    $consumptiondata = $sm->getConsumptionData($me->defaultGeschaeftspartnerRegistration->zaehlpunkt, $me->defaultGeschaeftspartnerRegistration->geschaeftspartner, $daybefore, $yesterday);
	    print_r($consumptiondata);
        } else {
	    echo "WN login error.";
        }
?>

```
## Requirements
- php-curl

## Disclaimer
This is not an official API of Wiener Netze.
