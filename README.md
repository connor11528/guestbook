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

