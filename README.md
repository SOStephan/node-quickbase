node-quickbase
==============

[![npm license](https://img.shields.io/npm/l/quickbase.svg)](https://www.npmjs.com/package/quickbase) [![npm version](https://img.shields.io/npm/v/quickbase.svg)](https://www.npmjs.com/package/quickbase) [![npm downloads](https://img.shields.io/npm/dm/quickbase.svg)](https://www.npmjs.com/package/quickbase) [![Build Status](https://travis-ci.org/tflanagan/node-quickbase.svg)](https://travis-ci.org/tflanagan/node-quickbase)

A lightweight, very flexible QuickBase API

[API Documentation](https://github.com/tflanagan/node-quickbase/blob/master/documentation/api.md)

[PHP Version](https://github.com/tflanagan/php-quickbase)

Install
-------
```
# Latest Stable Release
$ npm install quickbase

# Latest Commit
$ npm install tflanagan/node-quickbase

# Bower Install
$ bower install quickbase
```

Bower installation only includes the `quickbase.browserify.min.js` file.

Browserify
----------
This library works out of the box with Babel+Browserify.
```
$ npm install quickbase
$ npm install -g babel-cli babel-preset-es2015 browserify minifier
$ babel --presets es2015 quickbase.js > quickbase.es5.js
$ browserify quickbase.es5.js > quickbase.browserify.js
$ minify quickbase.browserify.js > quickbase.browserify.min.js
```
This exposes the QuickBase object to the global namespace (```window.QuickBase || QuickBase```).

__Warning: Native Browser Promises do not share the same functionality as Bluebird Promises!__

Chaining functions off of an ```.api()``` function call uses the Bluebird Promise Library. Declaring a ```new Promise()``` uses Native Browser Promises (if available).

node-quickbase, exposes the internal ```Promise``` object under ```QuickBase.Promise```.

The use is the same as in Nodejs, but there is no need to ```require('quickbase')```.

```html
<script type="text/javascript" src="quickbase.browserify.min.js"></script>
<script type="text/javascript">
	var quickbase = new QuickBase({
		realm: 'www',
		appToken: '*****'
	});

	...
</script>
```

Example
-------
```javascript
'use strict';

const QuickBase = require('quickbase');

const quickbase = new QuickBase({
	realm: 'www',
	appToken: '*****'
});

/* Promise Based */
quickbase.api('API_Authenticate', {
	username: '*****',
	password: '*****'
}).then((result) => {
	return quickbase.api('API_DoQuery', {
		dbid: '*****',
		clist: '3.12',
		options: 'num-5'
	}).then((result) => {
		return result.table.records;
	});
}).each((record) => {
	return quickbase.api('API_EditRecord', {
		dbid: '*****',
		rid: record[3],
		fields: [
			{ fid: 12, value: record[12] }
		]
	});
}).then(() => {
	return quickbase.api('API_DoQuery', {
		dbid: '*****',
		clist: '3.12',
		options: 'num-5'
	});
}).then((result) => {
	console.log(result);
}).catch((err) => {
	console.error(err);
});

/* Callback Based */
quickbase.api('API_Authenticate', {
	username: '*****',
	password: '*****'
}, (err, result) => {
	if (err)
		throw err;

	quickbase.api('API_DoQuery', {
		dbid: '*****',
		clist: '3.12',
		options: 'num-5'
	}, (err, result) => {
		if (err)
			throw err;

		let i = 0;

		const fin = () => {
			++i;

			if (i === result.table.records.length) {
				quickbase.api('API_DoQuery', {
					dbid: '*****',
					clist: '3.12',
					options: 'num-5'
				}, (err, result) => {
					if (err)
						throw err;

					console.log('done');
				});
			}
		};

		result.table.records.forEach((record) => {
			quickbase.api('API_EditRecord', {
				dbid: '*****',
				rid: record[3],
				fields: [
					{ fid: 12, value: record[12] }
				]
			}, (err, results) => {
				if (err)
					throw err;

				fin();
			});
		});
	});
});
```

Class
-----
```javascript
class QuickBase {

	public object defaults;

	public object actions;
	public object prepareOptions;
	public object xmlNodeParsers;

	public function api(action[, options[, callback]);

	public static function checkIsArrAndConvert(obj);
	public static function cleanXML(xml);

	public class QueryBuilder(parent, action[, options[, callback]]);
	public class Throttle([maxConnections = 10[, errorOnConnectionLimit = false]]);
	public class QuickBaseError(code, name[, message]);
	public class Promise;

}
```

License
-------

Copyright 2014 Tristian Flanagan

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
