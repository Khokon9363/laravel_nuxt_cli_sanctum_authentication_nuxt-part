## Laravel & Nuxt CLI Authentication - Laravel Part

A. Install & setup Sanctum

   1) composer require laravel/sanctum
   2 ) php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
   3 ) php artisan migrate
   4 ) EnsureFrontendRequestsAreStateful::class in Karnel.php

B. .env
   **Laravel Domain**
   SESSION_DOMAIN=localhost

    **Nuxt Domain**
    SANCTUM_STATEFUL_DOMAINS=localhost:3000

C. routes/api.php
    Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
        return $request->user();
    });
    // Login & Logout routes
    Route::post('/login', 'Auth\LoginController@login');
    Route::post('/logout', 'Auth\LoginController@logout');

D. config/cors.php - replace
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
                  into
    'paths' => ['api/*'],

E. App/Users.php
    use HasApiTokens

F. App/Http/Controllers/Auth/LoginController.php
    public function login(Request $request)
    {
        $request->validate([
            'email'    => 'required',
            'password' => 'required'
        ]);
        if (!Auth::attempt($request->only('email', 'password'))) {
            throw new AuthenticationException();
        }
        return $this->jsonResponse(true, auth()->user(), 200);
    }

    public function logout()
    {
        if (auth()->check()) {
            Auth::logout();
            return $this->jsonResponse(true, 'Logged out successfully', 200);
        }
        return $this->jsonResponse(true, 'Something went wrong', 404);
    }

    private function jsonResponse($success, $data, $status)
    {
        return response()->json([
            'success' => $success,
            'data'    => $data,
        ], $status);
    }


## Nuxt / Vue cli
A. login.vue
    1 ) import axios from 'axios'
    2 ) axios.defaults.withCredentials = true
    3 ) make an helper function - laravelCsrfCookies()
        every time before any function call this function and return true .
            
            methods: {
                laravelCsrfCoockie() {
                    axios.get(this.baseURL + 'sanctum/csrf-cookie').then((response) => {
                        return true
                    })
                },
                login() {
                this.laravelCsrfCoockie()
                    axios
                        .post(this.baseURL + 'api/login', {
                        email: this.email,
                        password: this.password,
                        })
                        .then((res) => {
                        localStorage.setItem('me', JSON.stringify(res.data))
                        this.loggedIn = localStorage.getItem('me')
                        this.userName = JSON.parse(localStorage.getItem('me')).data.name
                        console.log(this.loggedIn)
                        })
                        .catch((err) => {
                        console.log(err)
                        })
                },
                logout() {
                this.laravelCsrfCoockie()
                axios
                    .post(this.baseURL + 'api/logout')
                    .then((res) => {
                    localStorage.removeItem('me')
                    this.loggedIn = null
                    this.userName = 'Guest User'
                    console.log(this.loggedIn)
                    })
                    .catch((err) => {
                    console.log(err)
                    })
                },


B. Demo Login.vue
    <template>
    <div>
        <button v-if="!loggedIn" @click="login">Login</button>
        <button v-else @click="logout">logout</button>
        <h1 v-if="userName">{{ userName }}</h1>
    </div>
    </template>
    <script>
    import axios from 'axios'
    axios.defaults.withCredentials = true

    export default {
    data() {
        return {
        email: 'admin@admin.com',
        password: 'password',
        baseURL: 'http://localhost:8000/',
        loggedIn: localStorage.getItem('me'),
        userName: localStorage.getItem('me')
            ? JSON.parse(localStorage.getItem('me')).data.name
            : 'Guest User',
        }
    },
    mounted() {
        console.log(this.loggedIn)
    },
    methods: {
        laravelCsrfCoockie() {
        axios.get(this.baseURL + 'sanctum/csrf-cookie').then((response) => {
            return true
        })
        },
        login() {
        this.laravelCsrfCoockie()
        axios
            .post(this.baseURL + 'api/login', {
            email: this.email,
            password: this.password,
            })
            .then((res) => {
            localStorage.setItem('me', JSON.stringify(res.data))
            this.loggedIn = localStorage.getItem('me')
            this.userName = JSON.parse(localStorage.getItem('me')).data.name
            console.log(this.loggedIn)
            })
            .catch((err) => {
            console.log(err)
            })
        },
        logout() {
        this.laravelCsrfCoockie()
        axios
            .post(this.baseURL + 'api/logout')
            .then((res) => {
            localStorage.removeItem('me')
            this.loggedIn = null
            this.userName = 'Guest User'
            console.log(this.loggedIn)
            })
            .catch((err) => {
            console.log(err)
            })
        },
    },
    }
    </script>

**You can use this starter project for to remove the hasslement of Laravel & Sanctum/Vue Cli**