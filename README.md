# HMRC VAT Flows


## GRANT Flow
Grant Cashplus access to the following:
* Submit VAT to HMRC
* Read VAT details from HMRC
* Save Application Code in database.
* Save AccessToken and RefreshToken in database
* Grant is valid for 18 months


```mermaid

sequenceDiagram
    participant web.ui
    participant web.api
    participant logic.vat
    participant hmrc
    web.ui->>web.api:userHasValidMTDAccessToken
    web.api->>logic.vat: userHasValidMTDAccessToken
    logic.vat-->>web.api: false
    web.api-->>web.ui: false
    web.ui->>hmrc:authorise
    Note Right of hmrc: will have to be  done via popup iframe window
    hmrc-->>web.api:accessGranted
    web.api->>logic.vat : updateAccessGranted
    logic.vat->>logic.vat : saveAuthorizationCode
    loop WaitForHMRCCompletion
      web.ui->>web.api : hasAccessBeenGranted
      web.api->>logic.vat : hasAccessBeenGranted
      logic.vat-->>web.api : true/false
      web.api-->>web.ui : true/false
    end 
    web.ui->>web.api:getMTDToken
    web.api->>logic.vat:getMTDToken
    logic.vat->>hmrc:/token
    Note Right of hmrc: Note we use scope (write:vat and read:vat) of application token
    hmrc-->>logic.vat : token
    logic.vat-->>logic.vat : saveToken
    logic.vat-->>web.api : success
    web.api-->>web.ui : success

```

## Access Token Refresh Flow
* Use refresh token to get new access token.
* After call both the access token and refresh token will be updated.
* Updated values (both access token and refresh token) will be saved in DB


```mermaid
sequenceDiagram
    participant web.api
    participant logic.vat
    participant hmrc

    web.api->>logic.vat : RefreshAccessToken
    logic.vat ->> hmrc : /token (type RefreshToken)
    Note Right of hmrc: Refresh Token --> active for 4 hours
    hmrc-->>logic.vat : token (both refresh and access)
    logic.vat-->>web.api : MTDToken
```

## GET VAT Obligations Flow
```mermaid
sequenceDiagram
    participant web.api
    participant logic.vat
    participant hmrc

    web.api->>logic.vat : GetOutstandingReturns
    logic.vat ->> hmrc : /oblications (with unique number, date range and AccessToken)
    Note Right of hmrc: returns array of oblications
    hmrc-->>logic.vat : Obligationresult
    logic.vat-->>web.api : Obligationresult
```
    
## Submit VAT Return Flow
```mermaid
sequenceDiagram
    participant web.api
    participant logic.vat
    participant hmrc

    web.api->>logic.vat : SubmitReturns
    logic.vat ->> hmrc : /returns (with unique number, returns and AccessToken)
    Note Right of hmrc: Returns mandatory 9 VAT fields
    hmrc-->>logic.vat : SubmissionResult
    logic.vat-->>web.api : SubmissionResult
```

## Get VAT Return Flow
```mermaid
sequenceDiagram
    participant web.api
    participant logic.vat
    participant hmrc

    web.api->>logic.vat : GetReturn
    logic.vat ->> hmrc : /return (with unique number, period key and access token)
    Note Right of hmrc: returns vat submission result
    hmrc-->>logic.vat : VATReturnResult
    logic.vat-->>web.api : VATReturnResult
```
