# OpenAPIClient-php

Phrase is a translation management platform for software projects. You can collaborate on language file translation with your team or order translations through our platform. The API allows you to import locale files, download locale files, tag keys or interact in other ways with the localization data stored in Phrase for your account.

## API Endpoint

<div class=\"resource__code--outer\">
  <div class=\"code-section\">
    <pre><code>https://api.phrase.com/v2/</code></pre>
  </div>
</div>

The API is only accessible via HTTPS, the base URL is <code>https://api.phrase.com/</code>, and the current version is <code>v2</code> which results in the base URL for all requests: <code>https://api.phrase.com/v2/</code>.


## Usage

[curl](http://curl.haxx.se/) is used primarily to send requests to Phrase in the examples. On most you'll find a second variant using the [Phrase API v2 client](https://phrase.com/cli/) that might be more convenient to handle. For further information check its [documentation](https://help.phrase.com/help/phrase-in-your-terminal).


## Use of HTTP Verbs

Phrase API v2 tries to use the appropriate HTTP verb for accessing each endpoint according to REST specification where possible:

<div class=\"table-responsive\">
  <table class=\"basic-table\">
    <thead>
      <tr class=\"basic-table__row basic-table__row--header\">
        <th class=\"basic-table__cell basic-table__cell--header\">Verb</th>
        <th class=\"basic-table__cell basic-table__cell--header\">Description</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class=\"basic-table__cell\">GET</td>
        <td class=\"basic-table__cell\">Retrieve one or multiple resources</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">POST</td>
        <td class=\"basic-table__cell\">Create a resource</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">PUT</td>
        <td class=\"basic-table__cell\">Update a resource</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">PATCH</td>
        <td class=\"basic-table__cell\">Update a resource (partially)</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">DELETE</td>
        <td class=\"basic-table__cell\">Delete a resource</td>
      </tr>
    </tbody>
  </table>
</div>


## Identification via User-Agent

You must include the User-Agent header with the name of your application or project. It might be a good idea to include some sort of contact information  as well, so that we can get in touch if necessary (e.g. to warn you about Rate-Limiting or badly formed requests). Examples of excellent User-Agent headers:

<pre><code>User-Agent: Frederiks Mobile App (frederik@phrase.com)
User-Agent: ACME Inc Python Client (http://example.com/contact)</code></pre>

If you don't send this header, you will receive a response with 400 Bad Request.


## Lists

When you request a list of resources, the API will typically only return an array of resources including their most important attributes. For a detailed representation of the resource you should request its detailed representation.

Lists are usually [paginated](#pagination).


## Parameters

Many endpoints support additional parameters, e.g. for pagination. When passing them in a GET request you can send them as HTTP query string parameters:

<pre><code>$ curl -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects?page=2\"</code></pre>

When performing a POST, PUT, PATCH or DELETE request, we recommend sending parameters that are not already included in the URL, as JSON body:

<pre><code>$ curl -H 'Content-Type: application/json' -d '{\"name\":\"My new project\"}' -u EMAIL_OR_ACCESS_TOKEN https://api.phrase.com/v2/projects</code></pre>

Encoding parameters as JSON means better support for types (boolean, integer) and usually better readability. Don't forget to set the correct Content-Type for your request.

*The Content-Type header is omitted in some of the following examples for better readbility.*


## Errors


### Request Errors

If a request contains invalid JSON or is missing a required parameter (besides resource attributes), the status `400 Bad Request` is returned:

<pre><code>{
  \"message\": \"JSON could not be parsed\"
}</code></pre>


### Validation Errors

When the validation for a resource fails, the status `422 Unprocessable Entity` is returned, along with information on the affected fields:

<pre><code>{
  \"message\": \"Validation Failed\",
  \"errors\": [
    {
      \"resource\": \"Project\",
      \"field\": \"name\",
      \"message\": \"can't be blank\"
    }
  ]
}</code></pre>


## Date Format

Times and dates are returned and expected in [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) date format:

<pre><code>YYYY-MM-DDTHH:MM:SSZ</code></pre>

Instead of 'Z' for UTC time zone you can specify your time zone's locale offset using the following notation:

<pre><code>YYYY-MM-DDTHH:MM:SS¬±hh:mm</code></pre>

Example for CET (1 hour behind UTC):

<pre><code>2015-03-31T13:00+01:00</code></pre>

Please note that in HTTP headers, we will use the appropriate recommended date formats instead of ISO 8601.


## Authentication

<div class=\"alert alert-info\">For more detailed information on authentication, check out the <a href=\"#authentication\">API v2 Authentication Guide</a>.</div>

There are two different ways to authenticate when performing API requests:

* E-Mail and password
* Oauth Access Token


### E-Mail and password

To get started easily, you can use HTTP Basic authentication with your email and password:

<pre><code>$ curl -u username:password \"https://api.phrase.com/v2/projects\"</code></pre>


### OAuth via Access Tokens

You can create and manage access tokens in your [profile settings](https://app.phrase.com/settings/oauth_access_tokens) in Translation Center or via the [Authorizations API](#authorizations).

Simply pass the access token as the username of your request:

<pre><code>$ curl -u ACCESS_TOKEN: \"https://api.phrase.com/v2/projects\"</code></pre>

or send the access token via the `Authorization` header field:

<pre><code>$ curl -H \"Authorization: token ACCESS_TOKEN\" https://api.phrase.com/v2/projects</code></pre>

For more detailed information on authentication, check out the <a href=\"#authentication\">API v2 Authentication Guide</a>.

#### Send via parameter

As JSONP (and other) requests cannot send HTTP Basic Auth credentials, a special query parameter `access_token` can be used:

<pre><code>curl \"https://api.phrase.com/v2/projects?access_token=ACCESS_TOKEN\"</code></pre>

You should only use this transport method if sending the authentication via header or Basic authentication is not possible.

### Two-Factor-Authentication

Users with Two-Factor-Authentication enabled have to send a valid token along their request with certain authentication methods (such as Basic authentication). The necessity of a Two-Factor-Authentication token is indicated by the `X-PhraseApp-OTP: required; :MFA-type` header in the response. The `:MFA-type`field indicates the source of the token, e.g. `app` (refers to your Authenticator application):

<pre><code>X-PhraseApp-OTP: required; app</code></pre>

To provide a Two-Factor-Authentication token you can simply send it in the header of the request:

<pre><code>curl -H \"X-PhraseApp-OTP: MFA-TOKEN\" -u EMAIL https://api.phrase.com/v2/projects</code></pre>

Since Two-Factor-Authentication tokens usually expire quickly, we recommend using an alternative authentication method such as OAuth access tokens.

### Multiple Accounts

Some endpoints require the account ID to be specified if the authenticated user is a member of multiple accounts. You can find the eight-digit account ID inside <a href=\"https://app.phrase.com/\" target=\"_blank\">Translation Center</a> by switching to the desired account and then visiting the account details page. If required, you can specify the account just like a normal parameter within the request.

## Pagination

Endpoints that return a list or resources will usually return paginated results and include 25 items by default. To access further pages, use the `page` parameter:

<pre><code>$ curl -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects?page=2\"</code></pre>

Some endpoints also allow a custom page size by using the `per_page` parameter:

<pre><code>$ curl -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects?page=2&per_page=50\"</code></pre>

Unless specified otherwise in the description of the respective endpoint, `per_page` allows you to specify a page size up to 100 items.


## Link-Headers

We provide you with pagination URLs in the [Link Header field](http://tools.ietf.org/html/rfc5988). Make use of this information to avoid building pagination URLs yourself.

<pre><code>Link: <https://api.phrase.com/v2/projects?page=1>; rel=\"first\", <https://api.phrase.com/v2/projects?page=3>; rel=\"prev\", <https://api.phrase.com/v2/projects?page=5>; rel=\"next\", <https://api.phrase.com/v2/projects?page=9>; rel=\"last\"</code></pre>

Possible `rel` values are:

<div class=\"table-responsive\">
  <table class=\"basic-table\">
    <thead>
      <tr class=\"basic-table__row basic-table__row--header\">
        <th class=\"basic-table__cell basic-table__cell--header\">Value</th>
        <th class=\"basic-table__cell basic-table__cell--header\">Description</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class=\"basic-table__cell\">next</td>
        <td class=\"basic-table__cell\">URL of the next page of results</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">last</td>
        <td class=\"basic-table__cell\">URL of the last page of results</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">first</td>
        <td class=\"basic-table__cell\">URL of the first page of results</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\">prev</td>
        <td class=\"basic-table__cell\">URL of the previous page of results</td>
      </tr>
    </tbody>
  </table>
</div>

## Rate Limiting

All API endpoints are subject to rate limiting to ensure good performance for all customers. The rate limit is calculated per user:

* 1000 requests per 5 minutes
* 4 concurrent (parallel) requests

For your convenience we send information on the current rate limit within the response headers:

<div class=\"table-responsive\">
  <table class=\"basic-table\">
    <thead>
      <tr class=\"basic-table__row basic-table__row--header\">
        <th class=\"basic-table__cell basic-table__cell--header\">Header</th>
        <th class=\"basic-table__cell basic-table__cell--header\">Description</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class=\"basic-table__cell\" style=\"white-space: nowrap;\"><code>X-Rate-Limit-Limit</code></td>
        <td class=\"basic-table__cell\">Number of max requests allowed in the current time period</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\" style=\"white-space: nowrap;\"><code>X-Rate-Limit-Remaining</code></td>
        <td class=\"basic-table__cell\">Number of remaining requests in the current time period</td>
      </tr>
      <tr>
        <td class=\"basic-table__cell\" style=\"white-space: nowrap;\"><code>X-Rate-Limit-Reset</code></td>
        <td class=\"basic-table__cell\">Timestamp of end of current time period as UNIX timestamp</td>
      </tr>
    </tbody>
  </table>
</div>

If you should run into the rate limit, you will receive the HTTP status code `429: Too many requests`.

If you should need higher rate limits, [contact us](https://phrase.com/contact).


## Conditional GET requests / HTTP Caching

<div class=\"alert alert-info\"><p><strong>Note:</strong> Conditional GET requests are currently only supported for <a href=\"#locales_download\">locales#download</a> and <a href=\"#translations_index\">translations#index</a></p></div>

We will return an ETag or Last-Modified header with most GET requests. When you request a resource we recommend to store this value and submit them on subsequent requests as `If-Modified-Since` and `If-None-Match` headers. If the resource has not changed in the meantime, we will return the status `304 Not Modified` instead of rendering and returning the resource again. In most cases this is less time-consuming and makes your application/integration faster.

Please note that all conditional requests that return a response with status 304 don't count against your rate limits.

<pre><code>$ curl -i -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects/1234abcd1234abcdefefabcd1234efab/locales/en/download\"
HTTP/1.1 200 OK
ETag: \"abcd1234abcdefefabcd1234efab1234\"
Last-Modified: Wed, 28 Jan 2015 15:31:30 UTC
Status: 200 OK

$ curl -i -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects/1234abcd1234abcdefefabcd1234efab/locales/en/download\" -H 'If-None-Match: \"abcd1234abcdefefabcd1234efab1234\"'
HTTP/1.1 304 Not Modified
ETag: \"abcd1234abcdefefabcd1234efab1234\"
Last-Modified: Wed, 28 Jan 2015 15:31:30 UTC
Status: 304 Not Modified

$ curl -i -u EMAIL_OR_ACCESS_TOKEN \"https://api.phrase.com/v2/projects/1234abcd1234abcdefefabcd1234efab/locales/en/download\" -H \"If-Modified-Since: Wed, 28 Jan 2015 15:31:30 UTC\"
HTTP/1.1 304 Not Modified
Last-Modified: Wed, 28 Jan 2015 15:31:30 UTC
Status: 304 Not Modified</code></pre>


## JSONP

The Phrase API supports [JSONP](http://en.wikipedia.org/wiki/JSONP) for all GET requests in order to deal with cross-domain request issues. Just send a `?callback` parameter along with the request to specify the Javascript function name to be called with the response content:

<pre><code>$ curl \"https://api.phrase.com/v2/projects?callback=myFunction\"</code></pre>

The response will include the normal output for that endpoint, along with a `meta` section including header data:

<pre><code>myFunction({
  {
    \"meta\": {
      \"status\": 200,
      ...
    },
    \"data\": [
      {
        \"id\": \"1234abcd1234abc1234abcd1234abc\"
        ...
      }
    ]
  }
});</code></pre>

To authenticate a JSONP request, you can send a valid [access token](#authentication) as the `?access_token` parameter along the request:

<pre><code>$ curl \"https://api.phrase.com/v2/projects?callback=myFunction&access_token=ACCESS-TOKEN\"</code></pre>


This PHP package is automatically generated by the [OpenAPI Generator](https://openapi-generator.tech) project:

- API version: 2.0.0
- Package version: 1.0.0
- Build package: org.openapitools.codegen.languages.PhpClientCodegen
For more information, please visit [https://developers.phrase.com/api/](https://developers.phrase.com/api/)

## Requirements

PHP 5.5 and later

## Installation & Usage

### Composer

To install the bindings via [Composer](http://getcomposer.org/), add the following to `composer.json`:

```json
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/GIT_USER_ID/GIT_REPO_ID.git"
    }
  ],
  "require": {
    "GIT_USER_ID/GIT_REPO_ID": "*@dev"
  }
}
```

Then run `composer install`

### Manual Installation

Download the files and include `autoload.php`:

```php
    require_once('/path/to/OpenAPIClient-php/vendor/autoload.php');
```

## Tests

To run the unit tests:

```bash
composer install
./vendor/bin/phpunit
```

## Getting Started

Please follow the [installation procedure](#installation--usage) and then run the following:

```php
<?php
require_once(__DIR__ . '/vendor/autoload.php');



// Configure HTTP basic authorization: Basic
$config = Phrase\Configuration::getDefaultConfiguration()
              ->setUsername('YOUR_USERNAME')
              ->setPassword('YOUR_PASSWORD');

// Configure API key authorization: Token
$config = Phrase\Configuration::getDefaultConfiguration()->setApiKey('Authorization', 'YOUR_API_KEY');
// Uncomment below to setup prefix (e.g. Bearer) for API key, if needed
// $config = Phrase\Configuration::getDefaultConfiguration()->setApiKeyPrefix('Authorization', 'Bearer');


$apiInstance = new Phrase\Api\AccountsApi(
    // If you want use custom http client, pass your client which implements `GuzzleHttp\ClientInterface`.
    // This is optional, `GuzzleHttp\Client` will be used as default.
    new GuzzleHttp\Client(),
    $config
);
$id = 'id_example'; // string | ID
$x_phrase_app_otp = 'x_phrase_app_otp_example'; // string | Two-Factor-Authentication token (optional)

try {
    $result = $apiInstance->accountShow($id, $x_phrase_app_otp);
    print_r($result);
} catch (Exception $e) {
    echo 'Exception when calling AccountsApi->accountShow: ', $e->getMessage(), PHP_EOL;
}

?>
```

## Documentation for API Endpoints

All URIs are relative to *https://api.phrase.com/v2*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*AccountsApi* | [**accountShow**](docs/Api/AccountsApi.md#accountshow) | **GET** /accounts/{id} | Get a single account
*AccountsApi* | [**accountsList**](docs/Api/AccountsApi.md#accountslist) | **GET** /accounts | List accounts
*AuthorizationsApi* | [**authorizationCreate**](docs/Api/AuthorizationsApi.md#authorizationcreate) | **POST** /authorizations | Create an authorization
*AuthorizationsApi* | [**authorizationDelete**](docs/Api/AuthorizationsApi.md#authorizationdelete) | **DELETE** /authorizations/{id} | Delete an authorization
*AuthorizationsApi* | [**authorizationShow**](docs/Api/AuthorizationsApi.md#authorizationshow) | **GET** /authorizations/{id} | Get a single authorization
*AuthorizationsApi* | [**authorizationUpdate**](docs/Api/AuthorizationsApi.md#authorizationupdate) | **PATCH** /authorizations/{id} | Update an authorization
*AuthorizationsApi* | [**authorizationsList**](docs/Api/AuthorizationsApi.md#authorizationslist) | **GET** /authorizations | List authorizations
*BitbucketSyncApi* | [**bitbucketSyncExport**](docs/Api/BitbucketSyncApi.md#bitbucketsyncexport) | **POST** /bitbucket_syncs/{id}/export | Export from Phrase to Bitbucket
*BitbucketSyncApi* | [**bitbucketSyncImport**](docs/Api/BitbucketSyncApi.md#bitbucketsyncimport) | **POST** /bitbucket_syncs/{id}/import | Import to Phrase from Bitbucket
*BitbucketSyncApi* | [**bitbucketSyncsList**](docs/Api/BitbucketSyncApi.md#bitbucketsyncslist) | **GET** /bitbucket_syncs | List Bitbucket syncs
*BlacklistedKeysApi* | [**blacklistedKeyCreate**](docs/Api/BlacklistedKeysApi.md#blacklistedkeycreate) | **POST** /projects/{project_id}/blacklisted_keys | Create a blacklisted key
*BlacklistedKeysApi* | [**blacklistedKeyDelete**](docs/Api/BlacklistedKeysApi.md#blacklistedkeydelete) | **DELETE** /projects/{project_id}/blacklisted_keys/{id} | Delete a blacklisted key
*BlacklistedKeysApi* | [**blacklistedKeyShow**](docs/Api/BlacklistedKeysApi.md#blacklistedkeyshow) | **GET** /projects/{project_id}/blacklisted_keys/{id} | Get a single blacklisted key
*BlacklistedKeysApi* | [**blacklistedKeyUpdate**](docs/Api/BlacklistedKeysApi.md#blacklistedkeyupdate) | **PATCH** /projects/{project_id}/blacklisted_keys/{id} | Update a blacklisted key
*BlacklistedKeysApi* | [**blacklistedKeysList**](docs/Api/BlacklistedKeysApi.md#blacklistedkeyslist) | **GET** /projects/{project_id}/blacklisted_keys | List blacklisted keys
*BranchesApi* | [**branchCompare**](docs/Api/BranchesApi.md#branchcompare) | **GET** /projects/{project_id}/branches/{name}/compare | Compare branches
*BranchesApi* | [**branchCreate**](docs/Api/BranchesApi.md#branchcreate) | **POST** /projects/{project_id}/branches | Create a branch
*BranchesApi* | [**branchDelete**](docs/Api/BranchesApi.md#branchdelete) | **DELETE** /projects/{project_id}/branches/{name} | Delete a branch
*BranchesApi* | [**branchMerge**](docs/Api/BranchesApi.md#branchmerge) | **PATCH** /projects/{project_id}/branches/{name}/merge | Merge a branch
*BranchesApi* | [**branchShow**](docs/Api/BranchesApi.md#branchshow) | **GET** /projects/{project_id}/branches/{name} | Get a single branch
*BranchesApi* | [**branchUpdate**](docs/Api/BranchesApi.md#branchupdate) | **PATCH** /projects/{project_id}/branches/{name} | Update a branch
*BranchesApi* | [**branchesList**](docs/Api/BranchesApi.md#brancheslist) | **GET** /projects/{project_id}/branches | List branches
*CommentsApi* | [**commentCreate**](docs/Api/CommentsApi.md#commentcreate) | **POST** /projects/{project_id}/keys/{key_id}/comments | Create a comment
*CommentsApi* | [**commentDelete**](docs/Api/CommentsApi.md#commentdelete) | **DELETE** /projects/{project_id}/keys/{key_id}/comments/{id} | Delete a comment
*CommentsApi* | [**commentMarkCheck**](docs/Api/CommentsApi.md#commentmarkcheck) | **GET** /projects/{project_id}/keys/{key_id}/comments/{id}/read | Check if comment is read
*CommentsApi* | [**commentMarkRead**](docs/Api/CommentsApi.md#commentmarkread) | **PATCH** /projects/{project_id}/keys/{key_id}/comments/{id}/read | Mark a comment as read
*CommentsApi* | [**commentMarkUnread**](docs/Api/CommentsApi.md#commentmarkunread) | **DELETE** /projects/{project_id}/keys/{key_id}/comments/{id}/read | Mark a comment as unread
*CommentsApi* | [**commentShow**](docs/Api/CommentsApi.md#commentshow) | **GET** /projects/{project_id}/keys/{key_id}/comments/{id} | Get a single comment
*CommentsApi* | [**commentUpdate**](docs/Api/CommentsApi.md#commentupdate) | **PATCH** /projects/{project_id}/keys/{key_id}/comments/{id} | Update a comment
*CommentsApi* | [**commentsList**](docs/Api/CommentsApi.md#commentslist) | **GET** /projects/{project_id}/keys/{key_id}/comments | List comments
*DistributionsApi* | [**distributionCreate**](docs/Api/DistributionsApi.md#distributioncreate) | **POST** /accounts/{account_id}/distributions | Create a distribution
*DistributionsApi* | [**distributionDelete**](docs/Api/DistributionsApi.md#distributiondelete) | **DELETE** /accounts/{account_id}/distributions/{id} | Delete a distribution
*DistributionsApi* | [**distributionShow**](docs/Api/DistributionsApi.md#distributionshow) | **GET** /accounts/{account_id}/distributions/{id} | Get a single distribution
*DistributionsApi* | [**distributionUpdate**](docs/Api/DistributionsApi.md#distributionupdate) | **PATCH** /accounts/{account_id}/distributions/{id} | Update a distribution
*DistributionsApi* | [**distributionsList**](docs/Api/DistributionsApi.md#distributionslist) | **GET** /accounts/{account_id}/distributions | List distributions
*FormatsApi* | [**formatsList**](docs/Api/FormatsApi.md#formatslist) | **GET** /formats | List formats
*GitLabSyncApi* | [**gitlabSyncDelete**](docs/Api/GitLabSyncApi.md#gitlabsyncdelete) | **DELETE** /gitlab_syncs/{id} | Delete single Sync Setting
*GitLabSyncApi* | [**gitlabSyncExport**](docs/Api/GitLabSyncApi.md#gitlabsyncexport) | **POST** /gitlab_syncs/{gitlab_sync_id}/export | Export from Phrase to GitLab
*GitLabSyncApi* | [**gitlabSyncHistory**](docs/Api/GitLabSyncApi.md#gitlabsynchistory) | **GET** /gitlab_syncs/{gitlab_sync_id}/history | History of single Sync Setting
*GitLabSyncApi* | [**gitlabSyncImport**](docs/Api/GitLabSyncApi.md#gitlabsyncimport) | **POST** /gitlab_syncs/{gitlab_sync_id}/import | Import from GitLab to Phrase
*GitLabSyncApi* | [**gitlabSyncList**](docs/Api/GitLabSyncApi.md#gitlabsynclist) | **GET** /gitlab_syncs | List GitLab syncs
*GitLabSyncApi* | [**gitlabSyncShow**](docs/Api/GitLabSyncApi.md#gitlabsyncshow) | **GET** /gitlab_syncs/{id} | Get single Sync Setting
*GitLabSyncApi* | [**gitlabSyncUpdate**](docs/Api/GitLabSyncApi.md#gitlabsyncupdate) | **PUT** /gitlab_syncs/{id} | Update single Sync Setting
*GlossaryApi* | [**glossariesList**](docs/Api/GlossaryApi.md#glossarieslist) | **GET** /accounts/{account_id}/glossaries | List glossaries
*GlossaryApi* | [**glossaryCreate**](docs/Api/GlossaryApi.md#glossarycreate) | **POST** /accounts/{account_id}/glossaries | Create a glossary
*GlossaryApi* | [**glossaryDelete**](docs/Api/GlossaryApi.md#glossarydelete) | **DELETE** /accounts/{account_id}/glossaries/{id} | Delete a glossary
*GlossaryApi* | [**glossaryShow**](docs/Api/GlossaryApi.md#glossaryshow) | **GET** /accounts/{account_id}/glossaries/{id} | Get a single glossary
*GlossaryApi* | [**glossaryUpdate**](docs/Api/GlossaryApi.md#glossaryupdate) | **PATCH** /accounts/{account_id}/glossaries/{id} | Update a glossary
*GlossaryTermTranslationsApi* | [**glossaryTermTranslationCreate**](docs/Api/GlossaryTermTranslationsApi.md#glossarytermtranslationcreate) | **POST** /accounts/{account_id}/glossaries/{glossary_id}/terms/{term_id}/translations | Create a glossary term translation
*GlossaryTermTranslationsApi* | [**glossaryTermTranslationDelete**](docs/Api/GlossaryTermTranslationsApi.md#glossarytermtranslationdelete) | **DELETE** /accounts/{account_id}/glossaries/{glossary_id}/terms/{term_id}/translations/{id} | Delete a glossary term translation
*GlossaryTermTranslationsApi* | [**glossaryTermTranslationUpdate**](docs/Api/GlossaryTermTranslationsApi.md#glossarytermtranslationupdate) | **PATCH** /accounts/{account_id}/glossaries/{glossary_id}/terms/{term_id}/translations/{id} | Update a glossary term translation
*GlossaryTermsApi* | [**glossaryTermCreate**](docs/Api/GlossaryTermsApi.md#glossarytermcreate) | **POST** /accounts/{account_id}/glossaries/{glossary_id}/terms | Create a glossary term
*GlossaryTermsApi* | [**glossaryTermDelete**](docs/Api/GlossaryTermsApi.md#glossarytermdelete) | **DELETE** /accounts/{account_id}/glossaries/{glossary_id}/terms/{id} | Delete a glossary term
*GlossaryTermsApi* | [**glossaryTermShow**](docs/Api/GlossaryTermsApi.md#glossarytermshow) | **GET** /accounts/{account_id}/glossaries/{glossary_id}/terms/{id} | Get a single glossary term
*GlossaryTermsApi* | [**glossaryTermUpdate**](docs/Api/GlossaryTermsApi.md#glossarytermupdate) | **PATCH** /accounts/{account_id}/glossaries/{glossary_id}/terms/{id} | Update a glossary term
*GlossaryTermsApi* | [**glossaryTermsList**](docs/Api/GlossaryTermsApi.md#glossarytermslist) | **GET** /accounts/{account_id}/glossaries/{glossary_id}/terms | List glossary terms
*InvitationsApi* | [**invitationCreate**](docs/Api/InvitationsApi.md#invitationcreate) | **POST** /accounts/{account_id}/invitations | Create a new invitation
*InvitationsApi* | [**invitationDelete**](docs/Api/InvitationsApi.md#invitationdelete) | **DELETE** /accounts/{account_id}/invitations/{id} | Delete an invitation
*InvitationsApi* | [**invitationResend**](docs/Api/InvitationsApi.md#invitationresend) | **POST** /accounts/{account_id}/invitations/{id}/resend | Resend an invitation
*InvitationsApi* | [**invitationShow**](docs/Api/InvitationsApi.md#invitationshow) | **GET** /accounts/{account_id}/invitations/{id} | Get a single invitation
*InvitationsApi* | [**invitationUpdate**](docs/Api/InvitationsApi.md#invitationupdate) | **PATCH** /accounts/{account_id}/invitations/{id} | Update an invitation
*InvitationsApi* | [**invitationsList**](docs/Api/InvitationsApi.md#invitationslist) | **GET** /accounts/{account_id}/invitations | List invitations
*JobLocalesApi* | [**jobLocaleComplete**](docs/Api/JobLocalesApi.md#joblocalecomplete) | **POST** /projects/{project_id}/jobs/{job_id}/locales/{id}/complete | Complete a job locale
*JobLocalesApi* | [**jobLocaleDelete**](docs/Api/JobLocalesApi.md#joblocaledelete) | **DELETE** /projects/{project_id}/jobs/{job_id}/locales/{id} | Delete a job locale
*JobLocalesApi* | [**jobLocaleReopen**](docs/Api/JobLocalesApi.md#joblocalereopen) | **POST** /projects/{project_id}/jobs/{job_id}/locales/{id}/reopen | Reopen a job locale
*JobLocalesApi* | [**jobLocaleShow**](docs/Api/JobLocalesApi.md#joblocaleshow) | **GET** /projects/{project_id}/jobs/{job_id}/locale/{id} | Get a single job locale
*JobLocalesApi* | [**jobLocaleUpdate**](docs/Api/JobLocalesApi.md#joblocaleupdate) | **PATCH** /projects/{project_id}/jobs/{job_id}/locales/{id} | Update a job locale
*JobLocalesApi* | [**jobLocalesCreate**](docs/Api/JobLocalesApi.md#joblocalescreate) | **POST** /projects/{project_id}/jobs/{job_id}/locales | Create a job locale
*JobLocalesApi* | [**jobLocalesList**](docs/Api/JobLocalesApi.md#joblocaleslist) | **GET** /projects/{project_id}/jobs/{job_id}/locales | List job locales
*JobsApi* | [**jobComplete**](docs/Api/JobsApi.md#jobcomplete) | **POST** /projects/{project_id}/jobs/{id}/complete | Complete a job
*JobsApi* | [**jobCreate**](docs/Api/JobsApi.md#jobcreate) | **POST** /projects/{project_id}/jobs | Create a job
*JobsApi* | [**jobDelete**](docs/Api/JobsApi.md#jobdelete) | **DELETE** /projects/{project_id}/jobs/{id} | Delete a job
*JobsApi* | [**jobKeysCreate**](docs/Api/JobsApi.md#jobkeyscreate) | **POST** /projects/{project_id}/jobs/{id}/keys | Add keys to job
*JobsApi* | [**jobKeysDelete**](docs/Api/JobsApi.md#jobkeysdelete) | **DELETE** /projects/{project_id}/jobs/{id}/keys | Remove keys from job
*JobsApi* | [**jobReopen**](docs/Api/JobsApi.md#jobreopen) | **POST** /projects/{project_id}/jobs/{id}/reopen | Reopen a job
*JobsApi* | [**jobShow**](docs/Api/JobsApi.md#jobshow) | **GET** /projects/{project_id}/jobs/{id} | Get a single job
*JobsApi* | [**jobStart**](docs/Api/JobsApi.md#jobstart) | **POST** /projects/{project_id}/jobs/{id}/start | Start a job
*JobsApi* | [**jobUpdate**](docs/Api/JobsApi.md#jobupdate) | **PATCH** /projects/{project_id}/jobs/{id} | Update a job
*JobsApi* | [**jobsList**](docs/Api/JobsApi.md#jobslist) | **GET** /projects/{project_id}/jobs | List jobs
*KeysApi* | [**keyCreate**](docs/Api/KeysApi.md#keycreate) | **POST** /projects/{project_id}/keys | Create a key
*KeysApi* | [**keyDelete**](docs/Api/KeysApi.md#keydelete) | **DELETE** /projects/{project_id}/keys/{id} | Delete a key
*KeysApi* | [**keyShow**](docs/Api/KeysApi.md#keyshow) | **GET** /projects/{project_id}/keys/{id} | Get a single key
*KeysApi* | [**keyUpdate**](docs/Api/KeysApi.md#keyupdate) | **PATCH** /projects/{project_id}/keys/{id} | Update a key
*KeysApi* | [**keysDelete**](docs/Api/KeysApi.md#keysdelete) | **DELETE** /projects/{project_id}/keys | Delete collection of keys
*KeysApi* | [**keysList**](docs/Api/KeysApi.md#keyslist) | **GET** /projects/{project_id}/keys | List keys
*KeysApi* | [**keysSearch**](docs/Api/KeysApi.md#keyssearch) | **POST** /projects/{project_id}/keys/search | Search keys
*KeysApi* | [**keysTag**](docs/Api/KeysApi.md#keystag) | **PATCH** /projects/{project_id}/keys/tag | Add tags to collection of keys
*KeysApi* | [**keysUntag**](docs/Api/KeysApi.md#keysuntag) | **PATCH** /projects/{project_id}/keys/untag | Remove tags from collection of keys
*LocalesApi* | [**localeCreate**](docs/Api/LocalesApi.md#localecreate) | **POST** /projects/{project_id}/locales | Create a locale
*LocalesApi* | [**localeDelete**](docs/Api/LocalesApi.md#localedelete) | **DELETE** /projects/{project_id}/locales/{id} | Delete a locale
*LocalesApi* | [**localeDownload**](docs/Api/LocalesApi.md#localedownload) | **GET** /projects/{project_id}/locales/{id}/download | Download a locale
*LocalesApi* | [**localeShow**](docs/Api/LocalesApi.md#localeshow) | **GET** /projects/{project_id}/locales/{id} | Get a single locale
*LocalesApi* | [**localeUpdate**](docs/Api/LocalesApi.md#localeupdate) | **PATCH** /projects/{project_id}/locales/{id} | Update a locale
*LocalesApi* | [**localesList**](docs/Api/LocalesApi.md#localeslist) | **GET** /projects/{project_id}/locales | List locales
*MembersApi* | [**memberDelete**](docs/Api/MembersApi.md#memberdelete) | **DELETE** /accounts/{account_id}/members/{id} | Remove a user from the account
*MembersApi* | [**memberShow**](docs/Api/MembersApi.md#membershow) | **GET** /accounts/{account_id}/members/{id} | Get single member
*MembersApi* | [**memberUpdate**](docs/Api/MembersApi.md#memberupdate) | **PATCH** /accounts/{account_id}/members/{id} | Update a member
*MembersApi* | [**membersList**](docs/Api/MembersApi.md#memberslist) | **GET** /accounts/{account_id}/members | List members
*OrdersApi* | [**orderConfirm**](docs/Api/OrdersApi.md#orderconfirm) | **PATCH** /projects/{project_id}/orders/{id}/confirm | Confirm an order
*OrdersApi* | [**orderCreate**](docs/Api/OrdersApi.md#ordercreate) | **POST** /projects/{project_id}/orders | Create a new order
*OrdersApi* | [**orderDelete**](docs/Api/OrdersApi.md#orderdelete) | **DELETE** /projects/{project_id}/orders/{id} | Cancel an order
*OrdersApi* | [**orderShow**](docs/Api/OrdersApi.md#ordershow) | **GET** /projects/{project_id}/orders/{id} | Get a single order
*OrdersApi* | [**ordersList**](docs/Api/OrdersApi.md#orderslist) | **GET** /projects/{project_id}/orders | List orders
*ProjectsApi* | [**projectCreate**](docs/Api/ProjectsApi.md#projectcreate) | **POST** /projects | Create a project
*ProjectsApi* | [**projectDelete**](docs/Api/ProjectsApi.md#projectdelete) | **DELETE** /projects/{id} | Delete a project
*ProjectsApi* | [**projectShow**](docs/Api/ProjectsApi.md#projectshow) | **GET** /projects/{id} | Get a single project
*ProjectsApi* | [**projectUpdate**](docs/Api/ProjectsApi.md#projectupdate) | **PATCH** /projects/{id} | Update a project
*ProjectsApi* | [**projectsList**](docs/Api/ProjectsApi.md#projectslist) | **GET** /projects | List projects
*ReleasesApi* | [**releaseCreate**](docs/Api/ReleasesApi.md#releasecreate) | **POST** /accounts/{account_id}/distributions/{distribution_id}/releases | Create a release
*ReleasesApi* | [**releaseDelete**](docs/Api/ReleasesApi.md#releasedelete) | **DELETE** /accounts/{account_id}/distributions/{distribution_id}/releases/{id} | Delete a release
*ReleasesApi* | [**releasePublish**](docs/Api/ReleasesApi.md#releasepublish) | **POST** /accounts/{account_id}/distributions/{distribution_id}/releases/{id}/publish | Publish a release
*ReleasesApi* | [**releaseShow**](docs/Api/ReleasesApi.md#releaseshow) | **GET** /accounts/{account_id}/distributions/{distribution_id}/releases/{id} | Get a single release
*ReleasesApi* | [**releaseUpdate**](docs/Api/ReleasesApi.md#releaseupdate) | **PATCH** /accounts/{account_id}/distributions/{distribution_id}/releases/{id} | Update a release
*ReleasesApi* | [**releasesList**](docs/Api/ReleasesApi.md#releaseslist) | **GET** /accounts/{account_id}/distributions/{distribution_id}/releases | List releases
*ScreenshotMarkersApi* | [**screenshotMarkerCreate**](docs/Api/ScreenshotMarkersApi.md#screenshotmarkercreate) | **POST** /projects/{project_id}/screenshots/{screenshot_id}/markers | Create a screenshot marker
*ScreenshotMarkersApi* | [**screenshotMarkerDelete**](docs/Api/ScreenshotMarkersApi.md#screenshotmarkerdelete) | **DELETE** /projects/{project_id}/screenshots/{screenshot_id}/markers | Delete a screenshot marker
*ScreenshotMarkersApi* | [**screenshotMarkerShow**](docs/Api/ScreenshotMarkersApi.md#screenshotmarkershow) | **GET** /projects/{project_id}/screenshots/{screenshot_id}/markers/{id} | Get a single screenshot marker
*ScreenshotMarkersApi* | [**screenshotMarkerUpdate**](docs/Api/ScreenshotMarkersApi.md#screenshotmarkerupdate) | **PATCH** /projects/{project_id}/screenshots/{screenshot_id}/markers | Update a screenshot marker
*ScreenshotMarkersApi* | [**screenshotMarkersList**](docs/Api/ScreenshotMarkersApi.md#screenshotmarkerslist) | **GET** /projects/{project_id}/screenshots/{id}/markers | List screenshot markers
*ScreenshotsApi* | [**screenshotCreate**](docs/Api/ScreenshotsApi.md#screenshotcreate) | **POST** /projects/{project_id}/screenshots | Create a screenshot
*ScreenshotsApi* | [**screenshotDelete**](docs/Api/ScreenshotsApi.md#screenshotdelete) | **DELETE** /projects/{project_id}/screenshots/{id} | Delete a screenshot
*ScreenshotsApi* | [**screenshotShow**](docs/Api/ScreenshotsApi.md#screenshotshow) | **GET** /projects/{project_id}/screenshots/{id} | Get a single screenshot
*ScreenshotsApi* | [**screenshotUpdate**](docs/Api/ScreenshotsApi.md#screenshotupdate) | **PATCH** /projects/{project_id}/screenshots/{id} | Update a screenshot
*ScreenshotsApi* | [**screenshotsList**](docs/Api/ScreenshotsApi.md#screenshotslist) | **GET** /projects/{project_id}/screenshots | List screenshots
*SpacesApi* | [**spaceCreate**](docs/Api/SpacesApi.md#spacecreate) | **POST** /accounts/{account_id}/spaces | Create a Space
*SpacesApi* | [**spaceDelete**](docs/Api/SpacesApi.md#spacedelete) | **DELETE** /accounts/{account_id}/spaces/{id} | Delete Space
*SpacesApi* | [**spaceShow**](docs/Api/SpacesApi.md#spaceshow) | **GET** /accounts/{account_id}/spaces/{id} | Get Space
*SpacesApi* | [**spaceUpdate**](docs/Api/SpacesApi.md#spaceupdate) | **PATCH** /accounts/{account_id}/spaces/{id} | Update Space
*SpacesApi* | [**spacesList**](docs/Api/SpacesApi.md#spaceslist) | **GET** /accounts/{account_id}/spaces | List Spaces
*SpacesApi* | [**spacesProjectsCreate**](docs/Api/SpacesApi.md#spacesprojectscreate) | **POST** /accounts/{account_id}/spaces/{space_id}/projects | Add Project
*SpacesApi* | [**spacesProjectsDelete**](docs/Api/SpacesApi.md#spacesprojectsdelete) | **DELETE** /accounts/{account_id}/spaces/{space_id}/projects/{id} | Remove Project
*SpacesApi* | [**spacesProjectsList**](docs/Api/SpacesApi.md#spacesprojectslist) | **GET** /accounts/{account_id}/spaces/{space_id}/projects | List Projects
*StyleGuidesApi* | [**styleguideCreate**](docs/Api/StyleGuidesApi.md#styleguidecreate) | **POST** /projects/{project_id}/styleguides | Create a style guide
*StyleGuidesApi* | [**styleguideDelete**](docs/Api/StyleGuidesApi.md#styleguidedelete) | **DELETE** /projects/{project_id}/styleguides/{id} | Delete a style guide
*StyleGuidesApi* | [**styleguideShow**](docs/Api/StyleGuidesApi.md#styleguideshow) | **GET** /projects/{project_id}/styleguides/{id} | Get a single style guide
*StyleGuidesApi* | [**styleguideUpdate**](docs/Api/StyleGuidesApi.md#styleguideupdate) | **PATCH** /projects/{project_id}/styleguides/{id} | Update a style guide
*StyleGuidesApi* | [**styleguidesList**](docs/Api/StyleGuidesApi.md#styleguideslist) | **GET** /projects/{project_id}/styleguides | List style guides
*TagsApi* | [**tagCreate**](docs/Api/TagsApi.md#tagcreate) | **POST** /projects/{project_id}/tags | Create a tag
*TagsApi* | [**tagDelete**](docs/Api/TagsApi.md#tagdelete) | **DELETE** /projects/{project_id}/tags/{name} | Delete a tag
*TagsApi* | [**tagShow**](docs/Api/TagsApi.md#tagshow) | **GET** /projects/{project_id}/tags/{name} | Get a single tag
*TagsApi* | [**tagsList**](docs/Api/TagsApi.md#tagslist) | **GET** /projects/{project_id}/tags | List tags
*TranslationsApi* | [**translationCreate**](docs/Api/TranslationsApi.md#translationcreate) | **POST** /projects/{project_id}/translations | Create a translation
*TranslationsApi* | [**translationExclude**](docs/Api/TranslationsApi.md#translationexclude) | **PATCH** /projects/{project_id}/translations/{id}/exclude | Exclude a translation from export
*TranslationsApi* | [**translationInclude**](docs/Api/TranslationsApi.md#translationinclude) | **PATCH** /projects/{project_id}/translations/{id}/include | Revoke exclusion of a translation in export
*TranslationsApi* | [**translationReview**](docs/Api/TranslationsApi.md#translationreview) | **PATCH** /projects/{project_id}/translations/{id}/review | Review a translation
*TranslationsApi* | [**translationShow**](docs/Api/TranslationsApi.md#translationshow) | **GET** /projects/{project_id}/translations/{id} | Get a single translation
*TranslationsApi* | [**translationUnverify**](docs/Api/TranslationsApi.md#translationunverify) | **PATCH** /projects/{project_id}/translations/{id}/unverify | Mark a translation as unverified
*TranslationsApi* | [**translationUpdate**](docs/Api/TranslationsApi.md#translationupdate) | **PATCH** /projects/{project_id}/translations/{id} | Update a translation
*TranslationsApi* | [**translationVerify**](docs/Api/TranslationsApi.md#translationverify) | **PATCH** /projects/{project_id}/translations/{id}/verify | Verify a translation
*TranslationsApi* | [**translationsByKey**](docs/Api/TranslationsApi.md#translationsbykey) | **GET** /projects/{project_id}/keys/{key_id}/translations | List translations by key
*TranslationsApi* | [**translationsByLocale**](docs/Api/TranslationsApi.md#translationsbylocale) | **GET** /projects/{project_id}/locales/{locale_id}/translations | List translations by locale
*TranslationsApi* | [**translationsExclude**](docs/Api/TranslationsApi.md#translationsexclude) | **PATCH** /projects/{project_id}/translations/exclude | Set exclude from export flag on translations selected by query
*TranslationsApi* | [**translationsInclude**](docs/Api/TranslationsApi.md#translationsinclude) | **PATCH** /projects/{project_id}/translations/include | Remove exlude from import flag from translations selected by query
*TranslationsApi* | [**translationsList**](docs/Api/TranslationsApi.md#translationslist) | **GET** /projects/{project_id}/translations | List all translations
*TranslationsApi* | [**translationsReview**](docs/Api/TranslationsApi.md#translationsreview) | **PATCH** /projects/{project_id}/translations/review | Review translations selected by query
*TranslationsApi* | [**translationsSearch**](docs/Api/TranslationsApi.md#translationssearch) | **POST** /projects/{project_id}/translations/search | Search translations
*TranslationsApi* | [**translationsUnverify**](docs/Api/TranslationsApi.md#translationsunverify) | **PATCH** /projects/{project_id}/translations/unverify | Mark translations selected by query as unverified
*TranslationsApi* | [**translationsVerify**](docs/Api/TranslationsApi.md#translationsverify) | **PATCH** /projects/{project_id}/translations/verify | Verify translations selected by query
*UploadsApi* | [**uploadCreate**](docs/Api/UploadsApi.md#uploadcreate) | **POST** /projects/{project_id}/uploads | Upload a new file
*UploadsApi* | [**uploadShow**](docs/Api/UploadsApi.md#uploadshow) | **GET** /projects/{project_id}/uploads/{id} | View upload details
*UploadsApi* | [**uploadsList**](docs/Api/UploadsApi.md#uploadslist) | **GET** /projects/{project_id}/uploads | List uploads
*UsersApi* | [**showUser**](docs/Api/UsersApi.md#showuser) | **GET** /user | Show current User
*VersionsHistoryApi* | [**versionShow**](docs/Api/VersionsHistoryApi.md#versionshow) | **GET** /projects/{project_id}/translations/{translation_id}/versions/{id} | Get a single version
*VersionsHistoryApi* | [**versionsList**](docs/Api/VersionsHistoryApi.md#versionslist) | **GET** /projects/{project_id}/translations/{translation_id}/versions | List all versions
*WebhooksApi* | [**webhookCreate**](docs/Api/WebhooksApi.md#webhookcreate) | **POST** /projects/{project_id}/webhooks | Create a webhook
*WebhooksApi* | [**webhookDelete**](docs/Api/WebhooksApi.md#webhookdelete) | **DELETE** /projects/{project_id}/webhooks/{id} | Delete a webhook
*WebhooksApi* | [**webhookShow**](docs/Api/WebhooksApi.md#webhookshow) | **GET** /projects/{project_id}/webhooks/{id} | Get a single webhook
*WebhooksApi* | [**webhookTest**](docs/Api/WebhooksApi.md#webhooktest) | **POST** /projects/{project_id}/webhooks/{id}/test | Test a webhook
*WebhooksApi* | [**webhookUpdate**](docs/Api/WebhooksApi.md#webhookupdate) | **PATCH** /projects/{project_id}/webhooks/{id} | Update a webhook
*WebhooksApi* | [**webhooksList**](docs/Api/WebhooksApi.md#webhookslist) | **GET** /projects/{project_id}/webhooks | List webhooks


## Documentation For Models

 - [Account](docs/Model/Account.md)
 - [AccountDetails](docs/Model/AccountDetails.md)
 - [AccountDetails1](docs/Model/AccountDetails1.md)
 - [AffectedCount](docs/Model/AffectedCount.md)
 - [AffectedResources](docs/Model/AffectedResources.md)
 - [Authorization](docs/Model/Authorization.md)
 - [AuthorizationCreateParameters](docs/Model/AuthorizationCreateParameters.md)
 - [AuthorizationUpdateParameters](docs/Model/AuthorizationUpdateParameters.md)
 - [AuthorizationWithToken](docs/Model/AuthorizationWithToken.md)
 - [AuthorizationWithToken1](docs/Model/AuthorizationWithToken1.md)
 - [BitbucketSync](docs/Model/BitbucketSync.md)
 - [BitbucketSyncExportParameters](docs/Model/BitbucketSyncExportParameters.md)
 - [BitbucketSyncExportResponse](docs/Model/BitbucketSyncExportResponse.md)
 - [BitbucketSyncImportParameters](docs/Model/BitbucketSyncImportParameters.md)
 - [BlacklistedKey](docs/Model/BlacklistedKey.md)
 - [BlacklistedKeyCreateParameters](docs/Model/BlacklistedKeyCreateParameters.md)
 - [BlacklistedKeyUpdateParameters](docs/Model/BlacklistedKeyUpdateParameters.md)
 - [Branch](docs/Model/Branch.md)
 - [BranchCreateParameters](docs/Model/BranchCreateParameters.md)
 - [BranchMergeParameters](docs/Model/BranchMergeParameters.md)
 - [BranchUpdateParameters](docs/Model/BranchUpdateParameters.md)
 - [Comment](docs/Model/Comment.md)
 - [CommentCreateParameters](docs/Model/CommentCreateParameters.md)
 - [CommentMarkReadParameters](docs/Model/CommentMarkReadParameters.md)
 - [CommentUpdateParameters](docs/Model/CommentUpdateParameters.md)
 - [Distribution](docs/Model/Distribution.md)
 - [DistributionCreateParameters](docs/Model/DistributionCreateParameters.md)
 - [DistributionPreview](docs/Model/DistributionPreview.md)
 - [DistributionUpdateParameters](docs/Model/DistributionUpdateParameters.md)
 - [Format](docs/Model/Format.md)
 - [GitlabSync](docs/Model/GitlabSync.md)
 - [GitlabSyncExport](docs/Model/GitlabSyncExport.md)
 - [GitlabSyncExportParameters](docs/Model/GitlabSyncExportParameters.md)
 - [GitlabSyncHistory](docs/Model/GitlabSyncHistory.md)
 - [GitlabSyncImportParameters](docs/Model/GitlabSyncImportParameters.md)
 - [Glossary](docs/Model/Glossary.md)
 - [GlossaryCreateParameters](docs/Model/GlossaryCreateParameters.md)
 - [GlossaryTerm](docs/Model/GlossaryTerm.md)
 - [GlossaryTermCreateParameters](docs/Model/GlossaryTermCreateParameters.md)
 - [GlossaryTermTranslation](docs/Model/GlossaryTermTranslation.md)
 - [GlossaryTermTranslationCreateParameters](docs/Model/GlossaryTermTranslationCreateParameters.md)
 - [GlossaryTermTranslationUpdateParameters](docs/Model/GlossaryTermTranslationUpdateParameters.md)
 - [GlossaryTermUpdateParameters](docs/Model/GlossaryTermUpdateParameters.md)
 - [GlossaryUpdateParameters](docs/Model/GlossaryUpdateParameters.md)
 - [Invitation](docs/Model/Invitation.md)
 - [InvitationCreateParameters](docs/Model/InvitationCreateParameters.md)
 - [InvitationUpdateParameters](docs/Model/InvitationUpdateParameters.md)
 - [Job](docs/Model/Job.md)
 - [JobCompleteParameters](docs/Model/JobCompleteParameters.md)
 - [JobCreateParameters](docs/Model/JobCreateParameters.md)
 - [JobDetails](docs/Model/JobDetails.md)
 - [JobDetails1](docs/Model/JobDetails1.md)
 - [JobKeysCreateParameters](docs/Model/JobKeysCreateParameters.md)
 - [JobLocale](docs/Model/JobLocale.md)
 - [JobLocaleCompleteParameters](docs/Model/JobLocaleCompleteParameters.md)
 - [JobLocaleReopenParameters](docs/Model/JobLocaleReopenParameters.md)
 - [JobLocaleUpdateParameters](docs/Model/JobLocaleUpdateParameters.md)
 - [JobLocalesCreateParameters](docs/Model/JobLocalesCreateParameters.md)
 - [JobPreview](docs/Model/JobPreview.md)
 - [JobReopenParameters](docs/Model/JobReopenParameters.md)
 - [JobStartParameters](docs/Model/JobStartParameters.md)
 - [JobUpdateParameters](docs/Model/JobUpdateParameters.md)
 - [KeyCreateParameters](docs/Model/KeyCreateParameters.md)
 - [KeyPreview](docs/Model/KeyPreview.md)
 - [KeyUpdateParameters](docs/Model/KeyUpdateParameters.md)
 - [KeysSearchParameters](docs/Model/KeysSearchParameters.md)
 - [KeysTagParameters](docs/Model/KeysTagParameters.md)
 - [KeysUntagParameters](docs/Model/KeysUntagParameters.md)
 - [Locale](docs/Model/Locale.md)
 - [LocaleCreateParameters](docs/Model/LocaleCreateParameters.md)
 - [LocaleDetails](docs/Model/LocaleDetails.md)
 - [LocaleDetails1](docs/Model/LocaleDetails1.md)
 - [LocalePreview](docs/Model/LocalePreview.md)
 - [LocaleStatistics](docs/Model/LocaleStatistics.md)
 - [LocaleUpdateParameters](docs/Model/LocaleUpdateParameters.md)
 - [Member](docs/Model/Member.md)
 - [MemberUpdateParameters](docs/Model/MemberUpdateParameters.md)
 - [OrderConfirmParameters](docs/Model/OrderConfirmParameters.md)
 - [OrderCreateParameters](docs/Model/OrderCreateParameters.md)
 - [Project](docs/Model/Project.md)
 - [ProjectCreateParameters](docs/Model/ProjectCreateParameters.md)
 - [ProjectDetails](docs/Model/ProjectDetails.md)
 - [ProjectDetails1](docs/Model/ProjectDetails1.md)
 - [ProjectLocales](docs/Model/ProjectLocales.md)
 - [ProjectLocales1](docs/Model/ProjectLocales1.md)
 - [ProjectShort](docs/Model/ProjectShort.md)
 - [ProjectUpdateParameters](docs/Model/ProjectUpdateParameters.md)
 - [Release](docs/Model/Release.md)
 - [ReleaseCreateParameters](docs/Model/ReleaseCreateParameters.md)
 - [ReleasePreview](docs/Model/ReleasePreview.md)
 - [ReleaseUpdateParameters](docs/Model/ReleaseUpdateParameters.md)
 - [Screenshot](docs/Model/Screenshot.md)
 - [ScreenshotCreateParameters](docs/Model/ScreenshotCreateParameters.md)
 - [ScreenshotMarker](docs/Model/ScreenshotMarker.md)
 - [ScreenshotMarkerCreateParameters](docs/Model/ScreenshotMarkerCreateParameters.md)
 - [ScreenshotMarkerUpdateParameters](docs/Model/ScreenshotMarkerUpdateParameters.md)
 - [ScreenshotUpdateParameters](docs/Model/ScreenshotUpdateParameters.md)
 - [Space](docs/Model/Space.md)
 - [SpaceCreateParameters](docs/Model/SpaceCreateParameters.md)
 - [SpaceUpdateParameters](docs/Model/SpaceUpdateParameters.md)
 - [SpacesProjectsCreateParameters](docs/Model/SpacesProjectsCreateParameters.md)
 - [Styleguide](docs/Model/Styleguide.md)
 - [StyleguideCreateParameters](docs/Model/StyleguideCreateParameters.md)
 - [StyleguideDetails](docs/Model/StyleguideDetails.md)
 - [StyleguideDetails1](docs/Model/StyleguideDetails1.md)
 - [StyleguidePreview](docs/Model/StyleguidePreview.md)
 - [StyleguideUpdateParameters](docs/Model/StyleguideUpdateParameters.md)
 - [Tag](docs/Model/Tag.md)
 - [TagCreateParameters](docs/Model/TagCreateParameters.md)
 - [TagWithStats](docs/Model/TagWithStats.md)
 - [TagWithStats1](docs/Model/TagWithStats1.md)
 - [TagWithStats1Statistics](docs/Model/TagWithStats1Statistics.md)
 - [TagWithStats1Statistics1](docs/Model/TagWithStats1Statistics1.md)
 - [Translation](docs/Model/Translation.md)
 - [TranslationCreateParameters](docs/Model/TranslationCreateParameters.md)
 - [TranslationDetails](docs/Model/TranslationDetails.md)
 - [TranslationDetails1](docs/Model/TranslationDetails1.md)
 - [TranslationExcludeParameters](docs/Model/TranslationExcludeParameters.md)
 - [TranslationIncludeParameters](docs/Model/TranslationIncludeParameters.md)
 - [TranslationKey](docs/Model/TranslationKey.md)
 - [TranslationKeyDetails](docs/Model/TranslationKeyDetails.md)
 - [TranslationKeyDetails1](docs/Model/TranslationKeyDetails1.md)
 - [TranslationOrder](docs/Model/TranslationOrder.md)
 - [TranslationReviewParameters](docs/Model/TranslationReviewParameters.md)
 - [TranslationUnverifyParameters](docs/Model/TranslationUnverifyParameters.md)
 - [TranslationUpdateParameters](docs/Model/TranslationUpdateParameters.md)
 - [TranslationVerifyParameters](docs/Model/TranslationVerifyParameters.md)
 - [TranslationVersion](docs/Model/TranslationVersion.md)
 - [TranslationVersionWithUser](docs/Model/TranslationVersionWithUser.md)
 - [TranslationVersionWithUser1](docs/Model/TranslationVersionWithUser1.md)
 - [TranslationsExcludeParameters](docs/Model/TranslationsExcludeParameters.md)
 - [TranslationsIncludeParameters](docs/Model/TranslationsIncludeParameters.md)
 - [TranslationsReviewParameters](docs/Model/TranslationsReviewParameters.md)
 - [TranslationsSearchParameters](docs/Model/TranslationsSearchParameters.md)
 - [TranslationsUnverifyParameters](docs/Model/TranslationsUnverifyParameters.md)
 - [TranslationsVerifyParameters](docs/Model/TranslationsVerifyParameters.md)
 - [Upload](docs/Model/Upload.md)
 - [UploadCreateParameters](docs/Model/UploadCreateParameters.md)
 - [UploadSummary](docs/Model/UploadSummary.md)
 - [User](docs/Model/User.md)
 - [UserPreview](docs/Model/UserPreview.md)
 - [Webhook](docs/Model/Webhook.md)
 - [WebhookCreateParameters](docs/Model/WebhookCreateParameters.md)
 - [WebhookUpdateParameters](docs/Model/WebhookUpdateParameters.md)


## Documentation For Authorization



## Basic


- **Type**: HTTP basic authentication



## Token


- **Type**: API key
- **API key parameter name**: Authorization
- **Location**: HTTP header



## Author

support@phrase.com

