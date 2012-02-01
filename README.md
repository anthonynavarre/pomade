# Veil

Mustache templates for both Ruby & JavaScript.

## Alpha, not even beta

Veil is in early alpha stage. In fact, as of this writing, there isn't even a
line of code written. Don't expect production-ready-anything anytime soon.

## Usage:

*IMPORTANT*: The following steps don't work yet - this is merely
"README-driven-development" at this point.

### Setup:

```
$: echo "gem 'veil'" > Gemfile
$: bundle install
```

### Basic Mustache-template Usage:

Veil can be used solely to serve up mustache templates if you so desire. Just
write your templates under `/app/templates` like so:

```ruby
# /app/templates/todo.jst.erb
<li>{{name}}</li>
```

### As a decorator:

Veil uses the most excellent [Draper gem](https://github.com/jcasimir/draper)
under the covers, so anything you can do with Draper, you can do with a Veil
template. To generate a Veil template that decorates a model called `Todo`:

```
rails generate veil:decorator Todo
```

This generates a decorator at `/app/decorators/todo_decorator.rb` that can be
used just like any Draper decorator. It also generates a corresponding template
at `/app/templates/todo.jst.erb`

Any method calls within the template will use your decorator when called from
Ruby, and will render JavaScript-ready handlebars when compiled to
`application.js`:

```ruby
# /app/decorators/todo.rb
class TodoDecorator < ApplicationDecorator
  use Veil::Base

  def due_date
    date = h.content_tag(:span, todo.due_at.strftime("%A, %B %e").squeeze(" "), :class => 'date')
    time = h.content_tag(:span, todo.due_at.strftime("%l:%M%p"), :class => 'time').delete(" ")
    h.content_tag :span, date + time, :class => 'due_date'
  end
end
```

```ruby
# /app/controllers/todos_controller.rb
class TodosController < ApplicationController

  def show
    @todo = TodoDecorator.find(params[:id])
  end

end
```

```ruby
# /app/views/todos/show.html.erb
<p>Todo: <%= @todo.title %> (Due: <%= @todo.due_date %>)</p>
```

So, assuming your routes are all set up as usual, when you visit `/todos/1` you should get:

```html
<p>Todo: Shave a yak (Due: <span class="due_date"><span class="date">Wednesday,
February 1</span><span class="time"> 4:41PM</span></span>)</p>
```

And when you visit `/assets/application.js` you should get the following
template (by default saved in an object called `Handlebars.templates` with key
`todo_template`):

```html
<p>Todo: {{title}} (Due: {{dueDate}})</p>

```

#### TODO / Roadmap:

1. Figure out ways to preserve *some* decorator behavior from Draper when
   carrying over to JS Templates. eg: t'would be nice to somehow get `<span
   class="date">` etc involved.

### Asset Dependencies:

TODO: Figure out the idiomatic way to deal w/ template & specific decorator
changes in deve mode

## Goals

Veil is intended to allow Rails developers to maximize the usefulness of their
JavaScript templates, and minimize duplication. To achieve this, some up-front
design decisions:

1. I should be able to use a templating language of my choice (so, anything
   Tilt can handle - slim, haml, erb, etc).
2. Veil templates should be useful both server-side and client-side (using
   decorators server-side, and rendering "handlebars" client-side)
3. Veil templates shouldn't lock you into using Veil - we'll rely on
   well-established precedent and name our files in a way that they can be used
   later if you choose to stop using Veil. eg: `/app/templates/todo.jst.slim`
4. My views should not need to reference templates or be concerned about which
   templates they will need. We'll follow the lead of [the Hamlbars
   gem](https://github.com/jamesotron/hamlbars) and render all templates (by
   default) to application.js when the asset pipeline does its thing - they
   will already be loaded into a configurable client-side variable without
   needing to call `render partial: 'templates/todo'`.

