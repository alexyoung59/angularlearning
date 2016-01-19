# Pitney Bowes GeoCodeUsAddress Module

*Initial Configuration in PTS 4.0 for the Homeowner Product*

*Purpose:* The purpose of adding the Pitney Bowes GeoCodeUsAddress is to provide a level of address verification/validation along with returning
and storing the latitude and longitude coordinates for the property insured.

Configured to perform as follows:

When a user is initially quoting a homeowner policy and they enter the dwelling address fields, the system will watch for the user to exit those fields (streetNum, streetName, city, etc...)
and when the user exits the fields the system will check first to verify all required "key" fields have been completed. Those key fields are streetName, city, state and zip. Once those
fields have been completed and the user exits any address field the code will run a quick check to see if the address has changed from the initial value of the fields at load of the page.
For instance if the system loaded with 123 Main St Perry, GA 31069 and now the address fields read 123 Main St NW Perry, GA 31069 the system will recognize a change in address and attempt
to validate the new address. The call to Pitney Bowes will try and return a match or potential matches based on their own formulas. If there is no match the system will return a match code
that begins with an E followed by a three digit number such as E027. If there is a match the system will return it along with a confidence level. It will return longitude and latitude also.
There are many more rules and configurable options for this module but these are the main goals of the initial installation and you can read the users manual to get more detailed information
as needed.

Once the address is returned it pops up in a box that will only close once the user makes a selection. If the user enters an address that returns as a 100% match, then
the system will just pop up one radio button indicating "address verified" then the user clicks that and moves on. The system will store addressVerified as 1 indicating it was verified, latitude,
longitude and confidence level. If the user enters an address that returns one or more matches and the confidence level is not 100% the system will display the choices as "keep address as entered" 
and one or more potential addresses returned by Pitney Bowes. If the user enters an address that returns no matches and has an error code, the system will store addressVerified as a 3
and store the error code. This indicates an the Verification System returned an error.

SUSPENSES GENERATED:
The system will generate suspenses for two scenarios. In the event the user chooses to skip address validation even though a valid address was avaliable, the system will store this as a skip
and when the application is submitted (meaning the policy is in Pending status), the system will store a 15 day UW suspense for review but will allow agents to continue on to bind the policy.
This suspense will also generate anytime an address is changed on a Pending, Bound, Active new or renewal policy and meets these criteria (skipped address validation). 

The second suspense scenario is if the user enters an invalid address and an error code is returned. the system will store this as a an invalid address
and when the application is submitted (meaning the policy is in Pending status), the system will store a 15 day UW suspense for review but will allow agents to continue on to bind the policy.
This suspense will also generate anytime an address is changed on a Pending, Bound, Active new or renewal policy and meets these criteria (invalid address validation). 


## DB Changes:
### New Tables:
	[MainDB].[GeoCodeMatchCode] - Stores match codes provided by GeoCode that describe the result of a search. Currently populated table with error codes only and they are used to provide UW's
		additional information on suspenses when there was an invalid address entered.
	
	[MainDB].[GeoCodeRequest] - Stores each outgoing request and updates with the response from GeoCode so that we have a record of each call for purposes of debugging in the case the results
		are not as expected.
		
Modified:
*[MainDB].[HomeownerDwelling]* - addressVerified tinyint [0 - not checked, 1 - verified, 2 - user skipped, 3 - invalid], latitude decimal(19,6),
	longitude decimal(19,6), geoCodeConfidenceLevel tinyint, matchCode varchar(4)
	
*[LogDB].[HomeownerDwelling]* - addressVerified tinyint [0 - not checked, 1 - verified, 2 - user skipped, 3 - invalid], latitude decimal(19,6),
	longitude decimal(19,6), geoCodeConfidenceLevel tinyint, matchCode varchar(4)
	
*[MainDB]..[CreateHomeownerRenewal]* - Copy over HomeownerDwelling fields associated with this module.

*[LogDB]..[Insert_PolicyLogStateHO]* - Log the changes to HomeownerDwelling table fields associated with this module for each policy state.

## Files:
exportProcess/PitneyBowes/GeoCode/geoCode.cfm </br>
exportProcess/PitneyBowes/GeoCode/geoCode.inc </br>
exportProcess/PitneyBowes/GeoCode/geoCodeUsAddressObj.cfc </br>
exportProcess/PitneyBowes/GeoCode/geoCodeUsAddressScript.inc </br>
exportProcess/PitneyBowes/GeoCode/stdlib.cfc </br>
policy/Homeowner/SouthernTrust/GA/Panel_3/dwellingLocation.inc </br>
policy/Homeowner/SouthernTrust/TN/Panel_3/dwellingLocation.inc</br>
policy/Homeowner/policy_action.inc </br>
quote/Homeowner/SouthernTrust/GA/Panel_2/dwellingLocation.inc </br>
quote/Homeowner/SouthernTrust/GA/quote_action.inc </br>
quote/Homeowner/SouthernTrust/GA/scripts.inc </br>
quote/Homeowner/SouthernTrust/TN/Panel_2/dwellingLocation.inc </br>
quote/Homeowner/SouthernTrust/TN/quote_action.inc </br>
policy/Homeowner/policy_URSuspenseItems.inc </br>
policy/applyPendingEndorsement.cfm </br>
quote/Homeowner/SouthernTrust/TN/scripts.inc  </br>
images/fancybox/ </br>
