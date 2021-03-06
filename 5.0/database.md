# استفاده از بانک اطلاعاتی در لاراول

- [تنظیمات اولیه](#configuration)
- [تعریف نوع ارتباط (خواندن/نوشتن)](#read-write-connections)
- [اجرای Query](#running-queries)
- [ کار با Transaction ها  ](#database-transactions)
- [دسترسی به کانکشن های تعریف شده](#accessing-connections)
- [log گرفتن از Query  های اجرا شده ](#query-logging)

<a name="configuration"></a>
## تنظیمات اولیه
ارتباط با بانک اطلاعاتی و اجرای Query در لاراول بسیار ساده است ، تنظیمات بانک اطلاعاتی بطور پیشفرض در فایل `config/database.php` قرار دارد . در این فایل شما میتوانید تمامی کانکشن های بانک اطلاعاتی (database) خود را تعریف  ، و مشخص کنید کدامیک بعنوان کانکشن پیشفرض درنظر گرفته شود . به عنوان مثال  تمامی بانکهای اطلاعاتی زیر  میتوانند در این فایل ذخیره شوند. 

بانک های اطلاعاتی که در حال حاضر توسط لاراول پشتیبانی می شوند : MySQL, Postgres, SQLite, and SQL Server.

<a name="read-write-connections"></a>
## تعریف نوع ارتباط (خواندن/نوشتن)
ممکن است بخواهید از یک کانکشن برای اجرای دستورات SELECT و کانکشن دیگری را برای دستورات INSERT , UPDATE و DELETE  داشته باشید . لاراول این امر را ممکن می سازد و مطابق با نوع کوئری ، کاناکشن مربوطه را برای اجرای کوئری برقرار میکند . 
برای تعریف نوع ارتباط  (خواندن/نوشتن) از این طریق عمل میکنیم :

	'mysql' => [
		'read' => [
			'host' => '192.168.1.1',
		],
		'write' => [
			'host' => '196.168.1.2'
		],
		'driver'    => 'mysql',
		'database'  => 'database',
		'username'  => 'root',
		'password'  => '',
		'charset'   => 'utf8',
		'collation' => 'utf8_unicode_ci',
		'prefix'    => '',
	],
نکته : دو کلید به آرایه تنظیمات اضافه شده است که در آنها تنها مقدار `host` تعریف شده اند ، نیازی نیست برای هرکدام ، تنظیمات مربوط به `driver` ، `database` ، `username`  مجددا وارد شود ! لاراول این مقادیر را از گزینه  های تعریف شده در آرایه `mysql` دریافت میکند . برای مثال کانکشن ازنوع `read` در مثال بالا با آدرس هاست `192.168.1.1` و نوع درایور `mysql` و مقدار `username` برابر `root` و ... تعریف خواهد شد . و برای کانکشن `write`  با همین مشخصات ولی آدرس هاست `192.168.1.2` درنظر گرفته میشود .

<a name="running-queries"></a>
## اجرای Query
زمانی که تنظیمات مربوط به بانک اطلاعاتی خود را وارد کردید حال میتوانید  کوئری ها را با استفاده از فاساد `DB`  اجرا نمایید  .  
#### اجرای یک  کوئری SELECT

	$results = DB::select('select * from users where id = ?', [1]);
متد `select` همیشه یک آرایه از رکورد ها را به عنوان نتیجه برمیگرداند .

همچنین برای اجرای کوئری و ارسال پارامتر (Binding parameters) به صورت زیر عمل میکنیم :

	$results = DB::select('select * from users where id = :id', ['id' => 1]);

#### اجرای  دستور Insert

	DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### اجرای دستور Update

	DB::update('update users set votes = 100 where name = ?', ['John']);

#### اجرای دستور Delete 

	DB::delete('delete from users');

> **نکته :** دستورات `update` و `delete` تعداد رکورد هایی که Query با موفقیت بر روی آنها انجام شده است را برمیگرداند .

#### و برای اجرای دستورات دیگر

	DB::statement('drop table users');

#### معرفی یک listener برای رویدادهای پرس و جو

برای تعریف یک Listener برای تمامی کوئری هایی که اجرا میشوند از طریق متد  `DB::listen`  عمل نموده  :

	DB::listen(function($sql, $bindings, $time)
	{
		// برای مثال شما میتوانید تمامی کوئری هایی که اجرا میشوند را در اینجا لوگ بگیرید
	});

<a name="database-transactions"></a>
## کار با Transaction ها
برای اجرای مجموعه ای از عملیات بر روی بانک اطلاعات ( `Commit` تایید کردن ،`rollback`  یا رد کردن و بازگرداندن بانک اطلاعاتی به وضعیت قبل از اجرای کوئری های مورد نظر ) برحسب نیاز میتوانید  از متد `transaction` استفاده  نمایید :

	DB::transaction(function()
	{
		DB::table('users')->update(['votes' => 1]);

		DB::table('posts')->delete();
	});

> **نکته:** هرنوع اثتثنائی که از درون Closure تعریف شده در متد   `transaction` رخ بدهد ، باعث RollBack شدن transaction  بصورت خودکار میشود .

برای  شروع اجرای یک transaction  بصورت دستی  :

	DB::beginTransaction();

و برای مردود کردن آن :

	DB::rollback();

و درآخر ، برای تایید کردن Transaction و ذخیره سازی عملیات بر روی بانک اطلاعاتی :

	DB::commit();

<a name="accessing-connections"></a>
## دسترسی به کانکشن های تعریف شده
زمانی که از چند کانکشن در پروژه تان استفاده میکنید ممکن است برحسب نیاز بخواهید از یک کانکشن مشخص برای اجرای دستورات خود استفاده نمایید ، برای اینکار از متد `DB::connection` میتوانید استفاده نمایید :

	$users = DB::connection('foo')->select(...);

برای گرفتن نمونه شیء  PDO :

	$pdo = DB::connection()->getPdo();

درخواست  مجدد برقراری ارتباط :

	DB::reconnect('foo');
اگر میخواهید ارتباط را  بدلیل برقراری بیش از حد کانکشن های همزمان ، قطع کنید از متد `disconnect`  استفاده کنید .

	DB::disconnect('foo');

<a name="query-logging"></a>
## Log کردن کوئری ها
لاراول میتواند بصورت اختیاری کوئری های اجرا شده برای درخواست جاری را درحافظه دخیره کند که به شما کمک میکند نظارگر کوئری های اجرا شده بر روی بانک اطلاعاتی خود باشید . برای فعال کردن Log گیری ، متد `enableQueryLog` را اجرا کنید :

	DB::connection()->enableQueryLog();

برای دریافت آرایه ای از کوئری های Log شده از متد  `getQueryLog` استفاده نمایید :

	$queries = DB::getQueryLog();
