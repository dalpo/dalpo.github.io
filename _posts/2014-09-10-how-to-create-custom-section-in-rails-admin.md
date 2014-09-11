---
layout: post

title: 'How to create a custom section for the RailsAdmin Engine'
subtitle: 'In RailsAdmin a section is a configuration DSL for a custom action'
cover_image: ra-section-cover.jpg

excerpt: "How to create a custom section to configure your reusable Rails Admin custom action"

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

{% highlight ruby %}
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
{% endhighlight %}

So you can specify customize configurations in the rails_admin configuration initializer:

{% highlight ruby %}
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
{% endhighlight %}

Every value of your Section could be retrieved with the model_config object in your custom action. For instance:

{% highlight ruby %}
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
{% endhighlight %}

Important note:

When you define a custom section this must be initialized before rails_admin to work correctly. So in your Gemfile you must define rails_admin after your custom action gem.

{% highlight ruby %}
gem 'rails_admin_custom_action'
gem 'rails_admin'
{% endhighlight %}

