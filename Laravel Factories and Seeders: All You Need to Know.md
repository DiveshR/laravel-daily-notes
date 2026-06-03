
Laravel Factories and Seeders: All You Need to Know
April 10, 2025
26 min read
Seeding data in Laravel is quite simple, but also has a lot of caveats, less-known functions and use-cases. Both Seeders and Factories have so much "hidden" power that we've compiled this HUGE tutorial, with screenshots from real scenarios.

You can treat it as "Laravel docs on steroids": same functions explained in a more practical visual way, with a better flow to read step-by-step.

Among other topics, in this tutorial, we will answer questions, like:

How do you create model factories with complex relationships?
What's the best way to test with large datasets?
How can you make one factory field depend on another?
Which factory methods are powerful but rarely used?
How do you make test data look realistic?
So, let's dive in!

Table of Contents
1. Seeding Approaches

Using the DatabaseSeeder Class
Using Separate Seeder Files
Running Individual Seeders
2. Understanding Laravel Factories

Creating a Factory
Using Factories in Seeders
Other Factory Use Cases
3. Factory Methods and Features

Factory States
Persistent vs Non-Persistent Factories
Overriding Factory Values
Sequence Factories
Lifecycle Hooks: afterMaking and afterCreating
4. Working with Relationships

BelongsTo Relationships
HasMany Relationships
ManyToMany Relationships
Polymorphic Relationships
Reusing Models with recycle()
5. Tips and Best Practices

Accessing Other Attributes
Seeding Unique Values
Lesser-Known Factory Methods
Matching Real-World Data
Testing with Large Datasets
1. Seeding Approaches
In Laravel, we have a couple of ways to seed our Database:

Using the DatabaseSeeder class
Using separate files for each Seeder and calling them from the DatabaseSeeder class
Let's explore each approach.

Using the DatabaseSeeder Class
Laravel provides a DatabaseSeeder class by default:

database/seeders/DatabaseSeeder.php

// ...
 
class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        // User::factory(10)->create();
 
        User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
    }
}

Which already contains a seed for Test User. But what if we want to add more things? Let's take our Currency example:

database/seeders/DatabaseSeeder.php

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        User::factory(10)->create();
 
        User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
 
        $currencies = [
            [
                'name' => 'US Dollar',
                'code' => 'USD',
            ],
            [
                'name' => 'Euro',
                'code' => 'EUR',
            ],
            [
                'name' => 'British Pound',
                'code' => 'GBP',
            ],
        ];
 
        foreach ($currencies as $currency) {
            Currency::create($currency);
        }
    }
}

Let's try to run the Seeder:

php artisan db:seed
# Or (if you want to refresh the database)
php artisan migrate:fresh --seed



This screen shows that our Seeder ran successfully. But that's all the information we get. We don't know how long it took to run, which can lead us to think that the Seeder is stuck. This is because the Seeder is running synchronously, and we have no way to know how long it will take.

Of course, this is not the only problem. When the system grows - and it will - the DatabaseSeeder class will become a mess. It will be hard to maintain and understand. We should consider using separate files for each Seeder.

Using Separate Seeder Files
Let's take a look at separate files for each Seeder. In this case, we will move the Currency Seeder and User Seeder to separate files:

php artisan make:seeder CurrencySeeder
php artisan make:seeder UserSeeder

Let's fill the CurrencySeeder with the Currency data:

database/seeders/CurrencySeeder.php

class CurrencySeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $currencies = [
            [
                'name' => 'US Dollar',
                'code' => 'USD',
            ],
            [
                'name' => 'Euro',
                'code' => 'EUR',
            ],
            [
                'name' => 'British Pound',
                'code' => 'GBP',
            ],
        ];
 
        foreach ($currencies as $currency) {
            Currency::create($currency);
        }
    }
}

And the UserSeeder with the User data:

database/seeders/UserSeeder.php

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        User::factory(10)->create();
    }
}

Finally, let's call these Seeders from the DatabaseSeeder class:

database/seeders/DatabaseSeeder.php

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
 
        $this->call([
            CurrencySeeder::class,
            UserSeeder::class
        ]);
    }
}

Now, let's run the Seeder:

php artisan db:seed
# Or (if you want to refresh the database)
php artisan migrate:fresh --seed



As you can see, we have a detailed report of which Seeders ran and how long they took. This is a much better approach than using the DatabaseSeeder class.

Running Individual Seeders
Another advantage of using separate files is that we can run them individually. This can be useful when we want to run only one Seeder:

php artisan db:seed --class=UserSeeder

This will run only the UserSeeder and not the CurrencySeeder:



This is a great way to test and debug Seeders or in testing environments where we need to run only a specific Seeder.

2. Understanding Laravel Factories
Let's talk about Factories in Laravel. Factories are great if we want to generate fake data for our Database:

Factory Example

public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'password' => static::$password ??= Hash::make('password'),
        'remember_token' => Str::random(10),
    ];
}

This Factory will generate a User with a random name and a unique email. But how does it work?

Creating a Factory
Let's look at how Factories are Created. For this, we need a model:

php artisan make:model Product -m

With these fields:

Migration

Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description');
    $table->decimal('price', 8, 2);
    $table->integer('stock');
    $table->timestamps();
});

And, of course, we need a Model completed:

app/Models/Product.php

class Product extends Model
{
    protected $fillable = [
        'name',
        'description',
        'price',
        'stock',
    ];
 
    protected function casts(): array
    {
        return [
            'price' => 'decimal',
            'stock' => 'integer',
        ];
    }
}

Now that we have our Model in place, we must create a Factory. There are 2 steps we have to take:

Create the Factory
Add the HasFactory trait to the Model
Let's create the Factory:

php artisan make:factory ProductFactory --model=Product

This will create an empty Factory for our Product Model:

database/factories/ProductFactory.php

class ProductFactory extends Factory
{
    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            //
        ];
    }
}

And here's where we define the default state of our Model. In this case, we want:

name to be two random words
description to be a random paragraph
price to be a random number between 9.99 and 99.99
stock to be a random number between 1 and 100
We will use the fake() helper from the Faker library to do this. This is a helper function that will generate random data for us:

Note: fake() helper only exists in local env. Using Faker in production is not recommended.

Note 2: You might see $this->faker in some examples. This is the same as fake().

database/factories/ProductFactory.php

// ...
public function definition(): array
{
    return [
        'name' => fake()->words(2, true),
        'price' => fake()->randomFloat(2, 9.99, 99.99),
        'description' => fake()->paragraph(),
        'stock' => fake()->numberBetween(1, 100),
    ];
}
// ...

Now, there's a few things to note here:

Generating words() accepts two parameters - word count and if you want to return as a string or an array (default is false - array)
randomFloat() accepts 3 parameters - decimal places, min value, and max value
numberBetween() accepts 2 parameters - min value and max value
Our Factory is ready to be used, but we still need to add the HasFactory trait to our Model:

app/Models/Product.php

use Illuminate\Database\Eloquent\Factories\HasFactory;
 
// ...
 
class Product extends Model
{
    use HasFactory;
    // ...
}

Using Factories in Seeders
Now we are fully set up to use our Factory in Database Seeder:

database/seeders/DatabaseSeeder.php

use App\Models\Product;
 
// ...
 
public function run(): void
{
    Product::factory()->create();
 
    // ...
}

And finally, let's run the Seeder:

`php artisan migrate:fresh --seed`

This will result in our Products table having a record:



But that's just one record created - which might not be enough. We can, however, create multiple records in a few ways:

database/seeders/DatabaseSeeder.php

public function run(): void
{
    Product::factory()->create();// One Record 
    Product::factory(10)->create();// 10 Records 
    Product::factory()->count(10)->create();// 10 Records 
 
    // ...
}

Now, let's pick one of the ways to create 10 records and run it:

Note: I like the ->count() as it's more readable and easier to understand.



That's it! Creating simple Factories is easy and can be done in a few steps.

Other Factory Use Cases
However, factories are not just for seeding the database in Seeders. They can also be used in:

Tests - to generate fake data for testing
Tinker - to generate more data for manual testing
More on these in the following lessons.

3. Factory Methods and Features
Now let's explore some of the powerful features of Laravel factories.

Factory States
We can create essential records already, but what if we need to create a record with a specific state? For example, a user that is an admin? Or a user who has not yet filled in their profile?

For this, we can use Factory States, which allows us to define additional states for our factories.

Let's look at an example that comes with Laravel:

database/factories/UserFactory.php

// ...
class UserFactory extends Factory
{
    // ...
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }
 
    /**
     * Indicate that the model's email address should be unverified.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}

In this example, we have an unverified method that sets the email_verified_at attribute to null. We can use this state like this:

User::factory()->unverified()->create();

That's how simple it is to use factory states. You can define as many states as you need for your factories. And, of course, you can override as many fields as you need here.

Persistent vs Non-Persistent Factories
With Factories, you can create models in two ways: persistent and non-persistent. But what's the difference?

Persistent Factories
When discussing Persistence, we often mean that the model is saved to the database. This is the default behavior of the create method.

User::factory()->create();

This will write a new random user to our database:





In other words, the create method will create a new instance of the model and save it to the database just like we would do with:

User::create(['name' => 'John Doe']);

Non-Persistent Factories
Non-persistent - means that the model is not saved to the database. This is the default behavior of the make method.

User::factory()->make();

This will create a new random user, but it will not be saved to the database:



Note: Look at the id field. It is not here, and checking the database will show that no new user was created:



Here are a few great use cases for the make method:

Use-case 1. Testing.

It can be used in testing to fake API/Post calls to your endpoints:

Controller

class ProductStoreController extends Controller
{
    public function __invoke(StoreProductRequest $request)
    {
        return Product::create($request->validated());
    }
}

Request

public function rules(): array
{
    return [
        'name' => ['required'],
        'description' => ['required'],
        'price' => ['required', 'numeric'],
        'stock' => ['required', 'integer'],
    ];
}

Then, we can use the make method to create a new product in our test:

Test

// Typical Way
$response = $this->post('product', [
    'name' => 'Test Product',
    'price' => 100.00,
    'description' => 'This is a test product.',
    'stock' => 10,
]);
 
// Using Factory Make
$response = $this->post('product', Product::factory()->make()->toArray());

Note: The toArray() method is used to convert the model instance to an array, which is what the request expects.

Running the test for both cases will yield the same result:



It's a nice way to create a new model instance without manually typing all the fields.

Use-case 2. Generating Sample Data.

With the make method, we can generate bigger sample datasets unsaved to the database. For example, if we want to generate a CSV file for import testing:

class CreateFakecsvCommand extends Command
{
    protected $signature = 'create:fakecsv';
 
    protected $description = 'Command description';
 
    public function handle(): void
    {
        $users = User::factory(1000)->make();
        $csvFile = fopen('fake_users.csv', 'w');
        fputcsv($csvFile, ['name', 'email']);
        $this->withProgressBar($users->count(), function (ProgressBar $bar) use ($csvFile, $users) {
            foreach ($users as $user) {
                fputcsv($csvFile, [
                    $user->name,
                    $user->email,
                ]);
                $bar->advance();
            }
        });
        fclose($csvFile);
 
        $this->info('CSV file created successfully.');
    }
}

This will give us a CSV file with 1000 random users, and we can use this to test our import functionality:



Overriding Factory Values
Overriding the default attributes is pretty simple:

User::factory()->create(['name' => 'John Doe']);
User::factory()->make(['name' => 'John Doe']);

This will create a user with the name John Doe instead of the random name generated by the Factory. And this can be great if you want a specific value for a specific model.

Overriding State Values
Let's look at states and how we can override values inside them. We can do this by using the state method and passing an array of attributes to override.

First, let's look at the state we have:

State

public function admin(): static
{
    return $this->state(fn(array $attributes) => [
        'role_id' => 1
    ]);
}

Usage

Role::factory()->create(['name' => 'admin']);
 
$users = User::factory()
    ->admin()
    ->create();

This is fine, but what if we want to give our users a different role? We would have to create another state for that. But we can override the role_id attribute by using the state method again:

State

public function role(int $roleID): static
{
    return $this->state(fn(array $attributes) => [
        'role_id' => $roleID
    ]);
}

Usage

$admin = Role::factory()->create(['name' => 'admin']);
 
$users = User::factory()
    ->admin()
    ->role($admin->id)
    ->create();
 
$user = Role::factory()->create(['name' => 'user']);
 
$users = User::factory()
    ->admin()
    ->role($user->id)
    ->create();

Now, if we look at our database, we can see that the role_id is different for both users:



This is great if we want to create re-usable states that can be used in different scenarios.

Sequence Factories
While overriding one value is great for one use, what would happen if we needed to create a few? It would become repetitive! That's why a sequence method allows us to create a sequence of models with different attributes.

$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();

This will create two users with our specific names:



And, of course, this can be applied to any attribute. But what if we want a relationship like a Role? It can be done like this:

$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User', 'role_id' => Role::factory()->create(['name' => 'Admin'])],
        ['name' => 'Second User', 'role_id' => Role::factory()->create(['name' => 'User'])],
    )
    ->create();



Lastly, we can also use a callback to generate the values. This is useful when we want to generate a random value for each model:

use Illuminate\Database\Eloquent\Factories\Sequence;
 
// ...
 
$users = User::factory()
    ->count(10)
    ->sequence(fn (Sequence $sequence) => ['role_id' => Role::factory()->create(['name' => 'Role ' . $sequence->index])])
    ->create();

And it will generate:



It's great for generating a sequence of models with different attributes and/or relationships.

Lifecycle Hooks: afterCreating() and afterMaking()
Of course, factories are not just limited to creating models. We can also take some actions after they were created.

In this example, we want to create a Product, and once it is created - we want to create a few variations of the product and a few reviews for it.

Product Factory

class ProductFactory extends Factory
{
    public function definition()
    {
        return [
            'name' => fake()->productName(),
            'description' => fake()->paragraph(),
            'price' => fake()->randomFloat(2, 5, 500),
            'stock' => fake()->numberBetween(0, 100),
            'status' => 'active',
        ];
    }
 
    public function configure()
    {
        return $this->afterCreating(function (Product $product) {
            // Create 3 product variations (only for test data)
            $product->variations()->createMany([
                [
                    'size' => 'Small',
                    'color' => 'Red',
                    'sku' => "SKU-{$product->id}-S-RED",
                    'additional_price' => 0,
                ],
                [
                    'size' => 'Medium',
                    'color' => 'Blue',
                    'sku' => "SKU-{$product->id}-M-BLUE",
                    'additional_price' => 5.99,
                ],
                [
                    'size' => 'Large',
                    'color' => 'Black',
                    'sku' => "SKU-{$product->id}-L-BLACK",
                    'additional_price' => 10.99,
                ]
            ]);
 
            // Create sample reviews (only for testing)
            Review::factory()
                ->count(5)
                ->create([
                    'product_id' => $product->id,
                    'verified_purchase' => true,
                ]);
        });
    }
}

Now, running this command will automatically create variations and reviews:

Product::factory()->create();

You may say that it's also possible with Eloquent Observers? And yes, but the purpose is different. Factories are mostly used for testing purposes, so you would want to trigger additional data only when using Factories, and not globally for that Eloquent model.

Using afterMaking
We can also use the afterMaking method to do something after the model is made (but not saved to the database). This is useful if we want to do something with the model before it is saved.

Product Factory

// ...
 
public function configure()
{
    return $this->afterMaking(function (Product $product) {
        info("Product made: " . $product->name);
    });
}

Usage

Product::factory()->make();

And if we open the laravel.log file, we should see:

[2025-04-01 10:26:16] local.INFO: Product made: omnis illum

4. Working with Relationships
Factories are often used to create models, but do you know how to create various related models? In this section, we will cover:

belongsTo
hasMany
manyToMany with pivot table attributes
Polymorphic Relationships
Reusing Models for different factories and their relationships
Let's look at various relationships in factories.

BelongsTo Relationships
Our first example is the most common one - one Model belongs to another. In this case, our User has a role() relationship:

app/Models/User.php

class User extends Authenticatable
{
    // ...
 
    public function role(): BelongsTo
    {
        return $this->belongsTo(Role::class);
    }
}

Now, in our Factory, we need to define the role_id column:

database/factories/UserFactory.php

use App\Models\Role;
 
// ...
 
public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'email_verified_at' => now(),
        'password' => static::$password ??= Hash::make('password'),
        'remember_token' => Str::random(10),
        'role_id' => Role::factory()
    ];
}

Now, whenever we call User::factory()->create() - we will also create a Role model. This is a one-to-one relationship, so we can use the create() method directly.

HasMany Relationships
We have a Post model with many Comment models in this case. The relationship is defined in the Post model:

app/Models/Post.php

class Post extends Model
{
    // ...
 
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}

There are a couple of ways we can do this:

We can create the Comment relationship in the afterCreating method of the Post factory.
We can use ->has(...) helper method in the Post factory.
Option 1: Using afterCreating
This option is probably the most common one:

database/factories/PostFactory.php

use App\Models\Comment;
 
// ...
 
public function definition(): array
{
    return [
        'title' => fake()->sentence(),
        'body' => fake()->paragraph(),
        'user_id' => User::factory()
    ];
}
 
public function configure(): static
{
    return $this->afterCreating(function (Post $post) {
        Comment::factory(3)->create([
            'post_id' => $post->id
        ]);
    });
}

Now we can run:

Post::factory(2)->create();

And we should see that 2 posts are created, and each post has 3 comments:



But this is not ideal since it will always create 3 comments for each post - no matter what.

Option 2: Using ->has(...)
A better way to do this is to use the ->has(...) method in the Post factory:

database/factories/PostFactory.php

class PostFactory extends Factory
{
    protected $model = Post::class;
 
    public function definition(): array
    {
        return [
            'title' => fake()->sentence(),
            'body' => fake()->paragraph(),
            'user_id' => User::factory()
        ];
    }
}

And then all we have to do is call the Factory with the ->has(...) method:

Post::factory(3)
    ->has(Comment::factory()->count(5))
    ->create();

Now, we will get 3 posts, and each post will have 5 comments:



This is a much better way to do this since we can easily change the number of comments for each post or even create a post without any comments.

ManyToMany Relationships
When dealing with many-to-many relationships, there are a few ways to do this:

Using the attach() method in the afterCreating Factory method.
Using the ->hasAttached(...) method in the factory.
Using the ->has(...) method in the factory.
So let's look at each of them:

Option 1: Using attach()
The first example can be using the attach() method in the afterCreating method of the Factory:

database/factories/PostFactory.php

use App\Models\Tag;
// ...
 
public function configure(): static
{
    return $this->afterCreating(function (Post $post) {
        $post->tags()->attach(Tag::factory(3)->create());
    });
}

But again, this is not ideal since it will always create 3 tags for each post - no matter what.

Option 2: Using ->hasAttached(...)
A better way to do this is to use the ->hasAttached(...) method when calling the Factory:

Post::factory(3)
    ->hasAttached(Tag::factory()->count(5))
    ->create();

This will create 3 posts, and each post will have 5 tags:



Much better! But there are more options.

Option 3: Using ->has(...)
We can also use the ->has(...) method in the Post factory:

Post::factory(2)
    ->has(Tag::factory()->count(2))
    ->create();

And Laravel will try to auto-guess the relationship name. But if it is custom, you can specify it like this:

Post::factory(2)
    ->has(Tag::factory()->count(2), 'tagsRelationshipName')
    ->create();



Dealing With Pivot Table Attributes
If you want to add attributes to the pivot table, you can do it like this:

Note: You can only use this method OR the ->afterCreating(...) method. Others don't support pivot values.

Post::factory(2)
    ->hasAttached(Tag::factory()->count(2), [
        'priority' => true,
    ])
    ->create();

This will now create 2 posts, and each post will have 2 tags with the priority column set to true:



And if you want to create just one Tag with priority set to true, you can do it like this:

Post::factory(2)
    ->hasAttached(Tag::factory()->create(), [
        'priority' => true,
    ])
    ->hasAttached(Tag::factory()->count(2))
    ->create();

This will create one Tag with priority set to true, and 2 Tags without it:



With these methods - you can easily create complex relationships with pivot table attributes.

Polymorphic Relationships
With polymorphic relationships, we can use most of the methods we have used before. So things like:

->has(...)
->hasAttached(...)
->for(...)
It should work as you would expect. The only new thing here might be the definition itself since it allows this:

Factory

public function definition(): array
{
    return [
        'related_id' => User::factory(),
        'related_type' => function (array $attributes) {
            return User::find($attributes['user_id'])->type;
        },
        'path' => fake()->url(),
        'type' => 'jpg',
        'size' => fake()->numberBetween(100, 1000),
    ];
}

It simply allows the retrieval of the "just created" record and uses it to set the related_type column.

Reusing Models with recycle()
When creating many models, we might encounter an issue where we create a large number of records for a related table but only need a few.

A quick example of this is when creating many Post models, and each post has a User model. If we have already seeded 10 users, why not use them instead of creating new ones?

So let's look at how to do this:

$users = User::factory()->count(10)->create();
 
Post::factory(50)
    ->recycle($users) // Or any other related Model
    ->create();

This will create 50 posts, but it will use the existing users in a random order:



If we were to skip the ->recycle(...) call - we would get 50 new users:



In this case, it's better to recycle our Models, as that prevents database pollution and makes them easier to test.

5. Tips and Best Practices
And finally, here's a few tips and tricks that can help you get the most out of Laravel Factories and Seeders:

Accessing other attributes
Seeding unique values
Less used/known factory methods
Always try to match your factories with real data
Use factories/seeds to test big loads in the system
Let's go through each of these tips in detail.

Accessing Other Attributes
Sometimes, you must set a field to a specific value based on another field. For example, if a status is set to active, then we want to put the activated_at field to the current date:

public function definition(): array
{
    return [
        'active' => fake()->boolean(),
        'activated_at' => function (array $attributes) {
            return $attributes['active'] ? now() : null;
        },
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}

This allows us to create conditional field values based on other attributes in the model.

Seeding Unique Values
While getting random values is great, sometimes you need to ensure that the values are unique. For example, if we have a Status model with fixed values, we can use the unique() method to ensure that the values are unique:

public function definition(): array
{
    return [
        'name' => fake()->unique()->word(),
        'description' => fake()->sentence(),
        'status' => fake()->unique()->randomElement(Status::pluck('name')->toArray()),
    ];
}

This will automatically track values used in the current seeder and ensure they are unique for each status name.

Lesser-Known Factory Methods
Faker comes with many methods, but we often use only a few. So let's look at some of the rarer ones:

fake()->boolean($chanceOfGettingTrue = 50) - Returns a random boolean value with a chance of getting true. You can increase the chance of getting true by passing a number between 0 and 100.
fake()->firstNameMale() - Generates a random Male first name
fake()->firstNameFemale() - Generates a random Female first name
fake()->words($count = 5, $asText = false) - Generates an array of random words. If you set $asText to true, it will return a string with the words separated by spaces.
fake()->sentence($nbWords = 6, $variableNbWords = true) - Generates a random sentence with several words. If you set $variableNbWords to true, it will generate a random number of words between 1 and the number you pass.
fake()->paragraph($nbSentences = 3, $variableNbSentences = true) - Generates a random paragraph with several sentences. If you set $variableNbSentences to true, it will generate a random number of sentences between 1 and the number you pass.
fake()->paragraphs($nb = 3, $asText = false) - Generates an array of random paragraphs. If you set $asText to true, it will return a string with the paragraphs separated by new lines.
fake()->text($maxNbChars = 200) - Generates a random text with a maximum number of characters. This is useful for generating long texts.
fake()->realText($maxNbChars = 200) - Generates a random text with a maximum number of characters. This is useful for generating long texts that look like real text.
fake()->dateTimeThisMonth() - Generates a random date and time in the current month. You can also pass a boolean to get a date in the past or future.
fake()->streetName() - Generates a random street name. This is useful for generating addresses.
And many more. The complete list of methods is in the Faker documentation.

Matching Real-World Data
Next, let's talk about our data. Generating Fake information is okay for our tests. Still, it doesn't look great if we want to use it in demos or local manual testing.

So, our advice here would be to always try to match your factories with real-world data as closely as possible. Here are a few examples:

Instead of ->word() for titles, use ->sentence() or ->realText()
Instead of ->sentence() for content, use ->realText()
Instead of ->float() for prices, use ->randomFloat(2, 1, 1000) to get a price between 1 and 1000 (to avoid insane prices)
Instead of ->words(3, true) for address, use ->streetName() or ->address()
In general, pick the closest method to the real data you have in your production instance. This will give you a few benefits:

More realistic data for visualization
Easier to spot bugs in your code - when things look weird, you can spot them easier
Your test validation might be more strict - for example, if you have a validation rule that checks if the title is a sentence, then using ->word() will fail the validation. This is especially useful when you have a lot of data and want to ensure everything is valid.
And, of course, it looks better and can help you notice design problems in your application.

Testing with Large Datasets
While Factories and Seeders are great for generating data, they can also be used to test an application's performance.

We often ignore the performance with bigger data sets until it's too late. That's mainly because we only create enough data to test the functionality of our application. But that does not show the whole picture since you can have issues like:

Slow queries
N+1 queries
Slow foreach loops
Loading too much data from the database
and much more...
But that's where factories and seeders can help you. You can use them to generate a lot of data and test the performance of your application. For example - instead of creating 10 users, create 1000 and see how your application performs. Do the same for all the tables in your application.

Do you know any tips/tricks or practical Factory methods? Is there a function that you use all the time? If so, please share it with us in the comments below. We would love to hear from you!

Recent Courses on Laravel Daily
Next.js Basics for Laravel Developers
11 lessons
58 min
Apr 2026
Testing in Laravel 13 For Beginners
26 lessons
1 h 41 min read
Apr 2026
How to Structure Laravel 13 Projects
16 lessons
1 h 32 min read
Mar 2026
Get notified when participating
jj15 avatar
jj15
1 year ago
Thank you for this wonderful and detailed article! One factory method I discovered recently is forEachSequence() – it lets you pass in arrays of sequenced attributes and creates exactly that many models, eliminating the need to specify the count() method.

Post::factory()
    //->count(2) <- No need for this!
    ->forEachSequence(
        // Will create exactly two models with the below attributes:
        ['status' => 'published'],
        ['status' => 'draft'],
    )
    ->create();

👍 6

Modestas avatar
Modestas
1 year ago
Thank you for the kind words and another tip!

Will update the article with this change soon


Divesh avatar
Leave a reply
j.kludt avatar
j.kludt
1 year ago
useful course thanks


Divesh avatar
Leave a reply
kalDeveloper avatar
kalDeveloper
1 year ago
Another way I used to generate data for one to many and many to many relationships. my main purpose is to generate random records rather stick with constant value.

User::factory(10)->create()
  ->each(function ($user) {
     Post::factory(rand(1, 10))->create(['user_id' => $user->id]);
    });
 
$tags = Tag::factory(10)->create();
Post::all()->each(function ($post) use ($tags) {
     $post->tags()->attach($tags->random(rand(1, 10)),['is_published' => rand(0,1)]);
});

Someone might have a different opinion on this. I would like to hear those. Greate article by the way!!

👍 2

Povilas Korop avatar
Povilas Korop
1 year ago
Yeah, I think it's also a valid option. Thanks for the comment, Kal!


Divesh avatar
Leave a reply
Mohan avatar
Mohan
1 year ago
So I have a question. I have two tables that are large. One has 44K rows and other approximately 8K rows. If I use RefreshDatabase then simply reloading these two tables takes time and I need them to do adquate testing.

Do you have any thoughts or strategies to deal with large table issues and RefreshDatabase?

Currently I simply do not use RefreshDatabase and I truncate the tables I need to in the before and after events.


Modestas avatar
Modestas
1 year ago
In your case, I would not use RefreshDatabase but instead I would use DatabaseTransactions. That way records are automatically cleared after the test is done, but you can still manually seed the initial data and keep it there. Of course, this won't work on the CI.

To have this working on the CI, I would do a setup action for your tests where it would import an SQL file. Yes, a static test only file. That way, you'd spend less time on fake data :)


Mohan avatar
Mohan
1 year ago
Thank you for the suggestion, I will look into it.


Divesh avatar
Leave a reply
Mohan avatar
Mohan
1 year ago
I know for Nightwatch the team generated sample data for each hour of the day and more data in business hours vs. non business hours by timezone for their testing. I would be interested in a detailed understanding of how they did it efficiently. Just a suggestion for future directions for this excellent subject matter.


Divesh avatar
Leave a reply
Divesh avatar
Leave a comment
​
You can use Markdown
Comment
Laravel Daily
Laravel Daily
Your trusted source for modern Laravel courses, tutorials, and insights. Learn from real-world examples and level up your development skills.

Learn
Premium Courses
Roadmap
Project Examples
Company
About
Newsletter
Pricing
© 2026 Laravel Daily. All rights reserved.

For any questions - email info@laraveldaily.com


Feedback
