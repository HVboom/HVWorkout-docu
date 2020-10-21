---
title: Multi Language Support
author: Mario
layout: article
show_title: true
show_tags: true
aside:
  toc: true
---

## Introduce I18n support
If you plan to provide your application in different languages, then start as early as possible. Even if you do not have plans now I recommend to prepare everything right away.
You will find a lot of information regarding that topic and I will only list some basic sources to start:
* [Rails Guides - Rails Internationalization (I18n) API](https://guides.rubyonrails.org/i18n.html)
* [MobileAppDaily.com](https://www.mobileappdaily.com/multi-language-support-guide-for-rails-app)
* [Lokalise.com](https://lokalise.com/blog/rails-i18n/)
* [Devise Wiki](https://github.com/heartcombo/devise/wiki/I18n)
  * Gem [devise-i18n](https://github.com/tigrish/devise-i18n)
* [kapeli.com - cheat sheet](https://kapeli.com/cheat_sheets/Rails_i18n.docset/Contents/Resources/Documents/index)

### Necessary Gems
Add following Gems to the `Gemfile`:
  ```ruby
  ...
  # Multi-Language support
  gem 'rails-i18n'
  
  # Select preferred user language
  gem 'http_accept_language'
  ...
  ```

### Configuration changes
* Add a new initializer `config/initializers/locales.rb` to setup the I18n support:
  ```ruby
  # Search path setup for I18n data files
  I18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
  
  # Permitted locales available for the application
  I18n.available_locales = [:en, :de]
  I18n.enforce_available_locales = true
  
  # Set default locale
  I18n.default_locale = :de
  ```
* Set the current locale in the generated page header `app/views/layouts/application.html.haml`:
  ```haml
	...
	%meta{'http-equiv': 'Content-Language', content: "#{locale}"}/
	...
	```
	
### Adopting routing
All routes to pages where you wish to support multiple languages have to be grouped into a scope. The scope definition `scope "(:locale)", locale: /#{I18n.available_locales.join("|")}/ do` sets automatically the `param[:locale]`, if the url starts with a supported language (ensured with the regex constraint).

Additionally  following _Catch All_ route ensures that the user cannot mess up with the URL:
`get '*path', to: 'static_pages#home', locale: I18n.default_locale, constraints: lambda { |req| req.path.exclude? 'rails/' }`
The constraint still allows Rails 6 specific _action\_mailbox_, _conductor_ and _active\_storage_ routes.
### Adjustment to the Application Controller
Setup an around action to set the locale using following priorities:
1. use the language provided in the URL (see scoped routing above)
2. use the preferred language of the browser
3. use the default language (see configuration  above)

File `app/controllers/application_controller.rb`:
  ```ruby
  class ApplicationController < ActionController::Base
    prepend_before_action :switch_locale
  
  ...
  
    private
  
    def default_url_options
      {locale: I18n.locale}
    end
  
    def switch_locale
      I18n.locale = params[:locale] ||
                    http_accept_language.compatible_language_from(I18n.available_locales) ||
                    I18n.default_locale
    end
  end
  ```
#### Use browser default language settings
I found the easy to use [http_accept_language](https://github.com/iain/http_accept_language) gem allowing to set a compatible language with a call to `http_accept_language.compatible_language_from(I18n.available_locales)`
### Language selection feature
To allow the user to switch to a different language at any time provide a drop-down in the navigation showing the currently selected language. It iterates through the configured supported languages `I18n.available_locales` (ignoring the currently selected language). The Rails feature `url_for(locale: locale)` is used to generate a link to the current page with a different language. (see [Stackoverflow - rails-link-to-current-page-and-passing-parameters-to-it](https://stackoverflow.com/questions/2543576/rails-link-to-current-page-and-passing-parameters-to-it))

Create a partial `app/views/layouts/_multi_language.html.haml`:
  ```haml
  .navbar-nav.navbar-right.nav-item.dropdown
    %a#languageDropdown.nav-link.dropdown-toggle{aria: {label: "#{t(:language_selection)}", expanded: 'false', haspopup: 'true'}, data: {toggle: 'dropdown'}, href: '#', role: 'menu'}
      = image_pack_tag "flags/#{locale}.svg", class: 'hvworkout__language-img'
      %span.sr-only= t('.language.name', locale: locale)
    .dropdown-menu{aria: {labelledby: 'languageDropdown'}}
      - I18n.available_locales.reject{|l| l == locale}.sort.each do |locale|
        = link_to url_for(locale: locale), class: 'dropdown-item' do
          = image_pack_tag "flags/#{locale}.svg", class: 'hvworkout__language-img'
          = t('.language.name', locale: locale)
  ```

Call that partial in `app/views/layouts/_navigation.html.haml`.
### Adjusting views and templates
#### Use dedicated templates
The content of static pages in different languages is completely independent of each other. Therefore the best way to support such kind of translation is to introduce language specific templates. Though for the _About_ page exist an English version `app/views/static_pages/about.en.html.haml` and an independent German template version `app/views/static_pages/about.de.html.haml`. You need just to add the locale to the file name.

The same works for partials too. See example `app/views/static_pages/_content.<locale>.html.haml` used in `app/views/static_pages/home.html.haml`.
#### Use specific Gems
A lot of popular _Gems_ provide add-ons to support a multi-language setup. Good examples and really worth to look into the implementation are:
* [rails-i18n](https://github.com/svenfuchs/rails-i18n)
* [devise-i18n](https://github.com/tigrish/devise-i18n)

#### Dirty hands
The last resort is always to get the hands dirty and do it manually. For the version 0.9 of the app I created two files `config/locales/en.yml` and `config/locales/de.yml`. Maybe later on when the app grows I have to split them into dedicated files. You can either use different names `xxx.en.yml` or create a deeply nested sub-folder structure. The load path already supports such sub-folders.

English version:
```yml
en:
  activerecord:
    attributes:
      user:
        nickname: 'Nickname'
  gravatar:
    edit: 'Edit'
    home: 'on Gravatar.com'
  layouts:
    application:
      app:
        description: 'HVWorkout - Tracking Kieser Training exercises'
        title: 'HVWorkout'
    authentication:
      log_out: 'Log out'
      profile: 'Profile'
    multi_language:
      language:
        name: 'English'
        selection: 'Language selection'
    navigation:
      about: 'About'
      home: 'Home'
  shared:
    actions:
      back: 'Back'
      reset: 'Reset'
```

German version:
```yml
de:
  activerecord:
    attributes:
      user:
        nickname: 'Kurzname'
  gravatar:
    edit: 'Bearbeiten'
    home: 'auf Gravatar.com'
  layouts:
    application:
      app:
        description: 'HVWorkout - Protokoll meiner Kieser Training Übungen'
        title: 'HVWorkout'
    authentication:
      log_out: 'Ausloggen'
      profile: 'Profil'
    multi_language:
      language:
        name: 'Deutsch'
        selection: 'Sprachauswahl'
    navigation:
      about: 'Info'
      home: 'Startseite'
  shared:
    actions:
      back: 'Zurück'
      reset: 'Zurücksetzen'
```
