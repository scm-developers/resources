# Laravel Pusher

## Preview

![image](https://user-images.githubusercontent.com/59432845/171817167-1a9eceec-6335-47fd-9fe3-77be8923e0df.png)

## Table of Contents
  - [Create Pusher](#create-pusher)
  - [What is Pusher ?](#what-is-pusher-)
  - [Pusher Server Side Installation](#pusher-server-side-installation)
  - [Pusher Client Side Installation](#pusher-client-side-installation)
  - [Create Model](#create-model)
  - [Create Migration](#create-migration)
  - [Create Data By Running Seeder](#create-data-by-running-seeder)
  - [Create Event](#create-event)
  - [Create Route](#create-route)
  - [Create Controller](#create-controller)
  - [Create View](#create-view)

### Create Pusher
#### What is Pusher ?
[Pusher](https://pusher.com) is hosted API service to have Real Time Experience such as (Chat, Notification, etc.) without needing to refresh your website. You can click [here](./Create_Pusher_Account.md) to know more details and create free account.

### Pusher Server Side Installation
In **cmd**,
```
composer require pusher/pusher-php-server
```
In **env** file,
default *BROADCAST_DRIVER* is *log*.
Modify it to ...
```
BROADCAST_DRIVER=pusher
```
And add Pusher App keys in **env** file.<br>
_(If you don't have Pusher App keys, Creat Account in [Pusher](https://pusher.com).<br>[Click Here](./PusherSignUpManual/README.md) to see how to create Free Account in [Pusher](https://pusher.com) to use)_
```
PUSHER_APP_ID=your-pusher-app-id
PUSHER_APP_KEY=your-pusher-key
PUSHER_APP_SECRET=your-pusher-secret
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```
Make sure _BroadcastServiceProvider_ is uncommented in **app.php**

```php
App\Providers\BroadcastServiceProvider::class,
```
### Pusher Client Side Installation
In **cmd**,
```
npm install --save-dev laravel-echo pusher-js
```
**app.js**
```js
import Echo from 'laravel-echo';
 
window.Pusher = require('pusher-js');
 
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

### Create Model
After Installation, Let's create **Vote Model**.
```
php artisan make:model Vote
```

### Create Migration
After that, Create **Migration** for **Vote Model**.
```
php artisan make:migration create_votes_table
```

And add count column in migrations. 

My migration file will be looked like this ...
```php
class CreateVotesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('votes', function (Blueprint $table) {
            $table->id();
            $table->integer('count')->default(0);
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('votes');
    }
}
```

### Create Data By Running Seeder
In this example, I will add seeder for Vote Model.
In my **DatabaseSeeder.php** ...
```php
class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $vote = new Vote();
        $vote->count = 1;
        $vote->save();
    }
}

```

### Create Event
And, Let's create **Event** for **Vote**.
```
php artisan make:event VoteEvent
```
My Event file will be look like this...
_(Note: Be sure to implement **ShouldBroadcast**)_
```php
class VoteEvent implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new Channel('vote-channel');
    }
}
```
In `broadcastOn()` function, we define new channel named **vote-channel**.

### Create Route
In **web.php**, we added route for vote.
```php
<?php

use App\Http\Controllers\VoteController;
use Illuminate\Support\Facades\Route;

Route::get('/', [VoteController::class, 'index']);
Route::post('/', [VoteController::class, 'update']);

```

### Create Controller
If you haven't created Controller yet ...
```
php artisan make:controller VoteController
```
In my controller, <br>
index() function is to show Vote.<br>
update() function is to increase count of vote.
```php
class VoteController extends Controller
{
    public function index()
    {
        $vote = Vote::first();
        return view('vote', compact('vote'));
    }

    public function update()
    {
        $vote = Vote::first();
        $vote->count = $vote->count + 1;
        $vote->save();

        broadcast(new VoteEvent());

        return redirect('/');
    }
}
```

### Create View
After that, in my blade file ...
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Vote</title>
  <script src="{{ mix('js/app.js') }}"></script>
</head>

<body>
  <h1>Vote result: {{ $vote->count }}</h1>
  <form method="post">
    @csrf
    <button type="submit">Vote</button>
  </form>

  <script>
    Echo.channel('vote-channel').listen("VoteEvent", (e) => {
     location.reload();
    });
  </script>
</body>

</html>
```
_(Note: **vote-channel** is the channel we named on event and **VoteEvent** is the event we created)_

and Finally, 
```
npm install
npm run dev
```
… and run our server:
```
php artisan serve
```
And Result will be displayed in [http://127.0.0.1:8000](http://127.0.0.1:8000)