# CS 270 - Spring 2015 - QR Code Scavenger Hunt - Phase 1

In this phase, we will create the Event and Location resources.  In
doing so, we will need to figure out how to wire together these two
resources in a logical manner to reflect the dependencies we have
specified for the project.

A new Rails app has already been created.  To get started, simply clone
the respository into your system.

As a side note, the `rqrcode` gem has already been included into the
Gemfile.  To complete this phase, then, you should simply need to do the
following:

## Event resource

An Event will consist of the following attributes:

- name (as a string)

It will also have a `has_and_belongs_to_many` association with the
Location resource.  In order to accomplish this, we will generate the
Event and Locations resources first, and then we will create the
association.  We can generate the Event resource with a scaffolding
command:

`rails g scaffold Event name:string`

## Location resource

A Location will consist of the following attributes:

- name (as a string)
- tag (as a randomly generated string of 8 alphanumeric characters)

It will also have a `has_and_belongs_to_many` association with the Event
resource.  We can generate this resource with a scaffolding command:

`rails g scaffold Location name:string tag:string event:references`

Remember to migrate the database to account for these resources: `rake
db:migrate`

## Creating the associations 

The association between Events and Locations is a bit tricky. It seems
reasonable that we should have a many-many type of association between Events
and Locations.  This will allow us to create an Event and simply "select"
Locations from a common pool, rather than specifying a potentially duplicate
list of Locations for each Event.  This association will take the form of a
"join table" that we will create with a database migration that simply maps
each Location in an Event to every Participant of that Event:

```
event 1 | location 1
event 1 | location 3
event 2 | location 1
event 2 | location 2
event 3 | location 2
event 3 | location 3`
```

We call this type of association in Rails a `has_and_belongs_to_many`
association.  It is a bit more involved to set it up: First, we must
edit the Event model and the Location model to specify the association:

in Event: `has_and_belongs_to_many :locations` in Location:
`has_and_belongs_to_many :events`

Next, we must create a database migration that will create a the join
table for us.  The Rails convention is that join tables are named in the
form of `model1plural_model2plural`, in alphabetical order:

`rails g migration CreateEventsParticipants`

The above command will generate a migration that creates a table called
`events_participants`.

We need to go in and edit the table to specify the columns: simply add
the belongs_to method for each model, as well as an index.

`t.belongs_to :participants, index: true` `t.belongs_to :locations,
index: true`

Finally, make sure we "turn off" the primary key id column before
passing the block into the create_table command, by modifying the line
like so:

`def create_table :events_locations, id: false do |t|`

Now, we should be able to run the migration as per normal: `rake
db:migrate`

## Seeding the database

We can initialize the database with some test data to get started.  We
will edit the `config/seeds.rb` file to generate 2 events and 3
locations for each event:

```
2.times do |i|
  Event.create(name: "Event #{i + 1}"
  3.times do |j|
    if Location.any?
      id = Location.last.id
    else
      id = 0
    end
    Location.create(
      name: "Location #{id + 1} belongs to Event #{i + 1}",
      tag: (('A'..'Z').to_a + ('a'..'z').to_a + (0..9).to_a).shuffle[0..7].join,
      event_id: i + 1
    )
  end
end
```
After creating our seed file, we can initialize the database with:

`rake db:seed`

and at anytime we "mess up" our data, we can reset it with:

`rake db:reset`

## Modifying the views

We want the following behaviour when we run our app:

- When we list an event, we want a list of locations displayed as well
- When we edit an event, we want to add, delete, or edit locations as
  well
- When we create a new location, we want to automatically assign it to
  the event from which we created the location

In order to do this, we will need to modify the `_form.html.erb` and
`edit.html.erb` views for the Event resource, as well as the
`_form.html.erb` view for the Location resource.
