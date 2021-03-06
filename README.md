# TTY::Prompt [![Gitter](https://badges.gitter.im/Join%20Chat.svg)][gitter]
[![Gem Version](https://badge.fury.io/rb/tty-prompt.svg)][gem]
[![Build Status](https://secure.travis-ci.org/piotrmurach/tty-prompt.svg?branch=master)][travis]
[![Code Climate](https://codeclimate.com/github/piotrmurach/tty-prompt/badges/gpa.svg)][codeclimate]
[![Coverage Status](https://coveralls.io/repos/github/piotrmurach/tty-prompt/badge.svg)][coverage]
[![Inline docs](http://inch-ci.org/github/piotrmurach/tty-prompt.svg?branch=master)][inchpages]

[gitter]: https://gitter.im/piotrmurach/tty
[gem]: http://badge.fury.io/rb/tty-prompt
[travis]: http://travis-ci.org/piotrmurach/tty-prompt
[codeclimate]: https://codeclimate.com/github/piotrmurach/tty-prompt
[coverage]: https://coveralls.io/github/piotrmurach/tty-prompt
[inchpages]: http://inch-ci.org/github/piotrmurach/tty-prompt

> A beautiful and powerful interactive command line prompt.

**TTY::Prompt** provides independent prompt component for [TTY](https://github.com/piotrmurach/tty) toolkit.

## Features

* Number of prompt types for gathering user input
* A robust API for validating complex inputs
* User friendly error feedback

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'tty-prompt'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install tty-prompt

## Contents

* [1. Usage](#1-usage)
* [2. Interface](#2-interface)
  * [2.1 ask](#21-ask)
  * [2.2 settings](#22-settings)
    * [2.2.1 convert](#221-convert)
    * [2.2.2 default](#222-default)
    * [2.2.3 echo](#223-echo)
    * [2.2.4 in](#224-in)
    * [2.2.5 modify](#225-modify)
    * [2.2.6 required](#226-required)
    * [2.2.7 validate](#227-validate)
    * [2.2.8 messages](#228-messages)
    * [2.2.9 prefix](#229-prefix)
    * [2.2.10 active_color](#2210-active_color)
    * [2.2.11 help_color](#2211-help_color)
  * [2.3 keypress](#23-keypress)
  * [2.4 multiline](#24-multiline)
  * [2.5 mask](#25-mask)
  * [2.6 yes?/no?](#26-yesno)
  * [2.7 select](#27-select)
  * [2.8 multi_select](#28-multi_select)
  * [2.9 enum_select](#29-enum_select)
  * [2.10 expand](#210-expand)
  * [2.11 collect](#211-collect)
  * [2.12 suggest](#212-suggest)
  * [2.13 slider](#213-slider)
  * [2.14 say](#214-say)
  * [2.15 ok](#215-ok)
  * [2.16 warn](#216-warn)
  * [2.17 error](#217-warn)

## 1. Usage

In order to start asking questions on the command line, create prompt:

```ruby
prompt = TTY::Prompt.new
```

and then call `ask` with the question for simple input:

```ruby
prompt.ask('What is your name?', default: ENV['USER'])
# => What is your name? (piotr)
```

To confirm input use `yes?`:

```ruby
prompt.yes?('Do you like Ruby?')
# => Do you like Ruby? (Y/n)
```

If you want to input password or secret information use `mask`:

```ruby
prompt.mask("What is your secret?")
# => What is your secret? ••••
```

Asking question with list of options couldn't be easier using `select` like so:

```ruby
prompt.select("Choose your destiny?", %w(Scorpion Kano Jax))
# =>
# Choose your destiny? (Use arrow keys, press Enter to select)
# ‣ Scorpion
#   Kano
#   Jax
```

Also, asking multiple choice questions is a breeze with `multi_select`:

```ruby
choices = %w(vodka beer wine whisky bourbon)
prompt.multi_select("Select drinks?", choices)
# =>
#
# Select drinks? (Use arrow keys, press Space to select and Enter to finish)"
# ‣ ⬡ vodka
#   ⬡ beer
#   ⬡ wine
#   ⬡ whisky
#   ⬡ bourbon
```

To ask for a selection from enumerated list you can use `enum_select`:

```ruby
choices = %w(emacs nano vim)
prompt.enum_select("Select an editor?", choices)
# =>
#
# Select an editor?
#   1) /bin/nano
#   2) /usr/bin/vim.basic
#   3) /usr/bin/vim.tiny
#   Choose 1-3 [2]:
```

However, if you have a lot of options to choose from you may want to use `expand`:

```ruby
choices = [
  { key: 'y', name: 'overwrite this file', value: :yes },
  { key: 'n', name: 'do not overwrite this file', value: :no },
  { key: 'a', name: 'overwrite this file and all later files', value: :all },
  { key: 'd', name: 'show diff', value: :diff },
  { key: 'q', name: 'quit; do not overwrite this file ', value: :quit }
]
prompt.expand('Overwrite Gemfile?', choices)
# =>
# Overwrite Gemfile? (enter "h" for help) [y,n,a,d,q,h]
```

If you wish to collect more than one answer use `collect`:

```ruby
result = prompt.collect do
  key(:name).ask('Name?')

  key(:age).ask('Age?', convert: :int)

  key(:address) do
    key(:street).ask('Street?', required: true)
    key(:city).ask('City?')
    key(:zip).ask('Zip?', validate: /\A\d{3}\Z/)
  end
end
# =>
# {:name => "Piotr", :age => 30, :address => {:street => "Street", :city => "City", :zip => "123"}}
```

## 2. Interface

### 2.1 ask

In order to ask a basic question do:

```ruby
prompt.ask("What is your name?")
```

However, to prompt for more complex input you can use robust API by passing hash of properties or using a block like so:

```ruby
prompt.ask("What is your name?") do |q|
  q.required true
  q.validate /\A\w+\Z/
  q.modify   :capitalize
end
```

### 2.2 settings

Below is a list of the settings that may be used for customizing `ask`, `mask`, `multiline`, `keypress` methods behaviour:

```ruby
:convert    # conversion applied to input such as :bool or proc
:default    # default value used if none is provided
:echo       # turn echo on and off (default: true)
:in         # specify range '0-9', '0..9', '0...9' or negative '-1..-9'
:modify     # apply answer modification :upcase, :downcase, :trim, :chomp etc..
:required   # If true, value entered must be non-empty (default: false)
:validate   # regex, proc against which input is checked
```

#### 2.2.1 convert

 The `convert` property is used to convert input to a required type. By default no conversion is performed. The following conversions are provided:

```ruby
:bool       # true or false for strings such as "Yes", "No"
:date       # date type
:datetime   # datetime type
:file       # File object
:float      # decimal or error if cannot convert
:int        # integer or error if cannot convert
:path       # Pathname object
:range      # range type
:regexp     # regex expression
:string     # string
:symbol     # symbol
```

For example, if you are interested in range type as answer do the following:

```ruby
prompt.ask("Provide range of numbers?", convert: :range)
# Provide range of numbers? 1-10
# => 1..10
```

You can also provide a custom conversion like so:

```ruby
prompt.ask('Ingredients? (comma sep list)') do |q|
  q.convert -> (input) { input.split(/,\s*/) }
end
# Ingredients? (comma sep list) milk, eggs, flour
# => ['milk', 'eggs', 'flour']
```

#### 2.2.2 default

The `:default` option is used if the user presses return key:

```ruby
prompt.ask('What is your name?', default: 'Anonymous')
# =>
# What is your name? (Anonymous)
```

#### 2.2.3 echo

To control whether the input is shown back in terminal or not use `:echo` option like so:

```ruby
prompt.ask('password:', echo: false)
```

#### 2.2.4 in

In order to check that provided input falls inside a range of inputs use the `in` option. For example, if we wanted to ask a user for a single digit in given range we may do following:

```ruby
ask("Provide number in range: 0-9?") { |q| q.in('0-9') }
```

#### 2.2.5 modify

Set the `:modify` option if you want to handle whitespace or letter capitalization.

```ruby
prompt.ask('Enter text:') do |q|
  q.modify :strip, :collapse
end
```

Available letter casing settings are:
```ruby
:up         # change to upper case
:down       # change to small case
:capitalize # capitalize each word
```

Available whitespace settings are:
```ruby
:trim     # remove whitespace from both ends of the input
:chomp    # remove whitespace at the end of input
:collapse # reduce all whitespace to single character
:remove   # remove all whitespace
```

#### 2.2.6 required

To ensure that input is provided use `:required` option:

```ruby
prompt.ask("What's your phone number?", required: true)
# What's your phone number?
# >> Value must be provided
```

#### 2.2.7 validate

In order to validate that input matches a given patter you can pass the `validate` option. Validate setting accepts `Regex`, `Proc` or `Symbol`.

```ruby
prompt.ask('What is your username?') do |q|
  q.validate /^[^\.]+\.[^\.]+/
end
```

The **TTY::Prompt** comes with bult-in validations for `:email` and you can use them directly like so:

```prompt
prompt.ask('What is your email?') { |q| q.validate :email }
```

### 2.2.8 messages

By default `tty-prompt` comes with predefined error messages for `required`, `in`, `validate` options. You can change these and configure to your liking either by inling them with the option:

```ruby
prompt.ask('What is your email?') do |q|
  question.validate(/\A\w+@\w+\.\w+\Z/, 'Invalid email address')
end
```

or change the `messages` key entry out of `:required?`, `:valid?`, `:range?`:

```ruby
prompt.ask('What is your email?') do |q|
  question.validate(/\A\w+@\w+\.\w+\Z/)
  question.messages[:valid?] = 'Invalid email address'
end
```

### 2.2.9 prefix

You can prefix each question asked using the `:prefix` option. This option can be applied either globally for all prompts or individual for each one:

```ruby
prompt = TTY::Prompt.new(prefix: '[?] ')
```

### 2.2.10 active_color

All prompt types support `:active_color` option. In case of `select`, `multi_select`, `enum_select` or `expand` this color is used to highlight the currently selected choice. All the resulted inputs provided by user that are read in by the prompt as answer are highlighted with this color. This option can be applied either globablly for all prompts or individually.

```ruby
prompt.select('What size?', %w(Large Medium Small), active_color: :cyan)
```

Please (see pastel)[https://github.com/piotrmurach/pastel#3-supported-colors] for all supported colors.

### 2.2.11 help_color

Prompts such as `select`, `multi_select`, `expand` support `:help_color` which is used to customize the help text. This option can be applied either globablly for all prompts or individually.

```ruby
prompt.select('What size?', %w(Large Medium Small), help_color: :cyan)
```

### 2.3 keypress

In order to ask question with a single character or keypress answer use `keypress`:

```ruby
prompt.keypress("Which one do you prefer a, b, c or d ?")
```

### 2.4 multiline

Asking for multiline input can be done with `multiline` method.

```ruby
prompt.multiline("Provide description?")
```

The reading of input will terminate when empty line is submitted.

### 2.5 mask

If you require input of confidential information use `mask` method. By default each character that is printed is replaced by `•` symbol. All configuration options applicable to `ask` method can be used with `mask` as well.

```ruby
prompt.mask('What is your secret?')
# => What is your secret? ••••
```

The masking character can be changed by passing `:mask` option:

```ruby
prompt.mask('What is your secret?', mask: '\u2665')
# => What is your secret? ♥♥♥♥♥
```

If you don't wish to show any output use `:echo` option like so:

```ruby
prompt.mask('What is your secret?', echo: false)
```

### 2.6 yes?/no?

In order to display a query asking for boolean input from user use `yes?` like so:

```ruby
prompt.yes?('Do you like Ruby?')
# =>
# Do you like Ruby? (Y/n)
```

You can further customize question by passing `suffix`, `positive`, `negative` and `convert` options. The `suffix` changes text of available options, the `positive` specifies display string for successful answer and `negative` changes display string for negative answer. The final value is a boolean provided the `convert` option evaluates to boolean.

It's enough to provide the `suffix` option for the prompt to accept matching answers with correct labels:

```ruby
prompt.yes?("Are you a human?") do |q|
  q.suffix 'Yup/nope'
end
# =>
# Are you a human? (Yup/nope)
```

Alternatively, instead of `suffix` option provide the `positive` and `negative` labels:

```ruby
prompt.yes?("Are you a human?") do |q|
  q.default false
  q.positive 'Yup'
  q.negative 'Nope'
end
# =>
# Are you a human? (yup/Nope)
```

Finally, providing all available options you can ask fully customized question:

```ruby
prompt.yes?('Are you a human?') do |q|
  q.suffix 'Agree/Disagree'
  q.positive 'Agree'
  q.negative 'Disagree'
  q.convert -> (input) { !input.match(/^agree$/i).nil? }
end
# =>
# Are you a human? (Agree/Disagree)
```

There is also the opposite for asking confirmation of negative question:

```ruby
prompt.no?('Do you hate Ruby?')
# =>
# Do you hate Ruby? (y/N)
```

Similarly to `yes?` method, you can supply the same options to customize the question.

### 2.7 select

For asking questions involving list of options use `select` method by passing the question and possible choices:

```ruby
prompt.select("Choose your destiny?", %w(Scorpion Kano Jax))
# =>
# Choose your destiny? (Use arrow keys, press Enter to select)
# ‣ Scorpion
#   Kano
#   Jax
```

You can also provide options through DSL using the `choice` method for single entry and/or `choices` for more than one choice:

```ruby
prompt.select("Choose your destiny?") do |menu|
  menu.choice 'Scorpion'
  menu.choice 'Kano'
  menu.choice 'Jax'
end
# =>
# Choose your destiny? (Use arrow keys, press Enter to select)
# ‣ Scorpion
#   Kano
#   Jax
```

By default the choice name is used as return value, but you can provide your custom values including a `Proc` object:

```ruby
prompt.select("Choose your destiny?") do |menu|
  menu.choice 'Scorpion', 1
  menu.choice 'Kano', 2
  menu.choice 'Jax', -> { 'Nice choice captain!' }
end
# =>
# Choose your destiny? (Use arrow keys, press Enter to select)
# ‣ Scorpion
#   Kano
#   Jax
```

If you wish you can also provide a simple hash to denote choice name and its value like so:

```ruby
choices = {'Scorpion' => 1, 'Kano' => 2, 'Jax' => 3}
prompt.select("Choose your destiny?", choices)
```

To mark particular answer as selected use `default` with index of the option starting from `1`:

```ruby
prompt.select("Choose your destiny?") do |menu|
  menu.default 3

  menu.choice 'Scorpion', 1
  menu.choice 'Kano', 2
  menu.choice 'Jax', 3
end
# =>
# Choose your destiny? (Use arrow keys, press Enter to select)
#   Scorpion
#   Kano
# ‣ Jax
```

For ordered choices set `enum` to any delimiter String. In that way, you can use arrows keys and numbers (0-9) to select the item.

```ruby
prompt.select("Choose your destiny?") do |menu|
  menu.enum '.'

  menu.choice 'Scorpion', 1
  menu.choice 'Kano', 2
  menu.choice 'Jax', 3
end
# =>
# Choose your destiny? (Use arrow or number (0-9) keys, press Enter to select)
#   1. Scorpion
#   2. Kano
# ‣ 3. Jax
```

You can configure help message and/or marker like so

```ruby
choices = %w(Scorpion Kano Jax)
prompt.select("Choose your destiny?", choices, help: "(Bash keyboard)", marker: '>')
# =>
# Choose your destiny? (Bash keyboard)
# > Scorpion
#   Kano
#   Jax
```

### 2.8 multi_select

For asking questions involving multiple selection list use `multi_select` method by passing the question and possible choices:

```ruby
choices = %w(vodka beer wine whisky bourbon)
prompt.select("Select drinks?", choices)
# =>
#
# Select drinks? (Use arrow keys, press Space to select and Enter to finish)"
# ‣ ⬡ vodka
#   ⬡ beer
#   ⬡ wine
#   ⬡ whisky
#   ⬡ bourbon
```

As a return value, the `multi_select` will always return an array by default populated with the names of the choices. If you wish to return custom values for the available choices do:

```ruby
choices = {vodka: 1, beer: 2, wine: 3, whisky: 4, bourbon: 5}
prompt.select("Select drinks?", choices)

# Provided that vodka and beer have been selected, the function will return
# => [1, 2]
```

Similar to `select` method, you can also provide options through DSL using the `choice` method for single entry and/or `choices` call for more than one choice:

```ruby
prompt.multi_select("Select drinks?") do |menu|
  menu.choice :vodka, {score: 1}
  menu.choice :beer, 2
  menu.choice :wine, 3
  menu.choices whisky: 4, bourbon: 5
end
```

To mark choice(s) as selected use the `default` option with index(s) of the option(s) starting from `1`:

```ruby
prompt.multi_select("Select drinks?") do |menu|
  menu.default 2, 5

  menu.choice :vodka,   {score: 10}
  menu.choice :beer,    {score: 20}
  menu.choice :wine,    {score: 30}
  menu.choice :whisky,  {score: 40}
  menu.choice :bourbon, {score: 50}
end
# =>
# Select drinks? beer, bourbon
#   ⬡ vodka
#   ⬢ beer
#   ⬡ wine
#   ⬡ whisky
# ‣ ⬢ bourbon
```

Like `select`, for ordered choices set `enum` to any delimiter String. In that way, you can use arrows keys and numbers (0-9) to select the item.

```ruby
prompt.multi_select("Select drinks?") do |menu|
  menu.enum ')'

  menu.choice :vodka,   {score: 10}
  menu.choice :beer,    {score: 20}
  menu.choice :wine,    {score: 30}
  menu.choice :whisky,  {score: 40}
  menu.choice :bourbon, {score: 50}
end
# =>
# Select drinks? beer, bourbon
#   ⬡ 1) vodka
#   ⬢ 2) beer
#   ⬡ 3) wine
#   ⬡ 4) whisky
# ‣ ⬢ 5) bourbon
```

And when you press enter you will see the following selected:

```ruby
# Select drinks? beer, bourbon
# => [{score: 20}, {score: 50}]
```

You can configure help message and/or marker like so

```ruby
choices = {vodka: 1, beer: 2, wine: 3, whisky: 4, bourbon: 5}
prompt.select("Select drinks?", choices, help: 'Press beer can against keyboard')
# =>
# Select drinks? (Press beer can against keyboard)"
# ‣ ⬡ vodka
#   ⬡ beer
#   ⬡ wine
#   ⬡ whisky
#   ⬡ bourbon
```

### 2.9 enum_select

In order to ask for standard selection from indexed list you can use `enum_select` and pass question together with possible choices:

```ruby
choices = %w(emacs nano vim)
prompt.enum_select("Select an editor?")
# =>
#
# Select an editor?
#   1) nano
#   2) vim
#   3) emacs
#   Choose 1-3 [1]:
```

Similar to `select` and `multi_select`, you can provide question options through DSL using `choice` method and/or `choices` like so:

```ruby
choices = %w(nano vim emacs)
prompt.enum_select("Select an editor?") do |menu|
  menu.choice :nano,  '/bin/nano'
  menu.choice :vim,   '/usr/bin/vim'
  menu.choice :emacs, '/usr/bin/emacs'
end
# =>
#
# Select an editor?
#   1) nano
#   2) vim
#   3) emacs
#   Choose 1-3 [1]:
#
# Select an editor? /bin/nano
```

You can change the indexed numbers by passing `enum` option and the default option by using `default` like so

```ruby
choices = %w(nano vim emacs)
prompt.enum_select("Select an editor?") do |menu|
  menu.default 2
  menu.enum '.'

  menu.choice :nano,  '/bin/nano'
  menu.choice :vim,   '/usr/bin/vim'
  menu.choice :emacs, '/usr/bin/emacs'
end
# =>
#
# Select an editor?
#   1. nano
#   2. vim
#   3. emacs
#   Choose 1-3 [2]:
#
# Select an editor? /usr/bin/vim
```

### 2.10 expand

The `expand` provides a compact way to ask a question with many options.

As first argument `expand` takes the message to display and as a second an array of choices. Compared to the `select`, `multi_select` and `enum_select`, the choices need to be objects that include `:key`, `:name` and `:value` keys. The `:key` must be a single character. The help choice is added automatically as the last option and the key `h`.

```ruby
choices = [
  {
    key: 'y',
    name: 'overwrite this file',
    value: :yes
  }, {
    key: 'n',
    name: 'do not overwrite this file',
    value: :no
  }, {
    key: 'q',
    name: 'quit; do not overwrite this file ',
    value: :quit
  }
]
```

The choices can also be provided through DSL using the `choice` method. The `:value` can be a primitive value or `Proc` instance that gets executed and whose value is used as returned type. For example:

```ruby
prompt.expand('Overwrite Gemfile?') do |q|
  q.choice key: 'y', name: 'Overwrite'      do :ok end
  q.choice key: 'n', name: 'Skip',          value: :no
  q.choice key: 'a', name: 'Overwrite all', value: :all
  q.choice key: 'd', name: 'Show diff',     value: :diff
  q.choice key: 'q', name: 'Quit',          value: :quit
end
```

The first element in the array of choices or provided via `choice` DSL will be the default choice, you can change that by passing `default` option.

```ruby
prompt.expand('Overwrite Gemfile?', choices)
# =>
# Overwrite Gemfile? (enter "h" for help) [y,n,q,h]
```

Each time user types an option a hint will be displayed:

```ruby
# Overwrite Gemfile? (enter "h" for help) [y,n,a,d,q,h] y
# >> overwrite this file
```

If user types `h` and presses enter, an expanded view will be shown which further allows to refine the choice:

```ruby
# Overwrite Gemfile?
#   y - overwrite this file
#   n - do not overwrite this file
#   q - quit; do not overwrite this file
#   h - print help
#   Choice [y]:
```

Run `examples/expand.rb` to see the prompt in action.

### 2.11 collect

In order to collect more than one answer use `collect` method. Using the `key` you can describe the answers key name. All the methods for asking user input such as `ask`, `mask`, `select` can be directly invoked on the key. The key composition is very flexible by allowing nested keys. If you want the value to be automatically converted to required type use [convert](#221-convert).

For example to gather some contact information do:

```ruby
prompt.collect do
  key(:name).ask('Name?')

  key(:age).ask('Age?', convert: :int)

  key(:address) do
    key(:street).ask('Street?', required: true)
    key(:city).ask('City?')
    key(:zip).ask('Zip?', validate: /\A\d{3}\Z/)
  end
end
# =>
# {:name => "Piotr", :age => 30, :address => {:street => "Street", :city => "City", :zip => "123"}}
```

### 2.12 suggest

To suggest possible matches for the user input use `suggest` method like so:

```ruby
prompt.suggest('sta', ['stage', 'stash', 'commit', 'branch'])
# =>
# Did you mean one of these?
#         stage
#         stash
```

To customize query text presented pass `:single_text` and `:plural_text` options to respectively change the message when one match is found or many.

```ruby
possible = %w(status stage stash commit branch blame)
prompt.suggest('b', possible, indent: 4, single_text: 'Perhaps you meant?')
# =>
# Perhaps you meant?
#     blame
```

### 2.13 slider

If you have constrained range of numbers for user to choose from you may consider using `slider`. The slider provides easy visual way of picking a value marked by `O` marker.

```ruby
prompt.slider('What size?', min: 32, max: 54, step: 2)
# =>
#
# What size? (User arrow keys, press Enter to select)
# |------O-----| 44
```

Slider can be configured through DSL as well:

```ruby
prompt.slider('What size?') do |range|
  range.default 4
  range.min 0
  range.max 20
  range.step 2
end
# =>
#
# What size? (User arrow keys, press Enter to select)
# |--O-------| 4
```

### 2.14 say

To simply print message out to stdout use `say` like so:

```ruby
prompt.say(...)
```

The `say` method also accepts option `:color` which supports all the colors provided by [pastel](https://github.com/piotrmurach/pastel#3-supported-colors)

**TTY::Prompt** provides more specific versions of `say` method to better express intenation behind the message such as `ok`, `warn` and `error`.

### 2.15 ok

Print message(s) in green do:

```ruby
prompt.ok(...)
```

### 2.16 warn

Print message(s) in yellow do:

```ruby
prompt.warn(...)
```

### 2.17 error

Print message(s) in red do:

```ruby
prompt.error(...)
```

## Contributing

1. Fork it ( https://github.com/piotrmurach/tty-prompt/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Copyright

Copyright (c) 2015-2016 Piotr Murach. See LICENSE for further details.
