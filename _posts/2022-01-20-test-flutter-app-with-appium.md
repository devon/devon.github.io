---
layout: post
title:  "Test Flutter app with Appium"
date:   2022-01-20 12:23:06 -0500
categories: [Flutter, Ruby]
tags: [dart, flutter, ruby, test, appium]
---

## appium flutter driver

### Enviroment

```
Flutter (Channel stable, 2.8.1)
ruby 3.1.0p0
appium-flutter-driver@0.0.34
```

### Install Appium

```shell
# shell
npm install -g appium
npm install -g appium-flutter-driver
npm install -g appium-doctor

# verify install
appium-doctor

# run server
appium
```

### Prepare Flutter demo project

```shell
# shell
mkdir FlutterAppium
cd FlutterAppium

# create a flutter demo project
flutter create demo
```

Add `integration_test` 

```yml
# pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
```

Install packages

```shell
# shell
pub get
```

Modify `main.dart`, add `enableFlutterDriverExtension`

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:flutter_driver/driver_extension.dart';

void main() {
  enableFlutterDriverExtension(); // Add it before runApp
  runApp(const MyApp());
}
```

Add `ValueKey` for counter result widget:

```dart
// main.dart
Text(
  '$_counter',
  style: Theme.of(context).textTheme.headline4,
  key: const ValueKey('counter'), // Add value key
),
```

Compile a debug apk:

```shell
# shell
flutter build apk --debug
```

We will get an `app-debug.apk`, and we will use this apk to run the test.

### Prepare ruby test code

Make sure you have already installed [ruby](https://www.ruby-lang.org/en/) and [bundler](https://bundler.io/). 

```shell
# shell
cd ..
mkdir test
cd test

# Generate Gemfile
bundle init
```

Add gems to Gemfile

```ruby
# Gemfile
# frozen_string_literal: true

source "https://rubygems.org"

gem 'appium_flutter_finder'
gem 'appium_lib_core'
gem 'minitest'
```

Install gems:

```shell
# shell
bundle install
```

Add test code:

```ruby
# android_app_test.rb
require 'appium_lib_core'
require 'appium_flutter_finder'

require 'minitest/autorun'

class ExampleTests < Minitest::Test
  include ::Appium::Flutter::Finder

  ANDROID_CAPS = {
    caps: {
      platformName: 'Android',
      automationName: 'flutter',
      deviceName: 'Pixel 2',
      app: "../demo/build/app/outputs/flutter-apk/app-debug.apk"
    },
    appium_lib: {
      export_session: true,
      wait_timeout: 20,
      wait_interval: 1
    }
  }.freeze

  def test_run_example_android_scenario
    @core = ::Appium::Core.for(ANDROID_CAPS)
    @driver = @core.start_driver

    @driver.execute_script 'flutter:getRenderTree'
    assert_equal 'ok', @driver.execute_script('flutter:checkHealth', {})

    # Find and click FloatingActionButton
    tooltip_finder = by_tooltip 'Increment'
    @driver.execute_script('flutter:waitFor', tooltip_finder, 100)
    floating_button_element = ::Appium::Flutter::Element.new(@driver, finder: tooltip_finder)
    floating_button_element.click

    # Confirm the count result
    counter_finder = by_value_key 'counter'
    counter_element = ::Appium::Flutter::Element.new(@driver, finder: counter_finder)
    assert_equal '1', counter_element.text
  end
end
```

Run an [Android emulator](https://developer.android.com/studio/run/emulator) from Android Studio or command line, then run command to test the app.

```shell
# shell
ruby android_app_test.rb
```

We will get the test result:

![flutter-appium-android-01](/assets/images/2022/flutter-appium-android-01.png)

## Appium@next

Next, we will try to use latest appium 2.x.

### Enviroment

```
Appium v2.0.0-beta.24
```

### Install appium

```shell
# shell
npm install -g appium@next
appium driver install flutter

# confirm install
appium driver list --installed

# run appium server
appium
```

All other steps are the same as `appium flutter driver`, except we add the `server_url` in configuration:

```ruby
# android_app_test.rb
appium_lib: {
  server_url: 'http://127.0.0.1:4723', // Add server url
  export_session: true,
  wait_timeout: 20,
  wait_interval: 1
}
```

### Run test

```shell
# shell
ruby android_app_test.rb
```



## Read more

https://github.com/appium/ruby_lib

https://github.com/appium-userland/appium-flutter-driver