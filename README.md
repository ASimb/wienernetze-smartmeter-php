# wienernetze-smartmeter-php
Read energy-consumption from Wiener Netze Smartmeters.

## Object constructor:
new ViennaSmartmeter(username, password, [debug=true/false])

## Available functions:
- login()  
  does the login with Wiener Netze webpage credentials
  returns "true" if successful, otherwise "false"
- getProfile()
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
 - welcome()  
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
        // getConsumptionData(meterpoint, customerid, start_time, end_time, role, aggregate)
        //                        ... get energy-consumption for the period between start_time and end_time
        //                            start_time/end_time must have format yyyy-mm-dd hh:mm:ss (e.g. 2024-12-24 18:30:00)
        //                            aggregate has the possible values NONE (period is 15-minutes), SUM_PER_DAY (period is 1 day) 
        //                            role has the possible values V001 (period as selected for aggregate), V002 (period is 15-minutes anyway) 
        //                            returns an object, which consists of two parts:
        //                            part 1: "descriptor"
        //                            stdClass Object
        //                                    (
        //                                        [geschaeftspartnernummer] => 0000000000                   // customerid
        //                                        [zaehlpunktnummer] => AT0000000000000000000000000000000   // meterpoint
        //                                        [rolle] => V001
        //                                        [aggregat] => NONE
        //                                        [granularitaet] => D
        //                                        [einheit] => KWH
        //                                    )
        //                            part 2: "values" including an array of objects. Each object represents the measurements of the required period:
        //                            stdClass Object
        //                                    (
        //                                        [wert] => 0.043                           // value of consumption in kWh for the period
        //                                        [zeitpunktVon] => 2024-12-27T08:00:00Z
        //                                        [zeitpunktBis] => 2024-12-27T08:15:00Z
        //                                        [geschaetzt] => 
        //                                    )
        // getConsumptionByDay(meterpoint, customerid, startingday, [startingtime])
        //                        ... get energy-consumption by startingday
        //                            startingday must have the format yyyy-mm-dd
        //                            startingtime must have the format hh:mm (parameter is optional)
        //                            IMPORTANT: this function is obsolete and is implemented for compatibility reasons only.
        //                                       please use getConsumptionData()
        //                            returns an array of objects. Each object represents the measurements of the required period:
        //                            stdClass Object
        //                                    (
        //                                        [value] => 304                           // value of consumption in Wh (!) for the period
        //                                        [timestamp] => 2024-12-27T08:00:00Z
        //                                    )
        // getMeasurements(profile, start_date, end_date, type)
        //                        ... get energy-consumption for the period between start_day and end_day
        //                            start_date/end_date must have the format yyyy-mm-dd
        //                            type has the possible values QUARTER_HOUR, DAY, METER_READ
        //                            returns:
        //                            stdClass Object
        //                            (
        //                                [zaehlwerke] => Array
        //                                    (
        //                                        [0] => stdClass Object
        //                                            (
        //                                                [obisCode] => 1-1:1.9.0
        //                                                [einheit] => WH
        //                                                [messwerte] => Array
        //                                                    (
        //                                                        [0] => stdClass Object
        //                                                            (
        //                                                                [messwert] => 113
        //                                                                [zeitVon] => 2024-12-21T23:00:00.000Z
        //                                                                [zeitBis] => 2024-12-21T23:15:00.000Z
        //                                                                [qualitaet] => VAL
        //                                                            )
        //                            
        //                                                        [1] => stdClass Object
        //                                                            (
        //                                                                [messwert] => 144
        //                                                                [zeitVon] => 2024-12-21T23:15:00.000Z
        //                                                                [zeitBis] => 2024-12-21T23:30:00.000Z
        //                                                                [qualitaet] => VAL
        //                                                            )
        //                                                        [3] => ........
        //                                                    )
        //                                            )
        //                                    )
        //                                [zaehlpunkt] => AT0000000000000000000000000000000   // meterpoint
        //                            )
        // getEvents(meterpoint, start, end)
        //                        ... get events limited by start and end parameters
        //                            start/end must have the format yyyy-mm-dd hh:mm:ss
        //                            returns an array of events
        // createEvent(meterpoint, name, start, end)
        //                        ... create an event
        // deleteEvent(id)        ... deletes an event by id. The id is returned with getEvents().
        // getLimits()            ... get limits set by the user 
        //                            returns an array of all created limits       
        //                            each element of this list has an entry
        //                            [resourceLocation] => https://service.wienernetze.at/rest/smp/1.0/m/radar/benachrichtigung/12345678
        //                            The number at the end of this url is the ID
        // createLimit(name, end, period, threshold, type, meterpoint, customerid)
        //                        ... create new Limit
        //                            end must have the format yyyy-mm-dd hh:mm:ss
	//                            period can take d or m for day or month.
	//                            threshold in Watt per Hour, not kWh
	//                            type can take lt ( less than ) and gt ( greater than)
        // deleteLimit(id)        ... delete limit. The id is returned with getLimits() or createLimit()
        // getNotifications(limit, order): 
        //                        ... gets notifications limited by $limit and ordered by $order.
        // getMeterPoints()       ... gets all Meterpoints assinged to your account ( full detail ).
        //                            returns an array containing an object for each meterpoint with all available data



## Usage
```php
<?php
	require_once("smartmeter-vienna.class.php");
	$sm = new ViennaSmartmeter("[yourusername]", "[yourpassword]", $debug=false);
	
	if($sm->login()){
		$profile = $sm->getProfile();
		print_r($profile);

		$meterpoint = $profile->defaultGeschaeftspartnerRegistration->zaehlpunkt;
		$customerid = $profile->defaultGeschaeftspartnerRegistration->geschaeftspartner;

		$yesterday = date('Y-m-d',strtotime("-1 days"));

		$consumption = $sm->getConsumptionByDay($meterpoint, $customerid, $yesterday);
		print_r($consumption);
	}else{
		echo "WN login error.";
	}

```
## Requirements
- php-curl

## Disclaimer
This is not an official API of Wiener Netze.
