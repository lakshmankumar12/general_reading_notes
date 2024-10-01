# band info

3550 to 3700

# abbreviations

SAS - spectrum access system

# cbsd state machine

* Registered/UnRegistered
* Grant
    * Idle/Granted/Authorized
    * Not is spec, but in realiity
        * suspended
        * terminated( same as idle )

# messages

* Registration request
    * Identify CBSD to sas
        * userId                   ??
        * fccId                    ok
        * cbsdSerialNumber         ok
        * callSign                 ??
    * Equipment capability
        * cbsdCategory                ok
        * cbsdInfo               ??
        * airInterface           ??
        * installationParam           ??
    * Mesurement capabilty
        * measCapability           ??
    * Group info
        * groupingParam           ??
* Registration Response
    * Members
        * cbsdId
            * should be used for all subsequent procedures
        * measReportConfig
        * response
            * Indicates success/failure
            * failures:
                * REG_PENDING
                * BLACKLISTED
                * INVALID_VALUE
    * Notes
        * sas will delete all Grants on registration. So, should a cbsd
          before starting one.
* Spectrum Inquiry Procedure
    * AvailableChannel objects, each of which includes
        * a frequency range,
        * the channelType (“PAL” or “GAA”) and
        * the regulatory rule that the SAS used to deteremine availability.
    * optional. Not madatory for sending Grant request
* CBSD Grant Procedure
    * Request Fields
        * cbsdId, operationParam, measReport
        * operationParam gives the max EIRP, and frequency range
    * Response Fields
        * cbsdId, grantId, grantExpireTime,
        * heartbeatInterval, measReportConfig, operationParam, channelType, response
* CBSD Heartbeat procedure
    * Request fields
        * cbsdId, grantId, grantRenew, operationState, measReport
    * Response fields
        * cbsdId, grantId, transmitExpireTime, grantExpireTime,
        * heartbeatInterval, operationParam, measReportConfig, response).

# TR069

xml-rpc list from Enodebd to Enodeb

* GET
* SET
* Reboot
* Upload logs
* Download Firmware
* Add-object/Delete-object

State Transition

* Waiting for Inform in connected state
* Waiting for Inform in disconnected state
* Waiting for empty message
* Waiting for transient cfg-params
* Waiting for regular cfg-params
* Waiting for set responses

Inform params
* Boot
* Reboot
* Periodic
* Value change
* Upload logs complete
