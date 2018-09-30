### ActiveRecord apartment
---

https://github.com/influitive/apartment

####

```sh
gem 'apartment'
bundle exec rails g apartment:install

```

```ruby
# config/initializers/apartment.rb
Apartment::Tenant.create('tenant_name')

Apartment::Tenant.switch('tenant_name') do
end

# config/application.rb
require 'apartment/elevators/stbdomain'

# application.rb
module MyAppliction
  class Application < Rails::Application
    config.middleware.use Apartment::Elevators::Subdomain
  end
end

# config/initializers/apartment/subdomain_exclusions.rb
Apartment::Elevators::Subdomain.excluded_subdomains = ['www']

# application.rb
module MyApplication
  class Application < Rails::Application
    config.middleware.use Apartment::Elevators::FirstSubdomain
  end
end

# config/initializers/apartment/subdomain_exclusions.rb
Apartment::Elevators::FirstSubdomain.excluded_subdomains = ['www']

# application.rb
module MyApplication
  class Application < Rails::Application
    config.middleware.use Apartment::Elevators::Domain
  end
end

# application.rb
module MyApplication
  class Application < Rails::Application
    conifg.middleware.use Apartment::Elevators::HostHash, {'example.com' => 'example_tenant'}
  end
end

# application.rb
module MyApplication
  class Application < Rails::Application
    config.middleware.use Apartment::Elevators::Host
  end
end

Apartment::Elevators::Host.ignored_first_subdomains = ['www']

# application.rb
module MyApplication
  class Application < Rails::Application
    conifg.middleware.use Apartment::Elevators::Generic, Proc.new { |request| request.host.reverse }
  end
end

# app/middleware/my_custom_elevator.rb
class MyCustomElevator < Apartment::Elevators::Generic
  def parse_tenant_name(request)
    tenant_name = SomeModel.from_request(request)
    return tenant_name
  end
end

Rails.application.config.middleware.use Aparment::Elevators::Subdomain

Rails.application.config.middleware.insert_before Warden::Manager, Apartment::Elevators::Subdomain

Apartment::Tenant.drop('tenant_name')

# config/initializers/apartment.rb
Apartment.configure do |config|
end

config.excluded_models = ["User", "Company"]

config.default_schema = "some_other_schema"

config.persistent_schemas = ['some', 'other', 'schemas']

# lib/tasks/db_enhancements.rake
namespace :db do
  desc 'Also create shared_extensions Schema'
  task :extensions => :environment do
    ActiveRecord::Base.connection.execute 'CREATE SCHEMA IF NOT EXISTS shared_extensions;'
    ActiveRecord::Base.connection.execute 'CREATE EXTENSION IF NOT EXISTS HSTORE SCHEMA share_extensions;'
    ActiveRecord::Base.connection.execute 'CREATE EXTENSION IF NOT EXISTS "uuid-ossp" SCHEMA shared_extensions;'
    ActiveRecord::Base.connection.execute 'GRANT usage ON SHEMA shared_extensions to public;'
  end
end
Rake::Task["db:create"].enhance do
  Rake::Task["db:extensions"].invoke
end
Rake::Task["db:test:purge"].echance do
  Rake::Task["db:extensions"].invoke
end
```
```yml
adapter: postgresql
schema_search_path: "public,shared_extensions"
```
```ruby
# config/initializers/apartment.rb
config.persistent_schemas = ['shared_extensions']
```
```sh
psql -U postgres -d template1 -c "CREATE SCHEMA shared_extensions AUTHORIZATION some_username;"
psgl -U postgres -d template1 -c "CREATE EXTENSION IF NOT EXISTS hstore SCHEMA shared_extensions;"
# db/structure.sql
config.use_sql = true

rake db:migrate
```

```ruby
config.tenant_names = lambda{ Customer.pluck(:tenant_name) }
config.tenant_names = ['tenant1', 'tenant2']

config.prepend_environment = !Rails.env.production?

config.with_multi_server_setup = true
config.tenant_names = {
  'tenant1' => {
    adapter: 'postgresql',
    host: 'some_server',
    port: 5555,
    database: 'postgres'
  }
}
config.tenant_names = lambda do
  Tenant.all.each_with_object({}) do |tenant, hash|
    hash[tenant.name] = tenant.db_configuration
  end
end






```


