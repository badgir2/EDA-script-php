# EDA-script-php
login to EDA Portal (Energiewirtschaftlicher Datenaustausch GmbH Vienna/Austria - https://www.eda.at/portal) for energy community administrators in Austria
feel free to use and adapt the script to your own needs

## Available Methods: 

- login(): Login with [EDA Portal](https://prod.eda-portal.at/api/login) credentials and get token for further requests
- users(): details about the person who has access to the portal
- pwaUser(): not sure about the use of this request
- energycommunities: basic data about the energy community
- energycommunities STRING: details about the members of the energy community
- meter data: shows metering data of the energy community in a selected time range
- kpiData: data of self-sufficency and consumption of the energy comunity


## Usage
```php
<?php

// login to EDA Portal (Energiewirtschaftlicher Datenaustausch GmbH Vienna/Austria)

// use at your own risk 

// first create /temp directory 

$cookieFile = dirname(__DIR__)."/temp/cookieEDA.txt";

//if file doesn't exist
if(!file_exists($cookieFile)) {
    //fopen for writing
    $fh = fopen($cookieFile, "w");
    //write
    fwrite($fh, "");
    //close
    fclose($fh);
}

// write your access credentials for the EDA Portal

$username = 'email@example.com'; // the email used for registration 
$password = 'password'; // plain text password

// login Url

$loginUrl = 'https://prod.eda-portal.at/api/login';

// compose login string and convert to json

$login = array('email' => $username,
				   'password' => $password
				   );
$login = json_encode($login);


//do login

//init curl
$ch = curl_init();

//Set the URL to work with
curl_setopt($ch, CURLOPT_URL, $loginUrl);

// ENABLE HTTP POST
curl_setopt($ch, CURLOPT_POST, 1);

//Set the post parameters
curl_setopt($ch, CURLOPT_POSTFIELDS, $login);

//Handle cookies for the login
curl_setopt($ch, CURLOPT_COOKIEJAR, $cookieFile);
curl_setopt($ch, CURLOPT_COOKIEFILE, $cookieFile);

//Setting CURLOPT_RETURNTRANSFER variable to 1 will force cURL
//not to print out the results of its query.
//Instead, it will return the results as a string return value
//from curl_exec() instead of the usual true/false.

curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

$headers = array();
$headers[] = 'Content-Type: application/json';
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Cache-Control: no-cache';
$headers[] = 'Content-Type: application/json';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

//execute the request (the login)
$store = curl_exec($ch);

//the login is now done and you can continue to get the
//protected content.

//execute the request
$result = curl_exec($ch);

curl_close($ch);

// decode json reult

$json = json_decode($result, true);

// show result

/*
a typical result would be:

Array
(
    [token] => [a JWT base64 encoded string which contains a json string and other unreadable data: {"typ":"JWT","alg":"RS256"}{"iss":"https://prod-api.eda-portal.at/api/v4/auth/login","iat":*unix date*,"exp":*unix date*,"nbf":*unix date*,"jti":"*alphanumeric string*","user_id":"*36 digit alphanumeric string*","sub":"*4 digit number*","prv":"*another string*"}]
    [exp] => [expiry date of the token; usually 1 week after the call]
    [user] => Array
        (
            [confirmedAt] => [date and time when access has been given by the EDA Portal to the user]
            [defaultLocale] => de
            [familyName] => [the family name of the person who has applied for access to the EDA Portal]
            [givenName] => [the first name of the person who has applied for access to the EDA Portal]
            [id] => [an alphanumeric string with 16 digits which identifies the user also in future calls]
            [additionalAttributes] => Array
                (
                    [postalCode] => [postal code]
                    [streetAddress] => [address]
                    [addressCountry] => [country]
                    [acceptedTermsAt] => [date and time when access has been given by the EDA Portal to the user]
                    [addressLocality] => [city]
                    [acceptedPrivacyAt] => [date and time when access has been given by the EDA Portal to the user]
                )

            [receivedDataLinks] => Array
                (
                )

            [role] => PWA_USER
            [storedFilters] => Array
                (
                )

            [validReceivedDataLinks] => Array
                (
                )

            [watchLists] => Array
                (
                )

        )

)
*/

echo '<br>login result: <pre>';
print_r($json);
echo '</pre>';

// extract token from result, needed for further requests

$token = $json['token'];


// users

// more details about the person who has access to the portal

// this request must contain the $token variable received from the previous request

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod.eda-portal.at/api/users');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_COOKIEJAR, $cookieFile);
curl_setopt($ch, CURLOPT_COOKIEFILE, $cookieFile);


$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>users result: <pre>';
print_r($json);
echo '</pre>';


// pwaUser GET

// needs token variable from login response

// not sure about the use of this request; gives funny answer from the system: [isHotlineUser] => Hurra die Hex ist tot.

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod-api.eda-portal.at/api/pwa/pwaUser');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_COOKIEJAR, $cookieFile);
curl_setopt($ch, CURLOPT_COOKIEFILE, $cookieFile);


$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>pwaUser result: <pre>';
print_r($json);
echo '</pre>';


// energycommunities GET

// this request will produce basic data about the energy community; token variable from login request is needed. 

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod-api.eda-portal.at/api/pwa/energycommunities');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_COOKIEJAR, $cookieFile);
curl_setopt($ch, CURLOPT_COOKIEFILE, $cookieFile);

$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>energycommunities result: <pre>';
print_r($json);
echo '</pre>';

// extract id string from the previous request (there may be more strings if more communities are registered with the same user; cannot check this)

$string = $json['data'][0]['id'];



// energycommunities STRING

// this call will give all details about the members of the energy community associated with the id string; needs the id string from the prevoius request and token varaible from login

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod-api.eda-portal.at/api/pwa/energycommunities/'.$string);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_COOKIEJAR, $cookieFile);
curl_setopt($ch, CURLOPT_COOKIEFILE, $cookieFile);


$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>energycommunities STRING result: <pre>';
print_r($json);
echo '</pre>';


// meter data

// shows metering data of the energy community associated with the id string on a daily base related to the selected period; need token variable, id string, min and max data

// data format for min and max: UTC date format yyyy-mm-ddThh:mm, example 2025-05-11T00:00; don't forget 'Content-Type: application/json' in headers to be sent

$min = '0000-00-00T00:00';
$max = '0000-00-00T23:45';

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod-api.eda-portal.at/api/pwa/energycommunities/'.$string.'/meterdata');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, "{\"time\":{\"in\":{\"min\":\"".$min."\",\"max\":\"".$max."\"}},\"energyCommunityId\":\"".$string."\",\"groupBy\":\"day\"}");

$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
$headers[] = 'Content-Type: application/json';

curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>meter data day value result: <pre>';
print_r($json);
echo '</pre>';


// kpiData 

// data of self-sufficency and consumption of the energy comunity, needs token, id string and min/max date values; can eventually be grouped by day/month/year - not tested

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, 'https://prod-api.eda-portal.at/api/pwa/energycommunities/'.$string.'/kpiData');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, "{\"energyCommunityId\":\"".$string."\",\"time\":{\"in\":{\"min\":\"".$min."\",\"max\":\"".$max."\"}},\"groupBy\":\"day\"}");

$headers = array();
$headers[] = 'Accept: application/json, text/plain, */*';
$headers[] = 'Accept-Language: en-US,en;q=0.9,it;q=0.8,de-DE;q=0.7,de;q=0.6';
$headers[] = 'Authorization: Bearer '.$token;
$headers[] = 'Cache-Control: no-cache';
$headers[] = 'Content-Type: application/json';
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

$result = curl_exec($ch);
if (curl_errno($ch)) {
    echo 'Error:' . curl_error($ch);
}
curl_close($ch);

$json = json_decode($result, true);

echo '<br>kpiData result: <pre>';
print_r($json);
echo '</pre>';


?>


```
## Requirements
- php-curl

## Disclaimer
This is not an official API of EDA (Energiewirtschaftlicher Datenaustausch GmbH Vienna/Austria).
