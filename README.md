# Laravel Guestbook

> Following along with @rachidlaasri excellent tutorial on scotch.io: [Build a Guestbook with Laravel and Vue.js](https://scotch.io/tutorials/build-a-guestbook-with-laravel-and-vuejs)



### Generate the application

Use [this script](https://gist.github.com/connor11528/fcfbdb63bc9633a54f40f0a66e3d3f2e) to quickly scaffold a Laravel 5 app.

```
./create_laravel_app.sh guestbook
cd guestbook
php artisan serve
npm run watch
```

### Add a database

Add sqlite database:

```
touch database/database.sqlite 
```

The only line you want in the **.env** file related to the db is: `DB_CONNECTION=sqlite`, get rid of the host, port, username business.

Sqlite is great for development but you probably wouldn't use it for a production app. You could go with MySQL.
If you're interested in hooking up Laravel 5 to MySQL check this tutorial:

[Build an online forum with Laravel — Initial Setup and Seeding (Part 1)](https://medium.com/@connorleech/build-an-online-forum-with-laravel-initial-setup-and-seeding-part-1-a53138d1fffc)

`</shameless-plug>`

### Seeding

In the tutorial we set up a model factory for generating Signatures in our database and use tinker to generate the records. Another approach is to use database seeders.
Database seeders populate our database by running `php artisan db:seed`. This calls **database/seeds/DatabaseSeeder.php**. That file has a class that extends the Seeder class. 
In there is a run function that calls a seeder for generating Signatures:

```angular2html
public function run()
{
    $this->call(SignaturesTableSeeder::class);
}
``` 

Generate that seeder by running `php artisan make:seeder SignaturesTableSeeder` and include a call to the model factory that's outlined in the [scotch tutorial](https://scotch.io/tutorials/build-a-guestbook-with-laravel-and-vuejs).

```angular2html
public function run()
{
    factory(App\Signature::class, 100)->create();
}
```

To check and make sure it worked we could view the db in a GUI such as Sequel Pro or use tinker like:

```angular2html
php artisan tinker
> \DB::table('signatures')->count(); // view # of records
> \DB::table('signatures')->get(); // view all records
```

### Routes and Controllers

In this section we set up controllers for reporting signatures, displaying them, adding them and displaying an individual resource. We're building this as an API so I thought it was cool
that the author namespaced the controllers into an Api folder when creating them. That was as simple as `php artisan make:controller Api/SignatureController`.
Additionally, we use a Laravel [resource controller](https://laravel.com/docs/5.5/controllers#resource-controllers) but specify that we do not want ALL the routes, only a few:

```angular2html
Route::resource('signatures', 'Api\SignatureController')->only(['index', 'store', 'show']);
```

Most interestingly, in our controller we make calls to and return instances of `SignatureResource`, which we haven't defined yet.

There's also a nifty call to get the latest Signatures and ignore all signatures that have been flagged:

```angular2html
$signatures = Signature::latest()
            ->ignoreFlagged()
            ->paginate(20);
```

The `ignoreFlagged` method call is a custom query scope that we define with the convention of "scopeMethodNameInCamelCase". We defined the custom query scope in the Signature model:

```angular2html
public function scopeIgnoreFlagged($query)
{
    return $query->where('flagged_at', null);
}
```

The calls to latest() and paginate() are default parts of the [Eloquent ORM](https://laravel.com/docs/5.5/eloquent).

If you're new to Laravel it is also worth checking out the [route model binding](https://laravel.com/docs/5.5/routing#route-model-binding) included in the reporting signature route and controller.

The route looks like:

```angular2html
Route::put('signatures/{signature}/report', 'Api\ReportSignature@update');
```

Including the signature like this actually gives us access to the individual signature in our controller. So in the update method we pass in the database instance of the signature that we want to flag:

```angular2html
public function update(Signature $signature)
{
    $signature->flag();

    return $signature;
}
```

Flag is a custom method we define in the Signature model.

### The Signature Resource 

Eloquent Resources ([docs](https://laravel.com/docs/5.5/eloquent-resources)) are new in Laravel 5.5. Resources are a middle layer between your database and the JSON responses you send out to end users.
In our case we're going to modify the data so we don't send out people's email addresses and send an avatar for their image.

> Jeffery Way posted a Laracast about API Resources: [What's New in Laravel 5.5: API Resources](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/20)

In order to define a custom `avatar` attribute for signature instances we use the synatax "getAttributNameAttribute" syntax for out function:

```angular2html
public function getAvatarAttribute()
{
    return sprintf('https://www.gravatar.com/avatar/%s?s=100', md5($this->email));
}
```

`sprintf` returns a formatted string in PHP and `md5` creates a hash. Gravatar is a service that grabs a photo for the user based on their email address.