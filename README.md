## Installation

install the package via composer:

```bash
composer require yorcreative/laravel-urlshortener
```

Publish the packages assets.
```bash
php artisan vendor:publish --provider="YorCreative\UrlShortener\UrlShortenerServiceProvider"
```

Run migrations.
```bash
php artisan migrate
```

## Upgrade Guides

[Upgrading to v2.x from v1.x](https://github.com/YorCreative/Laravel-UrlShortener/wiki/Upgrading-to-2.x-from-1.x)

## Usage

Building Short Urls

```php
/**
* Basic
 */
$url = UrlService::shorten('something-extremely-long.com/even/longer?ref=with&some=thingelselonger')
        ->build(); 
// http(s)://host/prefix/identifier;

/**
* Advanced
 */
$url = UrlService::shorten('something-extremely-long.com/even/longer?ref=with&some=thingelselonger')
        ->withActivation(Carbon::now()->addHour()->timestamp)
        ->withExpiration(Carbon::now()->addDay()->timestamp)
        ->withOpenLimit(2)
        ->withOwnership(Model::find(1))
        ->withPassword('password')
        ->withTracing([
            'utm_id' => 't123',
            'utm_campaign' => 'campaign_name',
            'utm_source' => 'linkedin',
            'utm_medium' => 'social',      
        ])
        ->build();
// http(s)://host/prefix/identifier;
```

Finding Existing Short Urls

```php
/**
 * Find a Short URL by its identifier 
 */
$shortUrl = UrlService::findByIdentifier('identifier');
// returns instance of ShortUrl Model.


/**
 * Find a Short URL by its hashed signature
 */
$shortUrl = UrlService::findByHash(md5('long_url'));
// returns instance of ShortUrl Model.


/**
 * Find a Short URL by its plain text long url string 
 */
$shortUrl = UrlService::findByPlainText('long_url');
// returns instance of ShortUrl Model. 

/**
 * Find shortUrls by UTM combinations.
 * 
 * Note* This method only accepts the following array fields:
 *  - utm_id
 *  - utm_campaign
 *  - utm_source
 *  - utm_medium
 *  - utm_content
 *  - utm_term
 */
$shortUrlCollection = UrlService::findByUtmCombination([
    'utm_campaign' => 'alpha',
    'utm_source' => 'bravo',
    'utm_medium' => 'testing'
])
// returns an instance of Eloquent Collection of ShortUrl Models.
```

Getting Click Information

```php
$clicks = ClickService::get()->toArray();

dd($clicks);
[
    'results' => [
        [
            'id' => ...,
            'created_at' => ...,
            'short_url' => [
                'id' => ...,
                'identifier' => ...,
                'hashed' => ...,
                'plain_text' => ...,
                'limit' => ...,
                'tracing' => [
                    'id' => ...,
                    'utm_id' => ...,
                    'utm_source' => ...,
                    'utm_medium' => ...,
                    'utm_campaign' => ...,
                    'utm_content' => ...,
                    'utm_term' => ...,
                ]
                'created_at' => ...,
                'updated_at' => ...
            ],
            'location' => [
                'id' => ...,
                'ip' => ...,
                'countryName' => ...,
                'countryCode' => ...,
                'regionCode' => ...,
                'regionName' => ...,
                'cityName' => ...,
                'zipCode' => ...,
                'isoCode' => ...,
                'postalCode' => ...,
                'latitude' => ...,
                'longitude' => ...,
                'metroCode' => ...,
                'areaCode' => ...,
                'timezone' => ...,
                'created_at' => ...,
                'updated_at' => ...
            ],
            'outcome' => [
                'id' => ...,
                'name' => ...,
                'alias' => ...,
            ],
        ]  
    ],
    'total' => 1
];
```

Getting Click Information and Filtering on Ownership

```php
$clicks = ClickService::get([
    'ownership' =>  [
        Model::find(1),
        Model::find(2)
    ]        
]);
```


Filter on Outcome

```php
$clicks = ClickService::get([
    'outcome' => [
        1, // successful_routed
        2, // successful_protected
        3, // failure_password
        4, // failure_expiration
        5  // failure_limit
    ]        
]);
```
Filter on the Click's YorShortUrl Status

```php
$clicks = ClickService::get([
    'status' => [
        'active',
        'expired',
        'expiring' // within 30 minutes of expiring
    ]        
]);
```

Filtered on YorShortUrl Identifier(s)

```php
$clicks = ClickService::get([
    'identifier' => [
         'xyz',
         'yxz'
    ]
]);
```

Filtered Clicks by UTM parameter(s). These Can be filtered together or individually.
```php
$clicks = ClickService::get([
    'utm_id' => [
         'xyz',
         'yxz'
    ],
    'utm_source' => [
         'linkedin',
         'facebook'
    ],
    'utm_medium' => [
         'social'
    ],
    'utm_campaign' => [
         'sponsored',
         'affiliate'
    ],
    'utm_content' => [
         'xyz',
         'yxz'
    ],
    'utm_term' => [
         'marketing+software',
         'short+url'
    ],
]);
```

Iterate Through Results With Batches

```php
$clicks = ClickService::get([
    'limit' => 500
    'offset' => 1500
]); 
  
$clicks->get('results');
$clicks->get('total');
```

Putting it all Together

```php
/**
 * Get the successfully routed clicks for all active short urls that are owned by Model IDs 1,2,3 and 4.
 * Set the offset of results by 1500 clicks and limit by the results by 500.
 */
$clicks = ClickService::get([
    'ownership' => Model::whereIn('id', [1,2,3,4])->get()->toArray(),
    'outcome' => [
        3 // successful_routed
    ],
    'status' => [
        'active'
    ],    
    'utm_campaign' => [
        'awareness'
    ],      
    'utm_source' => [
        'github'
    ],
    'limit' => 500
    'offset' => 1500
]);
```
## Testing

```bash
composer test
```
