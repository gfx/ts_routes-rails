# TsRoutes for Rails [![Gem Version](https://badge.fury.io/rb/ts_routes.svg)](https://badge.fury.io/rb/ts_routes) [![Build Status](https://travis-ci.org/bitjourney/ts_routes-rails.svg?branch=master)](https://travis-ci.org/bitjourney/ts_routes-rails)

This gem generates Rails URL helpers in TypeScript, which is synchronized to `routes.rb`.

This is inspired by [js-routes](https://github.com/railsware/js-routes), which invents the great idea to export URL helpers to JavaScript.

## Usage

In your `lib/tasks/ts_routes.rake`:

```ruby
namespace :ts do
  TS_ROUTES_FILENAME = "javascripts/generated/routes.ts"

  desc "Generate #{TS_ROUTES_FILENAME}"
  task routes: :environment do
    Rails.logger.info("Generating #{TS_ROUTES_FILENAME}")
    source = TsRoutes.generate(
        exclude: [/admin/, /debug/],
    )
    File.write(TS_ROUTES_FILENAME, source)
  end
end
```

Then, execute `rake ts:routes` to generate `routes.ts` in your favorite path.

And you can import it in TypeScript code:

```typescript
import * as Routes from './generated/routes';

console.log(Routes.entriesPath({ page: 1, per: 20 })); // => /entries?page=1&per=20
console.log(Routes.entryPath(1)); // => /entries/1

// "anchor" is a special keyword to add anchors (as Rails's does)
console.log(Routes.entryPath(1, { anchor: 'foo' })); // => /entries/1#foo
```

Generated URL helpers are almost compatible with Rails, but they have some restriction:

* You must pass required parameters to the helpers as non-named (i.e. normal) arguments
  * i.e. `Routes.entryPath(1)` for `/entries/:id`
  * `Routes.entryPath({ id })` is not allowed
* Required parameters must not be `null` nor `undefined`
  * i.e. `Routes.entyPath(null)` does not compile
* You must pass optional parameters as the last argument
  * i.e. `Routes.entriesPath({ page: 1, per: 2 })`

### Options

Here are options for `TsRoutes.generate`:

<table>
<tr><th>name</th><th>description</th><th>default</th></tr>
<tr><td>routes</td><td>Rails routes to export</td><td>Rails.application.routes</td></tr>
<tr><td>camel_case</td><td>naming style; doesn't change if false</td><td>true</td></tr>
<tr><td>route_suffix</td><td>suffix for each route</td><td>"path"</td></tr>
<tr><td>include</td><td>Array of Regexp patterns to include</td><td>nil</td></tr>
<tr><td>exclude</td><td>Array of Regexp patterns to exclude</td><td>nil</td></tr>
<tr><td>header</td><td>additional parts of generated files</td><td>"/* tslint:disable */"</td></tr>
</table>

Note that `TsRoutes.generate(options)` is a shortcut of `TsRoutes::Generator.new(options).generate`.

### How to Keep routes.ts Up-To-Date

Use [Guard](https://github.com/guard/guard):

```ruby
# In Guardfile

# Run `rake ts:routes` when routes.rb is updated.
guard :rake, task: 'ts:routes' do
  watch(%r{config/routes\.rb$})
end
```

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'ts_routes'
```

And then execute:

```console
$ bundle
```

Or install it yourself as:

```console
$ gem install ts_routes
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/bitjourney/ts_routes-rails.

## Copyright and Licenses

Copyright 2017 Bit Journey, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
