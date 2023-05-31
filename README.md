# Microsoft

```bash
composer require socialiteproviders/microsoft
```

## Installation & Basic Usage

Please see the [Base Installation Guide](https://socialiteproviders.com/usage/), then follow the provider specific instructions below.

### Add configuration to `config/services.php`

```php
'microsoft' => [    
  'client_id' => env('MICROSOFT_CLIENT_ID'),  
  'client_secret' => env('MICROSOFT_CLIENT_SECRET'),  
  'redirect' => env('MICROSOFT_REDIRECT_URI') 
],
```

### Add provider event listener

Configure the package's listener to listen for `SocialiteWasCalled` events.

Add the event to your `listen[]` array in `app/Providers/EventServiceProvider`. See the [Base Installation Guide](https://socialiteproviders.com/usage/) for detailed instructions.

```php
protected $listen = [
    \SocialiteProviders\Manager\SocialiteWasCalled::class => [
        // ... other providers
        \SocialiteProviders\Microsoft\MicrosoftExtendSocialite::class.'@handle',
    ],
];
```

### Usage

You should now be able to use the provider like you would regularly use Socialite (assuming you have the facade installed):

```php
return Socialite::driver('microsoft')->redirect();
```

### Extended features

#### Tenant Details
You can also retrieve Tenant information at the same time as you retrieve users, this can be useful if you need to allow only your tenant/s or filter certain tenants.

To do this you first need to edit your `config/services.php` file and within your microsoft settings array include 'include_tenant_info' like the following:

```php
'microsoft' => [
        'client_id' => env('MICROSOFT_CLIENT_ID'),
        'client_secret' => env('MICROSOFT_CLIENT_SECRET'),
        'redirect' => env('MICROSOFT_REDIRECT_URI'),
        'tenant' => 'common',
        'include_tenant_info' => true,
    ],
```

The default tenant fields returned are:
* ID
* displayName
* city
* country
* countryLetterCode
* state
* street
* verifiedDomains

[Additional fields](#additional-tenant-fields-tenantfields) can be included.
 
#### Tenant types

The [supported values (defined by MS Identity Platform)](https://learn.microsoft.com/en-au/azure/active-directory/develop/active-directory-v2-protocols#endpoints)
for 'tenant' are listed below and can be used to control who can sign into the application.
- `common` - for both Microsoft accounts and work or school accounts (**most permissive**),
- `organizations` - for work or school accounts only,
- `consumers` - for Microsoft accounts only (_only services like Xbox, Teams for Life, or Outlook_),
- `tenant identifiers` - such as the tenant ID or domain name (**most restrictive**).

**Note:** when configuring the services.php microsoft entry with 

- `tenant => 'common'`
- `include_tenant_info => true`

and attempting to login with a 'consumer' account, the user's tenant value will be null
 
e.g. 

```
$user = Socialite::driver('microsoft')->user();
if ($user->tenant === null) {

    // do some consumer/public specific workflow
    
} else {

    // do your work / school tenant workflow
    Log::info(sprintf("Tenant found - %s", $user->tenant->displayName));
     
}
```


#### Additional tenant fields `tenant_fields`

Any additional fields can be returned with the attribute names detailed [here](https://learn.microsoft.com/en-us/graph/api/resources/organization?view=graph-rest-1.0).

e.g. `'tenantType', 'technicalNotificationMails'` can be requested as such

```
    'microsoft' => [
        'client_id' => env('MICROSOFT_CLIENT_ID'), 
        'client_secret' => env('MICROSOFT_CLIENT_SECRET'),
        'redirect' => env('MICROSOFT_REDIRECT_URI'), 
        'tenant' => env('MICROSOFT_TENANT_ID', 'common'), 
        'include_tenant_info' => true,
        'tenant_fields' => [ 'tenantType', 'technicalNotificationMails' ],
        'include_avatar' => true,
        'include_avatar_size' => '648x648',
    ], 
```