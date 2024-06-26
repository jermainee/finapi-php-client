# finAPI php client

<strong>RESTful API for Account Information Services (AIS) and Payment Initiation Services (PIS)</strong>
<br/>
<strong>Application Version:</strong> 2.36.2
<br/>

The following pages give you some general information on how to use our APIs.<br/>
The actual API services documentation then follows further below. You can use the menu to jump between API sections.
<br/>
<br/>
This page has a built-in HTTP(S) client, so you can test the services directly from within this page,
by filling in the request parameters and/or body in the respective services, and then hitting the TRY button.
Note that you need to be authorized to make a successful API call. To authorize, refer to the 'Authorization'
section of the API, or just use the OAUTH button that can be found near the TRY button.
<br/>

<h2 id=\"general-information\">General information</h2>

<h3 id=\"general-error-responses\"><strong>Error Responses</strong></h3>
When an API call returns with an error, then in general it has the structure shown in the following example:

<pre>
{
  \"errors\": [
    {
      \"message\": \"Interface 'FINTS_SERVER' is not supported for this operation.\",
      \"code\": \"BAD_REQUEST\",
      \"type\": \"TECHNICAL\"
    }
  ],
  \"date\": \"2020-11-19T16:54:06.854+01:00\",
  \"requestId\": \"selfgen-312042e7-df55-47e4-bffd-956a68ef37b5\",
  \"endpoint\": \"POST /api/v2/bankConnections/import\",
  \"authContext\": \"1/21\",
  \"bank\": \"DEMO0002 - finAPI Test Redirect Bank (id: 280002, location: none)\"
}
</pre>

If an API call requires an additional authentication by the user, HTTP code 510 is returned and the error response
contains the additional \"multiStepAuthentication\" object, see the following example:

<pre>
{
  \"errors\": [
    {
      \"message\": \"Es ist eine zusätzliche Authentifizierung erforderlich. Bitte geben Sie folgenden Code an: 123456\",
      \"code\": \"ADDITIONAL_AUTHENTICATION_REQUIRED\",
      \"type\": \"BUSINESS\",
      \"multiStepAuthentication\": {
        \"hash\": \"678b13f4be9ed7d981a840af8131223a\",
        \"status\": \"CHALLENGE_RESPONSE_REQUIRED\",
        \"challengeMessage\": \"Es ist eine zusätzliche Authentifizierung erforderlich. Bitte geben Sie folgenden Code an: 123456\",
        \"answerFieldLabel\": \"TAN\",
        \"redirectUrl\": null,
        \"redirectContext\": null,
        \"redirectContextField\": null,
        \"twoStepProcedures\": null,
        \"photoTanMimeType\": null,
        \"photoTanData\": null,
        \"opticalData\": null,
        \"opticalDataAsReinerSct\": false
      }
    }
  ],
  \"date\": \"2019-11-29T09:51:55.931+01:00\",
  \"requestId\": \"selfgen-45059c99-1b14-4df7-9bd3-9d5f126df294\",
  \"endpoint\": \"POST /api/v2/bankConnections/import\",
  \"authContext\": \"1/18\",
  \"bank\": \"DEMO0001 - finAPI Test Bank\"
}
</pre>

An exception to this error format are API authentication errors, where the following structure is returned:

<pre>
{
  \"error\": \"invalid_token\",
  \"error_description\": \"Invalid access token: cccbce46-xxxx-xxxx-xxxx-xxxxxxxxxx\"
}
</pre>

<h3 id=\"general-paging\"><strong>Paging</strong></h3>
API services that may potentially return a lot of data implement paging.
They return a limited number of entries within a \"page\". Further entries must be fetched with
subsequent calls.
<br/><br/>
Any API service that implements paging provides the following input parameters:<br/>
&bull; \"page\": the number of the page to be retrieved (starting with 1).<br/>
&bull; \"perPage\": the number of entries within a page. The default and maximum value is stated
in the documentation of the respective services.

A paged response contains an additional \"paging\" object with the following structure:

<pre>
{
  ...
  ,
  \"paging\": {
    \"page\": 1,
    \"perPage\": 20,
    \"pageCount\": 234,
    \"totalCount\": 4662
  }
}
</pre>

<h3 id=\"general-internationalization\"><strong>Internationalization</strong></h3>
The finAPI services support internationalization which means you can define the language you prefer
for API service responses.
<br/><br/>
The following languages are available: German, English, Czech, Slovak.
<br/><br/>
The preferred language can be defined by providing the official HTTP <strong>Accept-Language</strong>
header.
<br/><br/>
finAPI reacts on the official iso language codes &quot;de&quot;, &quot;en&quot;, &quot;cs&quot;
and &quot;sk&quot; for the named languages.
Additional subtags supported by the Accept-Language header may be provided,
e.g. &quot;en-US&quot;, but are ignored.
<br/>
If no Accept-Language header is given, German is used as the default language.
<br/><br/>
Exceptions:<br/>
&bull; Bank login hints and login fields are only available in the language of the bank and not
being translated.<br/>
&bull; Direct messages from the bank systems typically returned as BUSINESS errors will not
be translated.<br/>
&bull; BUSINESS errors created by finAPI directly are available in German and English.<br/>
&bull; TECHNICAL errors messages meant for developers are mostly in English, but also may be
translated.

<h3 id=\"general-request-ids\"><strong>Request IDs</strong></h3>
With any API call, you can pass a request ID via a header with name \"X-Request-Id\".
The request ID can be an arbitrary string with up to 255 characters.
Passing a longer string will result in an error.
<br/><br/>
If you don't pass a request ID for a call, finAPI will generate a random ID internally.
<br/><br/>
The request ID is always returned back in the response of a service, as a header with
name \"X-Request-Id\".
<br/><br/>
We highly recommend to always pass a (preferably unique) request ID, and include it into your
client application logs whenever you make a request or receive a response
(especially in the case of an error response). finAPI is also logging request IDs on its end.
Having a request ID can help the finAPI support team to work more efficiently and solve tickets faster.

<h3 id=\"general-overriding-http-methods\"><strong>Overriding HTTP methods</strong></h3>
Some HTTP clients do not support the HTTP methods PATCH or DELETE.
If you are using such a client in your application, you can use a POST request instead with a special
HTTP header indicating the originally intended HTTP method.
<br/><br/>
The header's name is <strong>X-HTTP-Method-Override</strong>. Set its value to either
<strong>PATCH</strong> or <strong>DELETE</strong>.
POST Requests having this header set will be treated either as PATCH or DELETE by the finAPI servers.
<br/><br/>
Example:
<br/><br/>
<strong>X-HTTP-Method-Override: PATCH</strong><br/>
POST /api/v2/label/51<br/>
{\"name\": \"changed label\"}<br/><br/>
will be interpreted by finAPI as:<br/><br/>
PATCH /api/v2/label/51<br/>
{\"name\": \"changed label\"}<br/>

<h3 id=\"general-user-metadata\"><strong>User metadata</strong></h3>
With the migration to PSD2 APIs, a new term called \"User metadata\"
(also known as \"PSU metadata\") has been introduced to the API.
This user metadata aims to inform the banking API if there was a real end-user
behind an HTTP request or if the request was triggered by a system
(e.g. by an automatic batch update). In the latter case, the bank may apply some restrictions
such as limiting the number of HTTP requests for a single consent.
Also, some operations may be forbidden entirely by the banking API.
For example, some banks do not allow issuing a new consent without the end-user being involved.
Therefore, it is certainly necessary and obligatory for the customer to provide the PSU metadata for such operations.
<br/><br/>
As finAPI does not have direct interaction with the end-user,
it is the client application's responsibility to provide all the necessary
information about the end-user. This must be done by sending additional headers with every request
triggered on behalf of the end-user.
<br/><br/>
At the moment, the following headers are supported by the API:<br/>
&bull; \"PSU-IP-Address\" - the IP address of the user's device. It has to be an IPv4 address, as some banks cannot work
with IPv6 addresses. If a non-IPv4 address is passed, we will replace the value with our own IPv4 address as a
fallback.<br/>
&bull; \"PSU-Device-OS\" - the user's device and/or operating system identification.<br/>
&bull; \"PSU-User-Agent\" - the user's web browser or other client device identification.

<h3 id=\"general-faq\"><strong>FAQ</strong></h3>
<strong>Is there a finAPI SDK?</strong>
<br/>
Currently we do not offer a native SDK, but there is the option to generate an SDK
for almost any target language via OpenAPI. Use the 'Download SDK' button on this page for SDK
generation.
<br/>
<br/>
<strong>How can I enable finAPI's automatic batch update?</strong>
<br/>
Currently there is no way to set up the batch update via the API.
Please contact support@finapi.io for this.
<br/>
<br/>
<strong>Why do I need to keep authorizing when calling services on this page?</strong>
<br/>
This page is a \"one-page-app\". Reloading the page resets the OAuth authorization context.
There is generally no need to reload the page, so just don't do it and your authorization will persist.


For more information, please visit [https://www.finapi.io/impressum](https://www.finapi.io/impressum).

## Installation & Usage

### Requirements

PHP 7.4 and later.
Should also work with PHP 8.0.

### Composer

To install the bindings via [Composer](https://getcomposer.org/), add the following to `composer.json`:

```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https:////.git"
    }
  ],
  "require": {
    "/": "*@dev"
  }
}
```

Then run `composer install`

### Manual Installation

Download the files and include `autoload.php`:

```php
<?php
require_once('/path/to/OpenAPIClient-php/vendor/autoload.php');
```

## Getting Started

Please follow the [installation procedure](#installation--usage) and then run the following:

```php
<?php
require_once(__DIR__ . '/vendor/autoload.php');



// Configure OAuth2 access token for authorization: finapi_auth
$config = OpenAPI\Client\Configuration::getDefaultConfiguration()->setAccessToken('YOUR_ACCESS_TOKEN');

// Configure OAuth2 access token for authorization: finapi_auth
$config = OpenAPI\Client\Configuration::getDefaultConfiguration()->setAccessToken('YOUR_ACCESS_TOKEN');


$apiInstance = new OpenAPI\Client\Api\AccountsApi(
    // If you want use custom http client, pass your client which implements `GuzzleHttp\ClientInterface`.
    // This is optional, `GuzzleHttp\Client` will be used as default.
    new GuzzleHttp\Client(),
    $config
);
$id = 56; // int | Identifier of the account to delete
$psu_ip_address = 'psu_ip_address_example'; // string | The IP address of the user's device. This header will be forwarded to the bank on XS2A requests. It has to be an IPv4 address, as some banks cannot work with IPv6 addresses. If a non-IPv4 address is passed, we will replace the value with our own IPv4 address as a fallback.
$psu_device_os = 'psu_device_os_example'; // string | The user's device and/or operating system identification. This header will be forwarded to the bank on XS2A requests.
$psu_user_agent = 'psu_user_agent_example'; // string | The user's web browser or other client device identification. This header will be forwarded to the bank on XS2A requests.
$x_http_method_override = 'x_http_method_override_example'; // string | Some HTTP clients do not support the HTTP methods PATCH or DELETE. If you are using such a client in your application, you can use a POST request instead with this header indicating the originally intended HTTP method. POST Requests having this  header set will be treated either as PATCH or DELETE by the finAPI servers.
$x_request_id = 'x_request_id_example'; // string | With any API call, you can pass a request ID. The request ID can be an arbitrary string with up to 255 characters. Passing a longer string will result in an error. If you don't pass a request ID for a call, finAPI will generate a random ID internally. The request ID is always returned back in the response of a service, as a header with name 'X-Request-Id'. We highly recommend to always pass a (preferably unique) request ID, and include it into your client application logs whenever you make a request or receive a response (especially in the case of an error response). finAPI is also logging request IDs on its end. Having a request ID can help the finAPI support team to work more efficiently and solve tickets faster.

try {
    $apiInstance->deleteAccount($id, $psu_ip_address, $psu_device_os, $psu_user_agent, $x_http_method_override, $x_request_id);
} catch (Exception $e) {
    echo 'Exception when calling AccountsApi->deleteAccount: ', $e->getMessage(), PHP_EOL;
}

```

## API Endpoints

All URIs are relative to *https://sandbox.finapi.io*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*AccountsApi* | [**deleteAccount**](docs/Api/AccountsApi.md#deleteaccount) | **DELETE** /api/v2/accounts/{id} | Delete an account
*AccountsApi* | [**deleteAllAccounts**](docs/Api/AccountsApi.md#deleteallaccounts) | **DELETE** /api/v2/accounts | Delete all accounts
*AccountsApi* | [**editAccount**](docs/Api/AccountsApi.md#editaccount) | **PATCH** /api/v2/accounts/{id} | Edit an account
*AccountsApi* | [**getAccount**](docs/Api/AccountsApi.md#getaccount) | **GET** /api/v2/accounts/{id} | Get an account
*AccountsApi* | [**getAndSearchAllAccounts**](docs/Api/AccountsApi.md#getandsearchallaccounts) | **GET** /api/v2/accounts | Get and search all accounts
*AccountsApi* | [**getDailyBalances**](docs/Api/AccountsApi.md#getdailybalances) | **GET** /api/v2/accounts/dailyBalances | Get daily balances
*AuthorizationApi* | [**getToken**](docs/Api/AuthorizationApi.md#gettoken) | **POST** /api/v2/oauth/token | Get tokens
*AuthorizationApi* | [**revokeToken**](docs/Api/AuthorizationApi.md#revoketoken) | **POST** /api/v2/oauth/revoke | Revoke a token
*BankConnectionsApi* | [**connectInterface**](docs/Api/BankConnectionsApi.md#connectinterface) | **POST** /api/v2/bankConnections/connectInterface | Connect a new interface
*BankConnectionsApi* | [**deleteAllBankConnections**](docs/Api/BankConnectionsApi.md#deleteallbankconnections) | **DELETE** /api/v2/bankConnections | Delete all bank connections
*BankConnectionsApi* | [**deleteBankConnection**](docs/Api/BankConnectionsApi.md#deletebankconnection) | **DELETE** /api/v2/bankConnections/{id} | Delete a bank connection
*BankConnectionsApi* | [**deleteConsent**](docs/Api/BankConnectionsApi.md#deleteconsent) | **DELETE** /api/v2/bankConnections/{id}/aisConsent | Delete a consent
*BankConnectionsApi* | [**editBankConnection**](docs/Api/BankConnectionsApi.md#editbankconnection) | **PATCH** /api/v2/bankConnections/{id} | Edit a bank connection
*BankConnectionsApi* | [**getAllBankConnections**](docs/Api/BankConnectionsApi.md#getallbankconnections) | **GET** /api/v2/bankConnections | Get all bank connections
*BankConnectionsApi* | [**getBankConnection**](docs/Api/BankConnectionsApi.md#getbankconnection) | **GET** /api/v2/bankConnections/{id} | Get a bank connection
*BankConnectionsApi* | [**importBankConnection**](docs/Api/BankConnectionsApi.md#importbankconnection) | **POST** /api/v2/bankConnections/import | Import a new bank connection
*BankConnectionsApi* | [**removeInterface**](docs/Api/BankConnectionsApi.md#removeinterface) | **POST** /api/v2/bankConnections/removeInterface | Remove an interface
*BankConnectionsApi* | [**updateBankConnection**](docs/Api/BankConnectionsApi.md#updatebankconnection) | **POST** /api/v2/bankConnections/update | Update a bank connection
*BanksApi* | [**getAndSearchAllBanks**](docs/Api/BanksApi.md#getandsearchallbanks) | **GET** /api/v2/banks | Get and search all banks
*BanksApi* | [**getBank**](docs/Api/BanksApi.md#getbank) | **GET** /api/v2/banks/{id} | Get a bank
*CategoriesApi* | [**createCategory**](docs/Api/CategoriesApi.md#createcategory) | **POST** /api/v2/categories | Create a new category
*CategoriesApi* | [**deleteAllCategories**](docs/Api/CategoriesApi.md#deleteallcategories) | **DELETE** /api/v2/categories | Delete all categories
*CategoriesApi* | [**deleteCategory**](docs/Api/CategoriesApi.md#deletecategory) | **DELETE** /api/v2/categories/{id} | Delete a category
*CategoriesApi* | [**editCategory**](docs/Api/CategoriesApi.md#editcategory) | **PATCH** /api/v2/categories/{id} | Edit a category
*CategoriesApi* | [**getAndSearchAllCategories**](docs/Api/CategoriesApi.md#getandsearchallcategories) | **GET** /api/v2/categories | Get and search all categories
*CategoriesApi* | [**getCashFlows**](docs/Api/CategoriesApi.md#getcashflows) | **GET** /api/v2/categories/cashFlows | Get cash flows
*CategoriesApi* | [**getCategory**](docs/Api/CategoriesApi.md#getcategory) | **GET** /api/v2/categories/{id} | Get a category
*CategoriesApi* | [**trainCategorization**](docs/Api/CategoriesApi.md#traincategorization) | **POST** /api/v2/categories/trainCategorization | Train categorization
*ClientConfigurationApi* | [**editClientConfiguration**](docs/Api/ClientConfigurationApi.md#editclientconfiguration) | **PATCH** /api/v2/clientConfiguration | Edit client configuration
*ClientConfigurationApi* | [**getClientConfiguration**](docs/Api/ClientConfigurationApi.md#getclientconfiguration) | **GET** /api/v2/clientConfiguration | Get client configuration
*LabelsApi* | [**createLabel**](docs/Api/LabelsApi.md#createlabel) | **POST** /api/v2/labels | Create a new label
*LabelsApi* | [**deleteAllLabels**](docs/Api/LabelsApi.md#deletealllabels) | **DELETE** /api/v2/labels | Delete all labels
*LabelsApi* | [**deleteLabel**](docs/Api/LabelsApi.md#deletelabel) | **DELETE** /api/v2/labels/{id} | Delete a label
*LabelsApi* | [**editLabel**](docs/Api/LabelsApi.md#editlabel) | **PATCH** /api/v2/labels/{id} | Edit a label
*LabelsApi* | [**getAndSearchAllLabels**](docs/Api/LabelsApi.md#getandsearchalllabels) | **GET** /api/v2/labels | Get and search all labels
*LabelsApi* | [**getLabel**](docs/Api/LabelsApi.md#getlabel) | **GET** /api/v2/labels/{id} | Get a label
*MandatorAdministrationApi* | [**changeClientCredentials**](docs/Api/MandatorAdministrationApi.md#changeclientcredentials) | **POST** /api/v2/mandatorAdmin/changeClientCredentials | Change client credentials
*MandatorAdministrationApi* | [**createIbanRules**](docs/Api/MandatorAdministrationApi.md#createibanrules) | **POST** /api/v2/mandatorAdmin/ibanRules | Create IBAN rules
*MandatorAdministrationApi* | [**createKeywordRules**](docs/Api/MandatorAdministrationApi.md#createkeywordrules) | **POST** /api/v2/mandatorAdmin/keywordRules | Create keyword rules
*MandatorAdministrationApi* | [**deleteIbanRules**](docs/Api/MandatorAdministrationApi.md#deleteibanrules) | **POST** /api/v2/mandatorAdmin/ibanRules/delete | Delete IBAN rules
*MandatorAdministrationApi* | [**deleteKeywordRules**](docs/Api/MandatorAdministrationApi.md#deletekeywordrules) | **POST** /api/v2/mandatorAdmin/keywordRules/delete | Delete keyword rules
*MandatorAdministrationApi* | [**deleteUsers**](docs/Api/MandatorAdministrationApi.md#deleteusers) | **POST** /api/v2/mandatorAdmin/deleteUsers | Delete users
*MandatorAdministrationApi* | [**getIbanRuleList**](docs/Api/MandatorAdministrationApi.md#getibanrulelist) | **GET** /api/v2/mandatorAdmin/ibanRules | Get IBAN rules
*MandatorAdministrationApi* | [**getKeywordRuleList**](docs/Api/MandatorAdministrationApi.md#getkeywordrulelist) | **GET** /api/v2/mandatorAdmin/keywordRules | Get keyword rules
*MandatorAdministrationApi* | [**getUserList**](docs/Api/MandatorAdministrationApi.md#getuserlist) | **GET** /api/v2/mandatorAdmin/getUserList | Get user list
*MandatorAdministrationApi* | [**switchApiVersion**](docs/Api/MandatorAdministrationApi.md#switchapiversion) | **POST** /api/v2/mandatorAdmin/switchApiVersion | Switch API Version
*MocksAndTestsApi* | [**checkCategorization**](docs/Api/MocksAndTestsApi.md#checkcategorization) | **POST** /api/v2/tests/checkCategorization | Check categorization
*MocksAndTestsApi* | [**mockBatchUpdate**](docs/Api/MocksAndTestsApi.md#mockbatchupdate) | **POST** /api/v2/tests/mockBatchUpdate | Mock batch update
*NotificationRulesApi* | [**createNotificationRule**](docs/Api/NotificationRulesApi.md#createnotificationrule) | **POST** /api/v2/notificationRules | Create a new notification rule
*NotificationRulesApi* | [**deleteAllNotificationRules**](docs/Api/NotificationRulesApi.md#deleteallnotificationrules) | **DELETE** /api/v2/notificationRules | Delete all notification rules
*NotificationRulesApi* | [**deleteNotificationRule**](docs/Api/NotificationRulesApi.md#deletenotificationrule) | **DELETE** /api/v2/notificationRules/{id} | Delete a notification rule
*NotificationRulesApi* | [**getAndSearchAllNotificationRules**](docs/Api/NotificationRulesApi.md#getandsearchallnotificationrules) | **GET** /api/v2/notificationRules | Get and search all notification rules
*NotificationRulesApi* | [**getNotificationRule**](docs/Api/NotificationRulesApi.md#getnotificationrule) | **GET** /api/v2/notificationRules/{id} | Get a notification rule
*PaymentsApi* | [**createDirectDebit**](docs/Api/PaymentsApi.md#createdirectdebit) | **POST** /api/v2/payments/directDebits | Create direct debit
*PaymentsApi* | [**createMoneyTransfer**](docs/Api/PaymentsApi.md#createmoneytransfer) | **POST** /api/v2/payments/moneyTransfers | Create money transfer
*PaymentsApi* | [**getPayments**](docs/Api/PaymentsApi.md#getpayments) | **GET** /api/v2/payments | Get payments
*PaymentsApi* | [**submitPayment**](docs/Api/PaymentsApi.md#submitpayment) | **POST** /api/v2/payments/submit | Submit payment
*PendingTransactionsApi* | [**getAndSearchAllPendingTransactions**](docs/Api/PendingTransactionsApi.md#getandsearchallpendingtransactions) | **GET** /api/v2/pendingTransactions | Get and search all pending transactions
*PendingTransactionsApi* | [**getPendingTransaction**](docs/Api/PendingTransactionsApi.md#getpendingtransaction) | **GET** /api/v2/pendingTransactions/{id} | Get a pending transaction
*SecuritiesApi* | [**getAndSearchAllSecurities**](docs/Api/SecuritiesApi.md#getandsearchallsecurities) | **GET** /api/v2/securities | Get and search all securities
*SecuritiesApi* | [**getSecurity**](docs/Api/SecuritiesApi.md#getsecurity) | **GET** /api/v2/securities/{id} | Get a security
*StandingOrdersApi* | [**createStandingOrder**](docs/Api/StandingOrdersApi.md#createstandingorder) | **POST** /api/v2/standingOrders | Create a standing order
*StandingOrdersApi* | [**getStandingOrders**](docs/Api/StandingOrdersApi.md#getstandingorders) | **GET** /api/v2/standingOrders | Get standing orders
*StandingOrdersApi* | [**submitStandingOrder**](docs/Api/StandingOrdersApi.md#submitstandingorder) | **POST** /api/v2/standingOrders/submit | Submit standing order
*TPPCertificatesApi* | [**createNewCertificate**](docs/Api/TPPCertificatesApi.md#createnewcertificate) | **POST** /api/v2/tppCertificates | Upload TPP certificate
*TPPCertificatesApi* | [**deleteCertificate**](docs/Api/TPPCertificatesApi.md#deletecertificate) | **DELETE** /api/v2/tppCertificates/{id} | Delete a TPP certificate
*TPPCertificatesApi* | [**getAllCertificates**](docs/Api/TPPCertificatesApi.md#getallcertificates) | **GET** /api/v2/tppCertificates | Get all TPP certificates
*TPPCertificatesApi* | [**getCertificate**](docs/Api/TPPCertificatesApi.md#getcertificate) | **GET** /api/v2/tppCertificates/{id} | Get a TPP certificate
*TPPCredentialsApi* | [**createTppCredential**](docs/Api/TPPCredentialsApi.md#createtppcredential) | **POST** /api/v2/tppCredentials | Upload TPP credentials
*TPPCredentialsApi* | [**deleteTppCredential**](docs/Api/TPPCredentialsApi.md#deletetppcredential) | **DELETE** /api/v2/tppCredentials/{id} | Delete a set of TPP credentials
*TPPCredentialsApi* | [**editTppCredential**](docs/Api/TPPCredentialsApi.md#edittppcredential) | **PATCH** /api/v2/tppCredentials/{id} | Edit a set of TPP credentials
*TPPCredentialsApi* | [**getAllTppCredentials**](docs/Api/TPPCredentialsApi.md#getalltppcredentials) | **GET** /api/v2/tppCredentials | Get all TPP credentials
*TPPCredentialsApi* | [**getAndSearchTppAuthenticationGroups**](docs/Api/TPPCredentialsApi.md#getandsearchtppauthenticationgroups) | **GET** /api/v2/tppCredentials/tppAuthenticationGroups | Get all TPP Authentication Groups
*TPPCredentialsApi* | [**getTppCredential**](docs/Api/TPPCredentialsApi.md#gettppcredential) | **GET** /api/v2/tppCredentials/{id} | Get a set of TPP credentials
*TransactionsApi* | [**deleteAllTransactions**](docs/Api/TransactionsApi.md#deletealltransactions) | **DELETE** /api/v2/transactions | Delete all transactions
*TransactionsApi* | [**deleteTransaction**](docs/Api/TransactionsApi.md#deletetransaction) | **DELETE** /api/v2/transactions/{id} | Delete a transaction
*TransactionsApi* | [**editMultipleTransactions**](docs/Api/TransactionsApi.md#editmultipletransactions) | **PATCH** /api/v2/transactions | Edit multiple transactions
*TransactionsApi* | [**editTransaction**](docs/Api/TransactionsApi.md#edittransaction) | **PATCH** /api/v2/transactions/{id} | Edit a transaction
*TransactionsApi* | [**getAndSearchAllTransactions**](docs/Api/TransactionsApi.md#getandsearchalltransactions) | **GET** /api/v2/transactions | Get and search all transactions
*TransactionsApi* | [**getTransaction**](docs/Api/TransactionsApi.md#gettransaction) | **GET** /api/v2/transactions/{id} | Get a transaction
*TransactionsApi* | [**restoreTransaction**](docs/Api/TransactionsApi.md#restoretransaction) | **POST** /api/v2/transactions/{id}/restore | Restore a transaction
*TransactionsApi* | [**splitTransaction**](docs/Api/TransactionsApi.md#splittransaction) | **POST** /api/v2/transactions/{id}/split | Split a transaction
*TransactionsApi* | [**triggerCategorization**](docs/Api/TransactionsApi.md#triggercategorization) | **POST** /api/v2/transactions/triggerCategorization | Trigger categorization
*UsersApi* | [**createUser**](docs/Api/UsersApi.md#createuser) | **POST** /api/v2/users | Create a new user
*UsersApi* | [**deleteAuthorizedUser**](docs/Api/UsersApi.md#deleteauthorizeduser) | **DELETE** /api/v2/users | Delete the authorized user
*UsersApi* | [**deleteUnverifiedUser**](docs/Api/UsersApi.md#deleteunverifieduser) | **DELETE** /api/v2/users/{userId} | Delete an unverified user
*UsersApi* | [**editAuthorizedUser**](docs/Api/UsersApi.md#editauthorizeduser) | **PATCH** /api/v2/users | Edit the authorized user
*UsersApi* | [**executePasswordChange**](docs/Api/UsersApi.md#executepasswordchange) | **POST** /api/v2/users/executePasswordChange | Execute password change
*UsersApi* | [**getAuthorizedUser**](docs/Api/UsersApi.md#getauthorizeduser) | **GET** /api/v2/users | Get the authorized user
*UsersApi* | [**getVerificationStatus**](docs/Api/UsersApi.md#getverificationstatus) | **GET** /api/v2/users/verificationStatus | Get a user&#39;s verification status
*UsersApi* | [**requestPasswordChange**](docs/Api/UsersApi.md#requestpasswordchange) | **POST** /api/v2/users/requestPasswordChange | Request password change
*UsersApi* | [**verifyUser**](docs/Api/UsersApi.md#verifyuser) | **POST** /api/v2/users/verify/{userId} | Verify a user

## Models

- [AccessToken](docs/Model/AccessToken.md)
- [Account](docs/Model/Account.md)
- [AccountCapability](docs/Model/AccountCapability.md)
- [AccountInterface](docs/Model/AccountInterface.md)
- [AccountInterfacePaymentCapabilities](docs/Model/AccountInterfacePaymentCapabilities.md)
- [AccountList](docs/Model/AccountList.md)
- [AccountParams](docs/Model/AccountParams.md)
- [AccountReference](docs/Model/AccountReference.md)
- [AccountStatus](docs/Model/AccountStatus.md)
- [AccountType](docs/Model/AccountType.md)
- [BadCredentialsError](docs/Model/BadCredentialsError.md)
- [Bank](docs/Model/Bank.md)
- [BankBankGroup](docs/Model/BankBankGroup.md)
- [BankConnection](docs/Model/BankConnection.md)
- [BankConnectionBank](docs/Model/BankConnectionBank.md)
- [BankConnectionInterface](docs/Model/BankConnectionInterface.md)
- [BankConnectionInterfaceAisConsent](docs/Model/BankConnectionInterfaceAisConsent.md)
- [BankConnectionInterfaceLastAutoUpdate](docs/Model/BankConnectionInterfaceLastAutoUpdate.md)
- [BankConnectionInterfaceLastManualUpdate](docs/Model/BankConnectionInterfaceLastManualUpdate.md)
- [BankConnectionList](docs/Model/BankConnectionList.md)
- [BankConnectionOwner](docs/Model/BankConnectionOwner.md)
- [BankConsent](docs/Model/BankConsent.md)
- [BankConsentStatus](docs/Model/BankConsentStatus.md)
- [BankGroup](docs/Model/BankGroup.md)
- [BankIcon](docs/Model/BankIcon.md)
- [BankImage](docs/Model/BankImage.md)
- [BankInterface](docs/Model/BankInterface.md)
- [BankInterfaceLoginField](docs/Model/BankInterfaceLoginField.md)
- [BankInterfacePaymentCapabilities](docs/Model/BankInterfacePaymentCapabilities.md)
- [BankInterfacePaymentConstraints](docs/Model/BankInterfacePaymentConstraints.md)
- [BankInterfaceProperty](docs/Model/BankInterfaceProperty.md)
- [BankInterfaceTppAuthenticationGroup](docs/Model/BankInterfaceTppAuthenticationGroup.md)
- [BankLogo](docs/Model/BankLogo.md)
- [BankingInterface](docs/Model/BankingInterface.md)
- [CashFlow](docs/Model/CashFlow.md)
- [CashFlowCategory](docs/Model/CashFlowCategory.md)
- [CashFlowList](docs/Model/CashFlowList.md)
- [CategorizationCheckResult](docs/Model/CategorizationCheckResult.md)
- [CategorizationCheckResultCategory](docs/Model/CategorizationCheckResultCategory.md)
- [CategorizationCheckResults](docs/Model/CategorizationCheckResults.md)
- [CategorizationRuleDirection](docs/Model/CategorizationRuleDirection.md)
- [CategorizationStatus](docs/Model/CategorizationStatus.md)
- [Category](docs/Model/Category.md)
- [CategoryParams](docs/Model/CategoryParams.md)
- [CertisTransactionData](docs/Model/CertisTransactionData.md)
- [ChangeClientCredentialsParams](docs/Model/ChangeClientCredentialsParams.md)
- [CheckCategorizationData](docs/Model/CheckCategorizationData.md)
- [CheckCategorizationTransactionData](docs/Model/CheckCategorizationTransactionData.md)
- [ClientConfiguration](docs/Model/ClientConfiguration.md)
- [ClientConfigurationParams](docs/Model/ClientConfigurationParams.md)
- [ConnectInterfaceParams](docs/Model/ConnectInterfaceParams.md)
- [ConnectInterfaceParamsMultiStepAuthentication](docs/Model/ConnectInterfaceParamsMultiStepAuthentication.md)
- [CounterpartAddressData](docs/Model/CounterpartAddressData.md)
- [CreateDirectDebitParams](docs/Model/CreateDirectDebitParams.md)
- [CreateMoneyTransferParams](docs/Model/CreateMoneyTransferParams.md)
- [CreateStandingOrderParams](docs/Model/CreateStandingOrderParams.md)
- [Currency](docs/Model/Currency.md)
- [DailyBalance](docs/Model/DailyBalance.md)
- [DailyBalanceList](docs/Model/DailyBalanceList.md)
- [DailyBalanceListPaging](docs/Model/DailyBalanceListPaging.md)
- [DeleteConsent](docs/Model/DeleteConsent.md)
- [DeleteConsentResult](docs/Model/DeleteConsentResult.md)
- [DirectDebitOrderParams](docs/Model/DirectDebitOrderParams.md)
- [DirectDebitSequenceType](docs/Model/DirectDebitSequenceType.md)
- [DirectDebitType](docs/Model/DirectDebitType.md)
- [DomesticMoneyTransferConstraints](docs/Model/DomesticMoneyTransferConstraints.md)
- [DomesticMoneyTransferMandatoryFields](docs/Model/DomesticMoneyTransferMandatoryFields.md)
- [EditBankConnectionParams](docs/Model/EditBankConnectionParams.md)
- [EditCategoryParams](docs/Model/EditCategoryParams.md)
- [EditTppCredentialParams](docs/Model/EditTppCredentialParams.md)
- [EnabledProducts](docs/Model/EnabledProducts.md)
- [ErrorCode](docs/Model/ErrorCode.md)
- [ErrorDetails](docs/Model/ErrorDetails.md)
- [ErrorDetailsMultiStepAuthentication](docs/Model/ErrorDetailsMultiStepAuthentication.md)
- [ErrorMessage](docs/Model/ErrorMessage.md)
- [ErrorType](docs/Model/ErrorType.md)
- [ExecutePasswordChangeParams](docs/Model/ExecutePasswordChangeParams.md)
- [ISO3166Alpha2Codes](docs/Model/ISO3166Alpha2Codes.md)
- [IbanRule](docs/Model/IbanRule.md)
- [IbanRuleCategory](docs/Model/IbanRuleCategory.md)
- [IbanRuleIdentifiersParams](docs/Model/IbanRuleIdentifiersParams.md)
- [IbanRuleList](docs/Model/IbanRuleList.md)
- [IbanRuleParams](docs/Model/IbanRuleParams.md)
- [IbanRulesParams](docs/Model/IbanRulesParams.md)
- [IdentifierList](docs/Model/IdentifierList.md)
- [ImportBankConnectionParams](docs/Model/ImportBankConnectionParams.md)
- [KeywordRule](docs/Model/KeywordRule.md)
- [KeywordRuleIdentifiersParams](docs/Model/KeywordRuleIdentifiersParams.md)
- [KeywordRuleList](docs/Model/KeywordRuleList.md)
- [KeywordRuleParams](docs/Model/KeywordRuleParams.md)
- [KeywordRulesParams](docs/Model/KeywordRulesParams.md)
- [Label](docs/Model/Label.md)
- [LabelParams](docs/Model/LabelParams.md)
- [Language](docs/Model/Language.md)
- [LoginCredential](docs/Model/LoginCredential.md)
- [LoginCredentialResource](docs/Model/LoginCredentialResource.md)
- [MandatorLicense](docs/Model/MandatorLicense.md)
- [MockAccountData](docs/Model/MockAccountData.md)
- [MockBankConnectionUpdate](docs/Model/MockBankConnectionUpdate.md)
- [MockBatchUpdateParams](docs/Model/MockBatchUpdateParams.md)
- [MoneyTransferOrderParams](docs/Model/MoneyTransferOrderParams.md)
- [MoneyTransferOrderParamsCounterpartAddress](docs/Model/MoneyTransferOrderParamsCounterpartAddress.md)
- [MonthlyUserStats](docs/Model/MonthlyUserStats.md)
- [MsaStatus](docs/Model/MsaStatus.md)
- [MultiStepAuthenticationCallback](docs/Model/MultiStepAuthenticationCallback.md)
- [MultiStepAuthenticationChallenge](docs/Model/MultiStepAuthenticationChallenge.md)
- [NewTransaction](docs/Model/NewTransaction.md)
- [NotificationRule](docs/Model/NotificationRule.md)
- [NotificationRuleList](docs/Model/NotificationRuleList.md)
- [NotificationRuleParams](docs/Model/NotificationRuleParams.md)
- [OrderInitiationStatus](docs/Model/OrderInitiationStatus.md)
- [PageableBankList](docs/Model/PageableBankList.md)
- [PageableCategoryList](docs/Model/PageableCategoryList.md)
- [PageableIbanRuleList](docs/Model/PageableIbanRuleList.md)
- [PageableKeywordRuleList](docs/Model/PageableKeywordRuleList.md)
- [PageableLabelList](docs/Model/PageableLabelList.md)
- [PageablePaymentResources](docs/Model/PageablePaymentResources.md)
- [PageablePendingTransactionResources](docs/Model/PageablePendingTransactionResources.md)
- [PageableSecurityList](docs/Model/PageableSecurityList.md)
- [PageableStandingOrderResources](docs/Model/PageableStandingOrderResources.md)
- [PageableTppAuthenticationGroupResources](docs/Model/PageableTppAuthenticationGroupResources.md)
- [PageableTppCertificateList](docs/Model/PageableTppCertificateList.md)
- [PageableTppCredentialResources](docs/Model/PageableTppCredentialResources.md)
- [PageableTransactionList](docs/Model/PageableTransactionList.md)
- [PageableUserInfoList](docs/Model/PageableUserInfoList.md)
- [Paging](docs/Model/Paging.md)
- [PasswordChangingResource](docs/Model/PasswordChangingResource.md)
- [Payment](docs/Model/Payment.md)
- [PaymentType](docs/Model/PaymentType.md)
- [PaypalTransactionData](docs/Model/PaypalTransactionData.md)
- [PendingTransaction](docs/Model/PendingTransaction.md)
- [PendingTransactionCertisData](docs/Model/PendingTransactionCertisData.md)
- [PendingTransactionPaypalData](docs/Model/PendingTransactionPaypalData.md)
- [PreferredConsentType](docs/Model/PreferredConsentType.md)
- [Product](docs/Model/Product.md)
- [RemoveInterfaceParams](docs/Model/RemoveInterfaceParams.md)
- [RequestPasswordChangeParams](docs/Model/RequestPasswordChangeParams.md)
- [ResponseMessage](docs/Model/ResponseMessage.md)
- [Security](docs/Model/Security.md)
- [SecurityPositionQuantityNominalType](docs/Model/SecurityPositionQuantityNominalType.md)
- [SecurityPositionQuoteType](docs/Model/SecurityPositionQuoteType.md)
- [SepaMoneyTransferConstraints](docs/Model/SepaMoneyTransferConstraints.md)
- [SepaMoneyTransferCounterpartAddressMandatoryFields](docs/Model/SepaMoneyTransferCounterpartAddressMandatoryFields.md)
- [SepaMoneyTransferMandatoryFields](docs/Model/SepaMoneyTransferMandatoryFields.md)
- [SplitTransactionsParams](docs/Model/SplitTransactionsParams.md)
- [StandingOrder](docs/Model/StandingOrder.md)
- [StandingOrderFrequency](docs/Model/StandingOrderFrequency.md)
- [SubTransactionParams](docs/Model/SubTransactionParams.md)
- [SubmitPaymentParams](docs/Model/SubmitPaymentParams.md)
- [SubmitStandingOrderParams](docs/Model/SubmitStandingOrderParams.md)
- [SwitchApiVersionParams](docs/Model/SwitchApiVersionParams.md)
- [TppAuthenticationGroup](docs/Model/TppAuthenticationGroup.md)
- [TppCertificate](docs/Model/TppCertificate.md)
- [TppCertificateParams](docs/Model/TppCertificateParams.md)
- [TppCertificateType](docs/Model/TppCertificateType.md)
- [TppCredentials](docs/Model/TppCredentials.md)
- [TppCredentialsParams](docs/Model/TppCredentialsParams.md)
- [TrainCategorizationData](docs/Model/TrainCategorizationData.md)
- [TrainCategorizationTransactionData](docs/Model/TrainCategorizationTransactionData.md)
- [Transaction](docs/Model/Transaction.md)
- [TransactionCategory](docs/Model/TransactionCategory.md)
- [TransactionDirection](docs/Model/TransactionDirection.md)
- [TriggerCategorizationParams](docs/Model/TriggerCategorizationParams.md)
- [TwoStepProcedure](docs/Model/TwoStepProcedure.md)
- [UpdateBankConnectionParams](docs/Model/UpdateBankConnectionParams.md)
- [UpdateMultipleTransactionsParams](docs/Model/UpdateMultipleTransactionsParams.md)
- [UpdateResult](docs/Model/UpdateResult.md)
- [UpdateResultStatus](docs/Model/UpdateResultStatus.md)
- [UpdateTransactionsParams](docs/Model/UpdateTransactionsParams.md)
- [User](docs/Model/User.md)
- [UserCreateParams](docs/Model/UserCreateParams.md)
- [UserIdentifiersList](docs/Model/UserIdentifiersList.md)
- [UserIdentifiersParams](docs/Model/UserIdentifiersParams.md)
- [UserInfo](docs/Model/UserInfo.md)
- [UserUpdateParams](docs/Model/UserUpdateParams.md)
- [VerificationStatusResource](docs/Model/VerificationStatusResource.md)

## Authorization

### finapi_auth

- **Type**: `OAuth`
- **Flow**: `password`
- **Authorization URL**: ``
- **Scopes**: 
    - **all**: no limitations


### finapi_auth

- **Type**: `OAuth`
- **Flow**: `application`
- **Authorization URL**: ``
- **Scopes**: 
    - **all**: no limitations

## Tests

To run the tests, use:

```bash
composer install
vendor/bin/phpunit
```

## Author

kontakt@finapi.io

## About this package

This PHP package is automatically generated by the [OpenAPI Generator](https://openapi-generator.tech) project:

- API version: `2024.14.3`
- Build package: `org.openapitools.codegen.languages.PhpClientCodegen`
