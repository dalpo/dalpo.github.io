---
layout: post

title: 'How to create a custom section for RailsAdmin Engine'
subtitle: 'Welcome to my blog!'
cover_image: blog-cover.jpg

excerpt: "How to create a custom section in RailsAdmin for a reusable custom action"

author:
  name: Andrea Dal Ponte
  twitter: dalpo
  gplus: 100687498195339762535
  bio: Full stack web developer
  image: avatar.png
---


Sections describe different views in the RailsAdmin engine. Theese are very useful if you are writing a reusable custom action.

Each section's class object can store generic configuration about that section (such as the number of visible tabs in the main navigation), while the instances (accessed via model configuration objects) store model specific configuration (such as the visibility of the model).

With the following lines you could easily write CustomAction configuration (aka Section):

```ruby
require 'rails_admin/config/sections/base'

module RailsAdmin
  module Config
    module Sections
      # Configuration of the clone action
      class SampleSection < RailsAdmin::Config::Sections::Base

        register_instance_option :sample_conf do
          nil # set a default value here
        end

        register_instance_option :another_sample_conf do
          nil # set a default value here
        end

      end
    end
  end
end
```

So you can specify customize configurations in the rails_admin configuration initializer:

```Ruby
RailsAdmin.config do |config|
  config.model 'Team' do
    sample_section do
      custom_method :some_value

      another_sample_conf do
        bindings[:object].do_something!
      end
    end
  end
end
```

Every value of your Section could be retrieved with the model_config object in your custom action. For instance:

```ruby
require 'rails_admin/config/actions'
require 'rails_admin/config/actions/base'

module RailsAdmin
  module Config
    module Actions
      class SampleCustomAction < Base
        RailsAdmin::Config::Actions.register(self)

        register_instance_option :controller do
          Proc.new do
            # retrieve configuration:
            model_config.sample_section.sample_conf
            model_config.sample_section.another_sample_conf

           # do something...

            respond_to do |format|
              format.html { render @action.template_name }
              format.js   { render @action.template_name, :layout => false }
            end

          end
        end
      end
    end
  end
end
```

Important note:

When you define a custom section this must be initialized before rails_admin to work correctly. So in your Gemfile you must define rails_admin after your custom action gem.

```Ruby
gem 'rails_admin_custom_action'
gem 'rails_admin'
```

