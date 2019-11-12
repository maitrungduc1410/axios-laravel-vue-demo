# Setup
Run the following commands:
```
composer install
npm install
cp .env.example .env
php artisan key:generate
```
Then setup the `.env` with your database infor
Next, migrate DB:
```
php artisan migrate
```
Now start the app:
```
php artisan serve
npm run dev # in another terminal window
```