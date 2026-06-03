6 Bad Practices When Building Laravel APIs


Laravel allows us to structure code in many ways, right? But with APIs, it's important to avoid some bad practices, cause it may break API clients and confuse other developers.
**1. Returning Status Code 200 When Errors Occur**
One of the most common mistakes is returning a 200 OK status code when something has **actually** gone wrong.
**Bad Practice:**

`public function store(Request $request){    try {        if (!$request->has('name')) {            return response()->json([                'success' => false,                'error_message' => 'Name is required'            ], 200); // WRONG! Using 200 for an error        }         // ...    } catch (\Exception $e) {        return response()->json([            'success' => false,            'error_message' => 'Something went wrong'        ], 200); // WRONG again!    }}`
**Better Approach:**

`public function store(Request $request){    try {        $validated = $request->validate([            'name' => 'required|string|max:255',        ]);         $user = User::create($validated);         return response()->json([            'data' => $user        ], 201); // Good 201 status code for success    } catch (ValidationException $e) {        return response()->json([            'message' => 'Validation failed',            'errors' => $e->errors()        ], 422); // Not 200 anymore!    } catch (\Exception $e) {        return response()->json([            'error_message' => 'Server error'        ], 500); // Also not 200!    }}`
There's even a classical meme about it, **found on Reddit**:
And it's not just about error/success status code. As you can see in the example, we're returning 4xx code for validation, and 5xx code for general Exception on the server.
So, use appropriate HTTP status codes - 201 for creation, 422 for validation errors, 404 for not found, etc. This helps API clients properly understand the response.
**2. Not Following RESTful Conventions**
For REST APIs, developers typically use **Resource Controllers** in Laravel. Some different non-standard naming and HTTP methods can make your API confusing and hard to maintain.
**Bad Practice:**

`// Routes fileRoute::get('/getUserById/{id}', [UserController::class, 'getOneUser']);Route::post('/createUser', [UserController::class, 'makeUser']);Route::post('/deleteUser/{id}', [UserController::class, 'removeUser']);Route::get('/getAllUsers', [UserController::class, 'fetchAllUsers']);`
**Better Approach:**
Use Laravel's resource Controllers to enforce RESTful conventions:

`// Routes fileRoute::apiResource('users', UserController::class); // UserController.phpclass UserController extends Controller{    // GET /users    public function index()    {        // ...    }     // POST /users    public function store(Request $request)    {        // ...    }     // GET /users/{id}    public function show(User $user)    {        // ...    }     // PUT/PATCH /users/{id}    public function update(Request $request, User $user)    {        // ...    }     // DELETE /users/{id}    public function destroy(User $user)    {        // ...    }}`
This automatically maps HTTP verbs to CRUD actions and follows conventional REST naming patterns.
**3. Making Breaking API Changes Without Versioning**
If you have real API clients already using your API, changing response structures or endpoint behaviors can break them.
**Bad Practice:**

`// BEFORE: Client expects 'name' fieldpublic function show($id){    $user = User::findOrFail($id);    return [        'id' => $user->id,        'name' => $user->name,        'email' => $user->email    ];} // AFTER: Breaking change! 'name' split into 'first_name' and 'last_name'public function show($id){    $user = User::findOrFail($id);    return [        'id' => $user->id,        'first_name' => $user->first_name,        'last_name' => $user->last_name,        'email' => $user->email    ];}`
**Better Approach - Option 1:**
Use API versioning:

`// routes/api.phpRoute::prefix('v1')->group(function () {    Route::apiResource('users', 'Api\V1\UserController');}); Route::prefix('v2')->group(function () {    Route::apiResource('users', 'Api\V2\UserController');}); // App\Http\Controllers\Api\V1\UserController.php - Original structure// App\Http\Controllers\Api\V2\UserController.php - New structure`
If you want to find out more about API versioning, we have a **long tutorial about it**.
**Better Approach - Option 2:**
Or maintain backward compatibility:

`public function show($id){    $user = User::findOrFail($id);    return [        'id' => $user->id,        'name' => $user->first_name . ' ' . $user->last_name, // Keep for compatibility        'first_name' => $user->first_name,        'last_name' => $user->last_name,        'email' => $user->email    ];}`
**4. NOT Using API Resources ("Reinventing the Wheel")**
Many developers create custom response formatters when Laravel already offers **API Resources**.
**Bad Practice:**

`public function index(){    $users = User::all();    $response = [];     foreach ($users as $user) {        $response[] = [            'id' => $user->id,            'full_name' => $user->name,            'email_address' => $user->email,             // ... Manual transformation        ];    }     return response()->json(['data' => $response]);}`
**Better Approach:**
Use Laravel's API Resources:

`// Create with: php artisan make:resource UserResourceclass UserResource extends JsonResource{    public function toArray($request)    {        return [            'id' => $this->id,            'name' => $this->name,            'email' => $this->email,            'joined_date' => $this->created_at->format('Y-m-d'),            'is_admin' => $this->hasRole('admin'),        ];    }} // Then in your controller:public function index(){    return UserResource::collection(User::all());} public function show(User $user){    return new UserResource($user);}`
API resources provide not only consistent structure, but also pagination support, and better maintainability.
There's even a saying *"don't fight the framework"*. Of course, there are exceptions, but you have to have a good reason to creating something custom here.
**5. Inconsistent Error Response Structure**
We already talked about API status code for success/errors, now let's talk about error messages.
Having different error formats across your API endpoints makes client error handling difficult.
**Bad Practice:**

`// In one controller:return response()->json(['error' => 'User not found'], 404); // In another controller:return response()->json(    ['status' => 'fail', 'message' => 'Not found'], 404); // In a third controller:return response()->json(    ['code' => 404, 'details' => 'The user does not exist'], 404);`
**Better Approach:**
Create a consistent error handling trait:

`// App\Traits\ApiResponder.phptrait ApiResponder{    protected function success($data, $code = 200)    {        return response()->json(['data' => $data], $code);    }     protected function error($message, $code)    {        return response()->json([            'error' => [                'message' => $message,                'code' => $code            ]        ], $code);    }} // Then in your controller:use App\Traits\ApiResponder; class UserController extends Controller{    use ApiResponder;     public function show($id)    {        $user = User::find($id);         if (!$user) {            return $this->error('User not found', 404);        }         return $this->success(new UserResource($user));    }}`
**6. Missing Rate Limiting**
APIs without rate limiting are vulnerable to abuse, whether intentional (*like a DDoS attack*) or accidental (*just poorly written scraper*).
Use Laravel's **built-in rate limiting**:

`// app/Providers/AppServiceProvider.phpuse Illuminate\Cache\RateLimiting\Limit;use Illuminate\Http\Request;use Illuminate\Support\Facades\RateLimiter; protected function boot(): void{    RateLimiter::for('api', function (Request $request) {        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());    });}`
**Conclusion**
All in all, remember that Laravel provides many tools to help implement these best practices - use them rather than fighting the framework's conventions.
Anything we've missed in this list?
**Recent Courses on Laravel Daily
Laravel 13 Starter Kit Teams and Customizations**10 lessons33 min**April 6, 2026
Testing in Laravel 13 For Beginners**26 lessons1 h 41 min read**April 1, 2026
How to Build Laravel 13 API From Scratch**30 lessons1 h 23 min**March 26, 2026**Get notified when participatingGavin Kimpson****
I would probably add caching as well - making lots of the same calls can be reduced with caching especially good if the API has some form of rate limiting👍 1Povilas Korop****
Yeah good point, but it's not a *bad* practice I would say, it's more like an improvement. Also, properly using cache is not a trivial skill.Ilya Savianok****
What about race condition? It's very important for a server to prevent multiple calls, especially when the system operates with money or some other sensitive stuff.Povilas Korop****
Yeah, also important but more like "advanced" skill, I would say, for larger or, as you're saying, more sensitive projects.Ilya Savianok****
But is it really advanced? It's just a protection from multiple clicks on a website when debouncing is offPovilas Korop****
Advanced in terms of how to implement it (and, more importantly, how to TEST it) properly. It's not "just" a protection, like validation rule or something similar.Steven Stratford****
I have a question about the resource routes. I had them , but then i added something with relationships and had to add a route for attach/dettach. So what would be the best practis for adding this routes ?Povilas Korop****
The non-resource custom routes ON TOP of resource are usually personal preference. There's no "best practice" here.Hassan Tahseen****
for
***1. Inconsistent Error Response Structure***
I would suggest to throw one of built in exceptions or a generic excpetion with some usefull message rather than making a trait. especially when we are using API Resources. what do you think?Modestas****
It is a personal preference. With the trait - you can always assume that the response will be with identical structure. Without it - you may end up with different response structures, that can lead to confusion.halil****
Hi Povilas,
First of all thank you for this post. What do you think about the json api spec?
I actually think it is an approach that covers all of these problems and keeps them under control so that fewer problems arise in the long term.
Especially the laraveljsonapi.io package doesn't look bad at all. I would love for you to make a detailed api video / article series on this subject.
Thank you.Povilas Korop****
Hi Halil, thanks for the comment. Yes, it's in my plans to talk about JSON API and expand the current API course with more topics on "standards", whenever I have more free time. This Summer is pretty harsh for me to plan - new things like Nightwatch, Filament 4, Cursor/AI news, family vacations, etc.
But I will get to this topic :)halil****
Thank you for taking the time to answer my question :) All the best.

`Leave a comment`

You can use Markdown**Comment**

![](https://laraveldaily.com/uploads/2025/02/api-200-return-400-meme.jpg)

Gavin Kimpson avatar

Povilas Korop avatar

Divesh avatar

Ilya Savianok avatar

Povilas Korop avatar

Ilya Savianok avatar

Povilas Korop avatar

Divesh avatar

Steven Stratford avatar

Povilas Korop avatar

Divesh avatar

Hassan Tahseen avatar

1. 1. Inconsistent Error Response Structure

Modestas avatar

Divesh avatar

halil avatar

Povilas Korop avatar

halil avatar

Divesh avatar

Divesh avatar

**Laravel Daily**
Your trusted source for modern Laravel courses, tutorials, and insights. Learn from real-world examples and level up your development skills.
**Learn**
• Premium Courses
• Roadmap
• Project Examples
**Company**
• About
• Newsletter
• Pricing
© 2026 Laravel Daily. All rights reserved.
For any questions - email info@laraveldaily.com

!Laravel Daily

- • Premium Courses
- • Roadmap
- • Project Examples
- • About
- • Newsletter
- • Pricing

**Feedback**
