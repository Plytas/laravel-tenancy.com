Tenancy is heavily driven by events. For event listeners to properly work, you
have to use the repositories to create new websites, hostnames and customers.

##### Create website

```php
use Hyn\Tenancy\Models\Website;
use Hyn\Tenancy\Contracts\Repositories\WebsiteRepository;

$website = new Website;
app(WebsiteRepository::class)->create($website);
dd($website->uuid); // Unique id
```

###### UUID

Each website will be given a unique identifier automatically. To prevent
guessing URLs and to add a slight increased level of security this is a
random hash. 

The UUID is used for the database username and database name. In addition
I'd recommend using this unique string for every public facing URL, eg
an admin dashboard.

By default tenancy will create a `uuid` automatically
using a UUID generator, you can replace that class in the 
`tenancy.php > website > random-id-generator`, make sure it implements
`Hyn\Tenancy\Contracts\Website\UuidGenerator`.

If you want to force your own UUID for the website, without creating a full
fledged UUID generator, set the `uuid` property of your website before passing
it to the repository. 

```php
$website = new Website;
// Implement custom random hash using Laravels Str::random
$website->uuid = str_random(10);
app(WebsiteRepository::class)->create($website);
dd($website->uuid); // Random string of length 10
```

##### Create and connect hostname

```php
use Hyn\Tenancy\Models\Hostname;
use Hyn\Tenancy\Contracts\Repositories\HostnameRepository;

$hostname = new Hostname;
$hostname->fqdn = 'luceos.demo.app';
app(HostnameRepository::class)->attach($hostname, $website);
dd($website->hostnames); // Collection with $hostname
```

##### Switch to new tenant

```php
use Hyn\Tenancy\Environment;
$tenancy = app(Environment::class);

$tenancy->hostname($hostname);

$tenancy->hostname(); // resolves $hostname as currently active hostname
$tenancy->website(); // resolves $website
$tenancy->customer(); // resolves $customer

$tenancy->identifyHostname(); // resets resolving $hostname by using the Request
```

##### Querying

In case you would like to query the database for hostnames, websites or customers. Use the
`query()` method of the repositories, this will return a [`Illuminate\Database\Eloquent\Builder`][query-builder]
instance.

[query-builder]: https://laravel.com/docs/5.5/queries