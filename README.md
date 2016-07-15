# The Task

To create a multi-tenant Laravel installation where Organisations can sign up and add there own domain name, kind of like one of those famous website builder sites. This would involve solving the below problems..

  - Add new domains to the Apache Vhost file on the VPS to enable resolution 
  - Once domain resolves to our /public Laravel folder, check the organisation exsists.
  - If orgranisation exsists, load there views folder (theme).
  - Ensure that all input/output is done via the organisation.
  - Enable content management for this organisation via 'widgets' (basic).

### Technology

Bos uses a number of open source projects to work properly:

* [Laravel5.2] - The best PHP MVC framework known to man.
* [Ace Editor] - awesome web-based text editor
* [markdown-it] - Markdown parser done right. Fast and easy to extend.
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [Gulp] - the streaming build system

### The Plan

After installing Laravel, create the organisations table migration.

```sh
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;

class CreateOrganisationsTable extends Migration
{
    /**
     * Run the migrations.
     * @return void
     */
    public function up()
    {

        Schema::create('organisations', function(Blueprint $table) {

            $table->increments('id');
            $table->string('name');
            $table->string('domain');
            $table->string('theme_folder');

            $table->timestamps();

        });
        
    }

    /**
     * Reverse the migrations.
     * @return void
     */
    public function down()
    {
        Schema::drop('organisations');
    }

}

```

Create the Organisation model

```sh
Terminal: php artisan make:model Organisation
```

Create the Themes folder where each sites views will be loaded from. In my example i've created it at '/resources/views/organisations/'

Next, we need to elevate the Organisation into the Session so we can use it anywhere in the Application.

```sh
if (isset($_SERVER['REQUEST_METHOD'])) {

    $organisation = Organisation::where('domain', '=', $_SERVER['SERVER_NAME'])->first();

    if (count($domain) > 0) {
        Session::set('organisation_id', $organisation->id);
    } else {
        abort(404);
    }
}
```

5. Once we have the Organisation available in the Session, we need to use it efficiently, so we'll add it to the main Controllers constructor. 'App/Http/Controllers/Controller.php'

```sh
<?php

namespace App\Http\Controllers;

use App\Organisation;
use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Routing\Controller as BaseController;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Support\Facades\Session;

abstract class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;

    public function __construct()
    {
        $this->organisation = Organisation::findOrFail(Session::get('organisation_id'));
    }
}
```

Now we have the current Organisation's model available accross all controllers, so we can do cool things like;

```sh
/**
 * Display a listing of the resource.
 * @return Response
 */
public function index()
{
    $pinapples = $this->organisation->pinapples;

    return view('organisations.' . $this->organisation->theme_folder . '.public.pinapples.index', compact('pinapples'));
}
```
